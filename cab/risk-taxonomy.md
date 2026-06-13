# Risk Taxonomy

Classification rules for assigning a risk level to a PR against `visubobby/kafka`.

**Rule of precedence:** the highest-matching tier wins. A PR that touches even one HIGH path is
**HIGH overall**, regardless of how trivial the rest of the diff is. Only when *no* HIGH rule
matches do you fall through to MEDIUM, and only when no HIGH or MEDIUM rule matches is a PR LOW.

---

## HIGH risk

Any PR that touches:

- **Wire protocol definitions** — `clients/src/main/resources/common/message/*.json`
- **`server-common/src/main/java/org/apache/kafka/server/common/MetadataVersion.java`**
- **`raft/`** module (KRaft consensus, controller election)
- **`metadata/`** module (cluster metadata, feature flags)
- **`core/`** module (broker critical path)
- **`storage/`** module (log format, segment management)
- Any file that **changes a released protocol version** — i.e. modifying an existing
  `validVersions` range (see [`protocol-change-rules.md`](protocol-change-rules.md))
- Any change to **`@InterfaceStability.Stable` public API signatures** in `clients/`
  (see [`api-stability-guide.md`](api-stability-guide.md))
- **RocksDB state store format** in `streams/`
- **Security-related code**: SASL, SSL, ACL, OAuth

> Why HIGH: these are cross-cluster, cross-version, and frequently **not cleanly reversible**.
> A mistake can break client/broker interop, corrupt or lock cluster metadata, change on-disk
> log/state formats, or open a security hole. Cross-check [`rollback-constraints.md`](rollback-constraints.md).

---

## MEDIUM risk

Any PR that touches (and matches **no** HIGH rule):

- **Config defaults** in `config/`, `server/`, `connect/`
- **`gradle/dependencies.gradle` version bumps** — especially `lz4`, `zstd`, `rocksDB`, `jetty`
  (if the bump is to lz4/zstd/RocksDB, **escalate to HIGH** — these affect on-the-wire/on-disk
  formats; see the checklist)
- **`@InterfaceStability.Evolving` API changes**
- **`group-coordinator/`**, **`transaction-coordinator/`**, **`share-coordinator/`**
- **`connect/`** runtime
- **`streams/`** (non-state-store changes)
- **New KIP feature flags** or feature version bumps
- **`docker/`** Dockerfile changes

> Why MEDIUM: real production impact (behavior change, rebalance/EOS semantics, dependency
> surface) but normally recoverable via redeploy/reconfig, and generally version-compatible.

---

## LOW risk

Any PR that touches **only**:

- **`*/src/test/`** — test-only changes
- **`tools/`**, **`trogdor/`**, **`jmh-benchmarks/`**
- **`checkstyle/`**, **`docs/`**, **`vagrant/`**, **`committer-tools/`**
- **`@InterfaceStability.Unstable` API changes**
- **Adding new tagged fields** to a protocol message (no version bump needed, per KIP-482)
- **`gradle/` build script changes** that are *not* dependency version changes

> Why LOW: no broker/client runtime path, no format/version impact, fully reversible.

---

## Quick decision flow

1. Does the diff touch any **HIGH** path/condition above? → **HIGH**. Stop.
2. Else, does it touch any **MEDIUM** path/condition? → **MEDIUM** (remember the
   lz4/zstd/RocksDB dependency-bump escalation to HIGH). Stop.
3. Else → **LOW**.
4. Always write a one-line justification naming the exact rule that fired.
