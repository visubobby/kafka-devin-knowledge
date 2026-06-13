# playbooks/

Imperative, step-by-step guides for common operational and development tasks in
`visubobby/kafka`. Each playbook references the **actual** config files (in
[`../configs/`](../configs/)) and script names (`bin/*.sh`, `docker/*.py`) from the repo.

## How Devin should use this

- When asked to perform one of these tasks, follow the corresponding playbook as a checklist.
- Commands assume you are running from a Kafka clone/distribution (so `bin/...` and `config/...`
  resolve). The `configs/` copies in this repo are the canonical reference for those config
  files.

## Files

| File | Task |
| --- | --- |
| `local-dev-setup.md` | Build (`./gradlew build`) and test (`./gradlew test`) Kafka from source |
| `kraft-cluster-setup.md` | Stand up a KRaft cluster (`kafka-storage.sh format`, `kafka-server-start.sh`) |
| `adding-kafka-topic.md` | Create/inspect/alter/delete topics with `kafka-topics.sh` |
| `connect-deployment.md` | Deploy Kafka Connect (`connect-distributed.sh` / `connect-standalone.sh`) |
| `streams-app-reset.md` | Reset a Streams app with `kafka-streams-application-reset.sh` |
| `docker-build.md` | Build/test Docker images via `docker/docker_build_test.py` |
| `release-process.md` | Overview of the release tooling in `release/` |

## Key references

- KRaft requires `bin/kafka-storage.sh format` before first start; broker `9092`, controller `9093`.
- Dev configs use replication factor `1` — raise to `3+` for production.
