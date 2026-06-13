# Playbook: Kafka Connect Deployment

Goal: run Kafka Connect to stream data between Kafka and external systems. Connect runs in two
modes: **standalone** (single process, config in a file) and **distributed** (scalable,
fault-tolerant, managed via REST). Reference configs are mirrored in
[`../configs/`](../configs/).

## Distributed mode (recommended for anything beyond a quick test)

Uses [`../configs/connect-distributed.properties`](../configs/connect-distributed.properties).
Key settings in that file:

- `bootstrap.servers=localhost:9092` — the Kafka cluster to connect to.
- `group.id=connect-cluster` — Connect worker group; all workers sharing this id form one
  cluster.
- `offset.storage.topic`, `config.storage.topic`, `status.storage.topic` — internal Kafka
  topics Connect uses to persist connector offsets, configs, and status.
- `offset.storage.replication.factor`, `config.storage.replication.factor`,
  `status.storage.replication.factor` — set to **1** in the dev config; use **3** in production.
- `key.converter` / `value.converter` — default to JSON; set per-connector as needed.
- `plugin.path` — directory containing connector plugins (uncomment/set this to where your
  connector jars live).

### Steps

1. Ensure the Kafka cluster is running (see
   [`kraft-cluster-setup.md`](kraft-cluster-setup.md)).
2. Make sure connector plugins are installed and `plugin.path` points at them.
3. For production, raise the three `*.storage.replication.factor` values to 3+ first.
4. Start a worker (repeat on each machine to scale out; they join via `group.id`):
   ```bash
   bin/connect-distributed.sh config/connect-distributed.properties
   # background: bin/connect-distributed.sh -daemon config/connect-distributed.properties
   ```
5. Manage connectors via the REST API (default port `8083`):
   ```bash
   # list installed plugins
   curl http://localhost:8083/connector-plugins

   # create a connector
   curl -X POST http://localhost:8083/connectors \
     -H 'Content-Type: application/json' \
     -d '{"name":"my-connector","config":{"connector.class":"...","tasks.max":"1", ...}}'

   # list / inspect / delete
   curl http://localhost:8083/connectors
   curl http://localhost:8083/connectors/my-connector/status
   curl -X DELETE http://localhost:8083/connectors/my-connector
   ```

## Standalone mode (single process, dev/testing)

Uses [`../configs/connect-standalone.properties`](../configs/connect-standalone.properties)
plus one or more connector property files passed on the command line:

```bash
bin/connect-standalone.sh config/connect-standalone.properties \
  config/connect-file-source.properties config/connect-file-sink.properties
```

Standalone stores offsets in a local file (`offset.storage.file.filename`) rather than Kafka
topics — there is no REST-based cluster coordination.

## MirrorMaker 2 (cross-cluster replication)

For replicating between clusters, use
[`../configs/connect-mirror-maker.properties`](../configs/connect-mirror-maker.properties) with
`bin/connect-mirror-maker.sh config/connect-mirror-maker.properties`.
