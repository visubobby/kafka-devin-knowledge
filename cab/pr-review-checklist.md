# PR Review Checklist

The step-by-step procedure a CAB member (or Devin) runs on every PR against `visubobby/kafka`.
Work top to bottom; each step points at the reference doc that backs it. Record your findings as
you go and finish with an explicit risk level + justification.

Start by getting the diff's file list:

```bash
git fetch origin
git diff --name-only origin/<base>...HEAD     # or use the PR "Files changed" tab
```

---

### 1. Identify changed modules
List every top-level directory the PR touches (e.g. `core/`, `clients/`, `streams/src/test/...`).
This is the raw input for everything below.

### 2. Check module criticality
For each touched directory, look it up in [`module-criticality.md`](module-criticality.md).
Note the highest criticality present. CRITICAL/HIGH modules in *production* code (not test) push
toward HIGH risk.

### 3. Check for protocol changes
Does the diff touch any `.json` under `clients/src/main/resources/common/message/`?
If so, open [`protocol-change-rules.md`](protocol-change-rules.md) and classify each change as
**incompatible (HIGH)** or **safe (LOW)**. Pay special attention to edits of existing fields and to
the `validVersions` range. (Also check `generator/` changes that affect serialization.)

### 4. Check for MetadataVersion changes
Does the diff add or modify an entry in
`server-common/src/main/java/org/apache/kafka/server/common/MetadataVersion.java`?
If a **new entry** is added, read the **last boolean parameter** (`didMetadataChange`):
- `true`  ⇒ metadata downgrade blocked → HIGH + rollback blocker
  (see [`rollback-constraints.md`](rollback-constraints.md) §1).
- `false` ⇒ still a metadata/feature-flag change (HIGH because it's `server-common`/metadata), but
  not a downgrade blocker by itself.

### 5. Check for dependency version bumps
Does the diff change `gradle/dependencies.gradle`? If a version was bumped, note which library.
If it is **lz4, zstd, RocksDB, or Jetty**, **escalate to HIGH**:
- `lz4` / `zstd` → compression on the wire / on disk.
- `rocksDB` → Streams state-store format → rollback requires deleting state stores
  ([`rollback-constraints.md`](rollback-constraints.md) §2).
- `jetty` → security-sensitive HTTP stack (Connect REST).
Other dependency bumps are MEDIUM.

### 6. Check for config default changes
Does the diff change a `ConfigDef` default, or a default in `config/`, `server/`, `connect/`?
Changing a shipped default changes behavior for every operator who relied on it → at least MEDIUM,
and it implies a rolling restart (see [`deployment-impact.md`](deployment-impact.md)).

### 7. Check API stability
For changed **public** APIs in `clients/`, find the `@InterfaceStability` annotation (or its
absence) per [`api-stability-guide.md`](api-stability-guide.md). Decide breaking vs. additive, then
map: `@Stable` (or unannotated) breaking → HIGH, `@Evolving` → MEDIUM, `@Unstable` → LOW.

### 8. Check test coverage
Does the PR include tests? If the diff is **only** under `*/src/test/`, it's LOW (test-only).
Production changes *without* tests don't lower risk — note the gap in your justification.

### 9. Check rollback feasibility
Walk [`rollback-constraints.md`](rollback-constraints.md). Flag any of: `didMetadataChange=true`
MetadataVersion entry, RocksDB bump, Streams serialization/upgrade changes (KIP-904 / KIP-1035),
protocol version removal. An irreversible change should be called out prominently.

### 10. Assign risk level
Using [`risk-taxonomy.md`](risk-taxonomy.md), assign **LOW / MEDIUM / HIGH** — highest matching
tier wins. Write a justification that names the exact rule(s) that fired, the affected modules and
blast radius, and any rollback/deployment constraints from
[`deployment-impact.md`](deployment-impact.md).

---

## Output template

```
PR: <title / link>
Changed modules: <list> (highest criticality: <CRITICAL|HIGH|MEDIUM|LOW>)
Protocol change: <none | safe (additive/tagged) | incompatible: ...>
MetadataVersion: <none | new entry didMetadataChange=<true|false>>
Dependency bumps: <none | lib@old->new (escalated? y/n)>
Config default changes: <none | ...>
API stability: <n/a | @Stable|@Evolving|@Unstable, breaking|additive>
Tests included: <yes|no|test-only>
Rollback constraints: <none | metadata-downgrade-blocked | rocksdb-state-wipe | ...>
Deployment impact: <rolling restart | config-only | offline upgrade | two-phase | client/broker order>

RISK: <LOW | MEDIUM | HIGH>
Justification: <which rule fired, blast radius, reversibility>
```
