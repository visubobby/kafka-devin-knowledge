# Kafka Architecture Overview (KRaft)

> Summary of Apache Kafka's KRaft-based architecture, written for Devin knowledge use.
> For canonical detail, see the adapted docs in this directory and the source repo `visubobby/kafka`.

## What Kafka is

Apache Kafka is a distributed event streaming platform. Producers append records to
**topics**, which are split into **partitions**. Each partition is an ordered, append-only
log persisted on disk. Consumers read records from partitions, tracking their position by
**offset**. Partitions are replicated across brokers for durability and availability.

## KRaft: Kafka without ZooKeeper

Modern Kafka runs in **KRaft** (Kafka Raft) mode, which replaces ZooKeeper for cluster
metadata management. Metadata (topics, partitions, ACLs, configs, broker registrations) is
itself stored in an internal Kafka log, the `__cluster_metadata` topic, managed by a Raft
quorum of **controllers**.

Key components:

- **Controller quorum** — a set of nodes running with `process.roles` including `controller`.
  They elect a leader via Raft and manage all cluster metadata. The metadata log is the single
  source of truth for cluster state.
- **Broker** — a node running with `process.roles` including `broker`. Brokers host topic
  partitions, serve produce/fetch requests, and replicate data. Brokers learn cluster state by
  replaying the metadata log from the controllers.
- **Combined mode** — a single node can run as both broker and controller
  (`process.roles=broker,controller`). Useful for development; not recommended for large
  production clusters where roles should be isolated.

Because metadata is a replicated log, controller failover and broker metadata propagation are
fast and consistent. See [`kraft-operations.md`](kraft-operations.md) for provisioning details
and [`zk-to-kraft-migration.md`](zk-to-kraft-migration.md) for migration considerations.

## Node roles and process model

| Role               | `process.roles`        | Default config file (`config/`)        |
| ------------------ | ---------------------- | -------------------------------------- |
| Combined           | `broker,controller`    | `server.properties`                    |
| Broker only        | `broker`               | `broker.properties`                    |
| Controller only    | `controller`           | `controller.properties`                |

Each node has a unique `node.id`. Controllers are listed in `controller.quorum.voters` (or
discovered via `controller.quorum.bootstrap.servers`). Listeners are declared in
`listeners` / `advertised.listeners`, with the controller listener named via
`controller.listener.names` (default `CONTROLLER`).

## Storage and the log

Each partition replica is stored as a sequence of **log segments** on disk under
`log.dirs`. Writes append to the active segment; reads are served by offset. Retention is
controlled by time/size. See [`log-implementation.md`](log-implementation.md) and
[`message-format.md`](message-format.md) for on-disk detail.

KRaft requires the storage directory to be **formatted** with a cluster ID before first start
using `bin/kafka-storage.sh format`. See [`../playbooks/kraft-cluster-setup.md`](../playbooks/kraft-cluster-setup.md).

## Replication, ISR, and the high watermark

Each partition has one **leader** and zero or more **follower** replicas. Followers fetch from
the leader to stay caught up. The set of replicas considered caught up is the **in-sync replica
(ISR)** set. The **high watermark** is the highest offset replicated to all ISR members; only
records below the high watermark are visible to consumers. Producer durability is controlled by
`acks` (`acks=all` waits for the full ISR).

In development configs the internal-topic and default replication factors are set to `1`. For
production these must be increased to `3` or more — see
[`../playbooks/kraft-cluster-setup.md`](../playbooks/kraft-cluster-setup.md).

## Higher-level components

- **Kafka Connect** — framework for streaming data between Kafka and external systems via
  source/sink connectors. Runs standalone or distributed. See
  [`../playbooks/connect-deployment.md`](../playbooks/connect-deployment.md).
- **Kafka Streams** — a client library for stateful stream processing on top of Kafka topics.
  See [`../playbooks/streams-app-reset.md`](../playbooks/streams-app-reset.md).
- **MirrorMaker 2** — cross-cluster replication, built on Connect.

## Source module map (in `visubobby/kafka`)

- `clients/` — Producer, Consumer, Admin client APIs (Java)
- `core/` — broker server logic, request handling (Scala/Java)
- `metadata/` — KRaft controller and cluster metadata
- `group-coordinator/` — consumer/share/streams group management
- `storage/` — log storage and tiered storage
- `server/` — shared server-side utilities and network layer
- `streams/` — Kafka Streams library
- `connect/` — Kafka Connect framework and runtime
- `bin/` — shell scripts to run components and tools
- `config/` — default configuration files (mirrored in this repo's `configs/`)
