# architecture/

Architecture and design documentation for Kafka, written for Devin knowledge use. `overview.md`
is authored fresh; the rest are **adapted** from `visubobby/kafka/docs/` (Hugo frontmatter and
license boilerplate stripped, content preserved and reorganized).

## How Devin should use this

- Read these to understand *why* Kafka behaves the way it does before changing server, storage,
  or KRaft code.
- These are documentation snapshots. If the source code has evolved, cross-check against
  `visubobby/kafka` — the code is authoritative over prose.

## Files

| File | Source | Purpose |
| --- | --- | --- |
| `overview.md` | authored | High-level summary of KRaft-based architecture, roles, replication |
| `kraft-operations.md` | `docs/operations/kraft.md` | Configuring/operating KRaft clusters |
| `zk-to-kraft-migration.md` | `docs/getting-started/zk2kraft.md` | ZooKeeper → KRaft differences & migration |
| `design.md` | `docs/design/design.md` | Core design rationale (persistence, efficiency, delivery semantics, replication) |
| `log-implementation.md` | `docs/implementation/log.md` | On-disk log/segment storage |
| `message-format.md` | `docs/implementation/message-format.md` | Record batch / record wire format |

## Related

- Operational steps live in [`../playbooks/`](../playbooks/).
- Default config values live in [`../configs/`](../configs/).
