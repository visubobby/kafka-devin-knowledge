# Protocol Change Rules

Extracted from `clients/src/main/resources/common/message/README.md` in `visubobby/kafka`.
The JSON files in `clients/src/main/resources/common/message/*.json` define the Kafka wire
protocol; at compile time they are translated into the Java serialization/deserialization code.
**Any change to these files changes how clients and brokers talk to each other across versions**,
which is why protocol changes are a HIGH-risk trigger in [`risk-taxonomy.md`](risk-taxonomy.md).

## Background (the rules these come from)

- Each message has a top-level **`validVersions`** range (e.g. `"0-2"` means versions 0, 1, 2 are
  understood). You must always specify the highest supported version.
- **The order of fields matters.** Fields are sent over the wire in definition order; reordering
  is an incompatible change.
- New versions add or remove fields by version spec: a new field in version 3 is `"3+"`; a field
  removed in version 3 changes from `"0+"` to `"0-2"`.
- **Dropping support for old message versions is not allowed without a KIP**, so you must not raise
  the lower bound of any message's version interval.
- **Tagged fields (KIP-482)** can be added to message versions that already exist; older servers
  simply ignore tagged fields they don't understand. Tagged fields are only allowed in "flexible"
  message versions, and a tag number can never be reused for something else.

## Incompatible changes — HIGH risk

A protocol change is **incompatible** (and therefore HIGH risk) if it does any of the following:

- **Modifying a protocol version that has already been released.** Once a version is shipped, its
  schema is frozen; changing it breaks deployed clients/brokers.
- **Re-ordering existing fields** in a message definition. Wire order is part of the contract.
- **Changing the default value of an existing field.** Receivers fill in defaults for absent
  fields, so changing a default silently changes behavior across versions.
- **Changing the type of an existing field** (e.g. `int32` → `int64`, or `string` → `uuid`).
- **Dropping support for old message versions without a KIP.**
- **Increasing the lower bound of `validVersions`** (this is the concrete form of "dropping old
  versions" — it strands clients that only speak the removed versions).

Any of these in a PR ⇒ **HIGH**, and likely a **client-before/after-broker upgrade ordering**
concern — see [`deployment-impact.md`](deployment-impact.md).

## Safe changes — LOW risk

These changes are backward/forward compatible and are LOW risk:

- **Adding new tagged fields (KIP-482).** No version bump needed; older peers ignore unknown tags.
- **Adding new fields with version spec `N+` in a *new* version.** The field only appears for peers
  that negotiated version N or later; older peers never see it.
- **Changing the `mapKey` annotation.** This is a code-generation/handling hint, not a wire-format
  change.

## Reviewer heuristics

- **Diff the `validVersions` line.** If the lower bound went up, or a released upper bound's schema
  changed, that's HIGH.
- **Look for edits to existing field lines** (type, default, order, version range narrowing) vs.
  **purely additive lines** (new `"N+"` fields, new `"tag"`/`taggedVersions` entries). Additive =
  usually safe; edits to existing fields = usually incompatible.
- **No KIP referenced for a version-dropping or schema-altering change?** Escalate and ask — the
  README explicitly forbids dropping versions without a KIP.
- **Generator changes count too.** A change in `generator/` that alters how `message/*.json` is
  serialized has the same blast radius as editing the JSON — treat as a protocol change.
