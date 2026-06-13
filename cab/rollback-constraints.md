# Rollback Constraints

Scenarios where rolling a change *back* is impossible, requires data loss, or needs a specific
multi-step procedure. A change being hard or impossible to reverse is a major input to risk — a
HIGH-risk PR that is also irreversible deserves the most scrutiny. Cross-reference
[`deployment-impact.md`](deployment-impact.md) for the forward-deployment side of each.

---

## 1. MetadataVersion with `didMetadataChange = true`

**Where:** `server-common/src/main/java/org/apache/kafka/server/common/MetadataVersion.java`

Each enum constant is declared as
`IBP_x_y_IVn(featureLevel, "x.y", "IVn", didMetadataChange)`. The **last boolean parameter** is
`didMetadataChange`. When it is `true`, the cluster metadata format changed at that version, and
**metadata downgrade across that version is not supported.**

The code enforces this: `checkIfMetadataChangedOrdered(...)` walks from the higher version down to
the lower one and, if any version in between has `didMetadataChange() == true`, reports that
metadata changed — i.e. the downgrade would cross a metadata-format boundary.

**CAB check:** if a PR **adds a new `MetadataVersion` entry with `true` as the last parameter**,
rollback past that version is **blocked**. Recent examples in the source enum:

```java
IBP_4_0_IV1(23, "4.0", "IV1", true),   // didMetadataChange = true  -> downgrade blocked
IBP_4_3_IV0(30, "4.3", "IV0", true),   // didMetadataChange = true  -> downgrade blocked
IBP_4_4_IV0(31, "4.4", "IV0", false),  // didMetadataChange = false -> downgrade across this is OK
```

A new entry with `false` does **not** by itself block downgrade. Read the boolean carefully —
this is the single most important rollback signal in the codebase.

---

## 2. RocksDB format changes in Streams

**Where:** `gradle/dependencies.gradle` (`rocksDB` version) + `streams/` state stores.

Kafka Streams uses RocksDB for local state stores. **Downgrading from 4.0.x+ to an older
release requires deleting local state stores**, because the on-disk RocksDB format can change
across major RocksDB upgrades and older code cannot open the newer files.

**CAB check:** any PR **bumping the `rocksDB` version in `gradle/dependencies.gradle`** triggers
this constraint. Current pin: `rocksDB: "10.1.3"`. A RocksDB bump is therefore HIGH risk (it is a
state-store format concern, not just a dependency bump) and implies state-store deletion on
rollback.

---

## 3. Streams serialization format (KIP-904)

Downgrading Kafka Streams **from 3.5+ to 3.4** requires **two rolling bounces** using the
`upgrade.from` config to negotiate the older serialization format during the transition. It is not
a simple single-step rollback.

**CAB check:** changes to Streams changelog/repartition serialization, or upgrade-path handling,
inherit this multi-bounce rollback choreography.

---

## 4. Streams KIP-1035 (4.3+)

For Streams on **4.3+**, an **in-flight downgrade to 4.2.x is not supported without deleting the
state directory.** Treat any KIP-1035-related Streams change as imposing a state-directory wipe on
rollback to 4.2.x.

---

## 5. Protocol API version removal

Once old protocol API versions are **removed** (as happened in the 4.0 release), **clients running
older versions can no longer connect** — and you cannot "roll back" the wire compatibility for
those clients without re-adding the versions. This couples to
[`protocol-change-rules.md`](protocol-change-rules.md): increasing the lower bound of
`validVersions` or dropping versions strands old clients.

**CAB check:** if a PR removes protocol versions, rollback does not restore connectivity for
clients that were already upgraded/forced off the old versions; plan client/broker upgrade order
accordingly.

---

## Summary table

| Scenario | Trigger in a PR | Rollback consequence |
|---|---|---|
| MetadataVersion bump | New `MetadataVersion` entry with `true` last param | Metadata downgrade blocked across that version |
| RocksDB format | `rocksDB` version bump in `dependencies.gradle` | Must delete local state stores to downgrade |
| Streams serialization (KIP-904) | Streams ser/de or upgrade-path change | Two rolling bounces with `upgrade.from` |
| Streams KIP-1035 (4.3+) | KIP-1035 Streams change | Delete state directory to downgrade to 4.2.x |
| Protocol API removal | Removing/raising `validVersions` lower bound | Old clients cannot reconnect |
