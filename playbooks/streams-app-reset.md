# Playbook: Kafka Streams Application Reset

Goal: reset the state of a Kafka Streams application so it can reprocess input from a clean
slate. Use the **application reset tool**, `bin/kafka-streams-application-reset.sh`.

## What "reset" does

A Streams app is identified by its `application.id`. The reset tool prepares the app for
reprocessing by:

- Resetting **input topic** offsets for the app's consumer group (to earliest, or a specified
  position).
- Skipping to the end of **intermediate** (through) topics.
- Deleting internal topics created by the app (repartition and changelog topics whose names are
  prefixed by the `application.id`).

It does **not** delete local state store directories — you must remove those on each app
instance separately (see below).

## Prerequisites (IMPORTANT)

- **Stop all running instances** of the Streams application first. Running instances will
  recreate state and interfere with the reset.
- Know the `application.id` of the app and the broker address.

## Steps

1. Stop every instance of the application.
2. Run the reset tool:
   ```bash
   bin/kafka-streams-application-reset.sh \
     --bootstrap-server localhost:9092 \
     --application-id my-streams-app \
     --input-topics input-topic-1,input-topic-2 \
     --intermediate-topics through-topic-1
   ```
   Useful options:
   - `--to-earliest` / `--to-latest` / `--to-offset <n>` / `--to-datetime <ISO8601>` /
     `--by-duration <PnDTnHnMnS>` — where to reset input offsets to.
   - `--dry-run` — show what would happen without making changes (run this first).
   - `--config-file <props>` — pass client/security settings.
3. **Delete local state** on each application instance (the reset tool cannot do this remotely).
   In your application code call `KafkaStreams#cleanUp()` before `start()`, **or** manually
   delete the state directory: `${state.dir}/<application.id>` (default `state.dir` is
   `/tmp/kafka-streams`).
4. Restart the application instances. It will reprocess input from the reset position and
   rebuild internal state.

## Notes

- Always do a `--dry-run` first to confirm which topics/offsets will be affected.
- Resetting is destructive to derived state; make sure downstream consumers expect reprocessed
  output (it may produce duplicates).
