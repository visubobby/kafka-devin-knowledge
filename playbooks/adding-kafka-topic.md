# Playbook: Adding a Kafka Topic

Goal: create, inspect, alter, and delete topics with `bin/kafka-topics.sh`. Assumes a running
cluster (see [`kraft-cluster-setup.md`](kraft-cluster-setup.md)) with a broker on `localhost:9092`.

## Create a topic

```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 \
  --create \
  --topic my-topic \
  --partitions 3 \
  --replication-factor 1
```

- `--partitions` controls parallelism/throughput. Choose based on expected consumer parallelism.
- `--replication-factor` must be ≤ number of brokers. Dev configs run a single broker, so `1`
  is the max locally; use **3** in production.
- Add per-topic overrides with `--config`, e.g.:
  ```bash
  --config retention.ms=604800000 --config cleanup.policy=delete
  ```

## List topics

```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

## Describe a topic (partitions, leaders, ISR)

```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic my-topic
```

## Increase partitions

Partition count can only be increased, never decreased:

```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 \
  --alter --topic my-topic --partitions 6
```

## Delete a topic

```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic my-topic
```

## Quick produce/consume smoke test

```bash
# produce (type messages, Ctrl-D to end)
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-topic

# consume from the beginning
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic my-topic --from-beginning
```

## Notes

- Always pass `--bootstrap-server` (the broker address); the old `--zookeeper` flag does not
  exist in KRaft mode.
- To set client defaults (security, etc.), pass `--command-config <properties-file>`; see
  [`../configs/producer.properties`](../configs/producer.properties) and
  [`../configs/consumer.properties`](../configs/consumer.properties) for reference settings.
