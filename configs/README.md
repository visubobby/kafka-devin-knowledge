# configs/

**Verbatim copies** of configuration files from `visubobby/kafka/config/` (and the broker/
controller/server property files used for KRaft). These are the **canonical reference** for
default Kafka configuration.

## How Devin should use this

- When you need a default value (ports, listeners, replication factors, retention, converter
  settings, etc.), **read the file here first** instead of guessing.
- These files are copied unchanged from the source repo. If a value here disagrees with the
  source repo, the source repo wins — re-sync this directory.
- When generating new config for a cluster, start from the relevant file here and override only
  what you need.

## Files

| File | Purpose |
| --- | --- |
| `server.properties` | Combined KRaft node (`process.roles=broker,controller`); broker `9092`, controller `9093` |
| `broker.properties` | Broker-only node (`process.roles=broker`) |
| `controller.properties` | Controller-only node (`process.roles=controller`) |
| `producer.properties` | Default producer client settings |
| `consumer.properties` | Default consumer client settings |
| `connect-distributed.properties` | Kafka Connect distributed worker config |
| `connect-standalone.properties` | Kafka Connect standalone worker config |
| `connect-mirror-maker.properties` | MirrorMaker 2 (cross-cluster replication) config |
| `log4j2.yaml` | Logging config for Kafka server/components |
| `tools-log4j2.yaml` | Logging config for CLI tools |

## Important defaults to remember

- Broker listener: `PLAINTEXT://...:9092`; controller listener: `CONTROLLER://...:9093`.
- Internal-topic replication factors and `min.isr` are **1** in these dev configs — must be
  raised to **3+** for production (see
  [`../playbooks/kraft-cluster-setup.md`](../playbooks/kraft-cluster-setup.md)).
- `log.dirs` points at `/tmp/kraft-*` in dev — change to durable storage for real deployments.
