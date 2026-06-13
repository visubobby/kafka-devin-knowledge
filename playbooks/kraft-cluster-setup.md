# Playbook: KRaft Cluster Setup

Goal: stand up a KRaft-mode Kafka cluster (no ZooKeeper) using the config files mirrored in
[`../configs/`](../configs/). For background, see
[`../architecture/kraft-operations.md`](../architecture/kraft-operations.md) and
[`../architecture/overview.md`](../architecture/overview.md).

The canonical config files (copied from `visubobby/kafka/config/`) are:

- [`../configs/server.properties`](../configs/server.properties) — combined node
  (`process.roles=broker,controller`)
- [`../configs/broker.properties`](../configs/broker.properties) — broker-only
  (`process.roles=broker`)
- [`../configs/controller.properties`](../configs/controller.properties) — controller-only
  (`process.roles=controller`)

Use the actual paths under `config/` in the repo when running the scripts below.

## Modes

### Combined mode (broker + controller) — simplest, good for dev

Uses `config/server.properties`, which sets:
- `process.roles=broker,controller`
- `listeners=PLAINTEXT://:9092,CONTROLLER://:9093`
- `controller.listener.names=CONTROLLER`
- `controller.quorum.bootstrap.servers=localhost:9093`
- `log.dirs=/tmp/kraft-combined-logs`

### Broker-only mode

Uses `config/broker.properties` with `process.roles=broker`, listening on `PLAINTEXT://...:9092`,
pointing at the controller quorum via `controller.quorum.bootstrap.servers`. Pair it with one or
more controller-only nodes using `config/controller.properties`
(`process.roles=controller`, `listeners=CONTROLLER://:9093`).

## Default ports

- **Broker**: `9092` (`PLAINTEXT` listener)
- **Controller**: `9093` (`CONTROLLER` listener)

## Steps (combined single-node dev cluster)

1. **Generate a cluster ID** (once per cluster):
   ```bash
   KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
   ```
2. **Format the storage directory** — KRaft requires this before the first start. It writes the
   cluster ID and bootstrap metadata into `log.dirs`:
   ```bash
   bin/kafka-storage.sh format -t "$KAFKA_CLUSTER_ID" -c config/server.properties --standalone
   ```
   (Use `-c config/controller.properties` / `config/broker.properties` when formatting
   role-separated nodes. The `--standalone` flag bootstraps a single-voter quorum for a dev
   node; omit it and configure `controller.quorum.voters` for a multi-node quorum.)
3. **Start the server**:
   ```bash
   bin/kafka-server-start.sh config/server.properties
   ```
   To run in the background: `bin/kafka-server-start.sh -daemon config/server.properties`.
4. **Verify** by listing topics (broker on 9092):
   ```bash
   bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
   ```
5. **Stop**:
   ```bash
   bin/kafka-server-stop.sh
   ```

## Multi-node (role-separated) outline

1. Pick unique `node.id` per node.
2. On controllers, format with `config/controller.properties`; on brokers, format with
   `config/broker.properties` — all using the **same** `$KAFKA_CLUSTER_ID`.
3. Configure the controller quorum (`controller.quorum.voters=<id1>@host1:9093,...` or
   `controller.quorum.bootstrap.servers`) consistently across nodes.
4. Start controllers first, then brokers.

## Production checklist (IMPORTANT)

The dev configs set replication factors and ISR minimums to **1** so a single broker works.
These are **not safe for production**. Before going to production, raise them to **3 or more**
(and ensure you have at least that many brokers):

- `offset.storage.replication.factor` / `offsets.topic.replication.factor`
- `transaction.state.log.replication.factor` and `transaction.state.log.min.isr`
- `share.coordinator.state.topic.replication.factor` and `share.coordinator.state.topic.min.isr`
- default topic `replication.factor` and per-topic `min.insync.replicas`

Also use a secured listener (e.g. `SASL_SSL`) instead of `PLAINTEXT`, run controllers on
dedicated nodes, and point `log.dirs` at durable storage rather than `/tmp`.
