# Module Criticality

Ranked list of `visubobby/kafka` modules with their criticality and blast radius. Use this in
step 2 of [`pr-review-checklist.md`](pr-review-checklist.md) to translate "which directories did
the PR touch" into "how far can this blow up."

| Module | Criticality | Blast Radius |
|---|---|---|
| `core/` | CRITICAL | Broker startup, partition leadership, all broker operations |
| `raft/` | CRITICAL | KRaft consensus, controller election |
| `metadata/` | CRITICAL | Cluster metadata, feature flags |
| `storage/` | CRITICAL | Log format, segment management |
| `clients/` | HIGH | All producer/consumer/admin client users |
| `group-coordinator/` | HIGH | Consumer group rebalancing |
| `transaction-coordinator/` | HIGH | Exactly-once semantics |
| `server/` | HIGH | Broker configuration and lifecycle |
| `server-common/` | HIGH | Shared broker utilities, MetadataVersion |
| `connect/` | MEDIUM | Kafka Connect runtime |
| `streams/` | MEDIUM | Kafka Streams applications |
| `coordinator-common/` | MEDIUM | Shared coordinator logic |
| `share-coordinator/` | MEDIUM | Share groups (preview feature) |
| `shell/` | LOW | CLI tooling |
| `tools/` | LOW | Admin tools, not in broker path |
| `trogdor/` | LOW | Test framework only |
| `jmh-benchmarks/` | LOW | Benchmarks only |
| `generator/` | LOW | Code generation, build-time only |

## How to read this

- **CRITICAL / HIGH** modules feed the **HIGH** risk tier in
  [`risk-taxonomy.md`](risk-taxonomy.md). A production-code change in `core/`, `raft/`,
  `metadata/`, `storage/`, or a `@Stable` API in `clients/` should be treated as HIGH.
- **Criticality is about blast radius, not difficulty.** A one-line change in `core/` is still
  CRITICAL because of *where* it runs, not how big it is.
- **Module ≠ final risk.** A change confined to `*/src/test/` inside a CRITICAL module is still
  LOW (test-only), and a dependency/format change inside a MEDIUM module (e.g. RocksDB in
  `streams/`) escalates to HIGH. Always reconcile with the taxonomy rules.
- **`server-common/` is special:** it hosts `MetadataVersion.java`. Treat any change there as
  potentially metadata-affecting and check [`rollback-constraints.md`](rollback-constraints.md).

## Notes on a few entries

- **`coordinator-common/`** is shared plumbing for the coordinator modules; a change here can
  ripple into group/transaction/share coordinators, so weigh it against those consumers.
- **`generator/`** is build-time only, but it generates the protocol serialization code from the
  `message/*.json` files — so a generator change can still alter wire behavior. If a generator
  change affects how messages are serialized, treat it like a protocol change (HIGH).
- **`shell/`, `tools/`, `trogdor/`, `jmh-benchmarks/`** are not on the broker request path;
  problems there do not take down a cluster.
