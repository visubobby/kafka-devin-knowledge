# kafka-devin-knowledge

A **shared Devin knowledge base** for the [`visubobby/kafka`](https://github.com/visubobby/kafka)
repository. This repo is intended to be added as a **Devin Knowledge source** so that Devin can
reference Kafka's canonical configs, enforced conventions, architecture docs, and operational
playbooks across all sessions.

> **This is a knowledge/reference repo, not a buildable project.** It contains copies of config
> and tooling files plus authored documentation. Nothing here is meant to be compiled or run on
> its own.

## Why this exists

When Devin works in `visubobby/kafka`, it should not guess at config values, version pins, or
coding conventions. This repo centralizes the answers:

- **Configs here are the canonical reference.** When working in `visubobby/kafka`, Devin should
  **check here first** before guessing at a config value, a dependency version, or a convention.
- The copied files are kept **verbatim** from the source repo; the authored files
  (`architecture/overview.md`, everything in `conventions/` and `playbooks/`, and all
  `README.md`s) summarize the rules that the copied files enforce.
- If anything here disagrees with `visubobby/kafka`, **the source repo wins** — treat that as a
  signal to re-sync this knowledge base.

## Directory map (and how each maps to the source repo)

| Directory | Mirrors / derives from | What's inside |
| --- | --- | --- |
| [`configs/`](configs/) | `visubobby/kafka/config/` | Verbatim broker/controller/server, producer/consumer, Connect, and log4j2 configs |
| [`checkstyle/`](checkstyle/) | `visubobby/kafka/checkstyle/` | Verbatim Checkstyle ruleset, suppressions, scalafmt config, and per-module import-control XMLs |
| [`gradle/`](gradle/) | `visubobby/kafka/gradle/dependencies.gradle` | Verbatim centralized dependency/version definitions |
| [`docker/`](docker/) | `visubobby/kafka/docker/` | Verbatim JVM/native Dockerfiles and container `server.properties` |
| [`architecture/`](architecture/) | `visubobby/kafka/docs/` | Authored overview + adapted design/KRaft/log/message-format docs |
| [`conventions/`](conventions/) | derived from `checkstyle/` + `gradle/` | Human-readable Java/Scala coding standards, module layering, and build conventions |
| [`playbooks/`](playbooks/) | references `bin/*.sh`, `config/*`, `docker/*.py`, `release/` | Step-by-step guides for dev setup, KRaft cluster setup, topics, Connect, Streams reset, Docker, and releases |
| [`cab/`](cab/) | derived from `message/README.md`, `MetadataVersion.java`, `InterfaceStability.java`, `gradle/dependencies.gradle`, module layout | Change Advisory Board risk-assessment kit for reviewing PRs (risk taxonomy, module criticality, protocol/rollback/deployment rules, review checklist) |

Each subdirectory has its own `README.md` explaining its purpose and how Devin should use the
files within it.

## How Devin should use this repo

1. **Before changing code** in `visubobby/kafka`, read the relevant
   [`conventions/`](conventions/) doc so the change passes Checkstyle/Spotless and the build.
2. **Before writing or generating config**, start from the matching file in
   [`configs/`](configs/) and override only what's needed.
3. **Before picking a dependency version**, look it up in
   [`gradle/dependencies.gradle`](gradle/dependencies.gradle) — and check the coordinated-bump
   notes in [`conventions/build-conventions.md`](conventions/build-conventions.md).
4. **For operational tasks** (cluster setup, topics, Connect, Streams reset, Docker builds,
   releases), follow the matching guide in [`playbooks/`](playbooks/).
5. **For architectural context**, read [`architecture/overview.md`](architecture/overview.md)
   and the adapted design docs alongside it.

## Keeping it in sync

The copied directories (`configs/`, `checkstyle/`, `gradle/`, `docker/`) are snapshots of
`visubobby/kafka`. When those files change upstream, re-copy them and update the derived
`conventions/` and `playbooks/` docs accordingly.

## Highlights worth remembering

- KRaft (no ZooKeeper): combined node uses `config/server.properties`
  (`process.roles=broker,controller`); broker `9092`, controller `9093`; run
  `bin/kafka-storage.sh format` before first start.
- Dev configs use replication factor `1` — raise to `3+` for production.
- No direct `System.exit()` — use `Exit.exit` (small allowlist applies).
- `FinalLocalVariable` is enforced only in the `streams` package; `generated/` code is exempt
  from all Checkstyle checks.
- Versions are centralized in `gradle/dependencies.gradle`; some bumps (lz4/zstd, scalafmt,
  Jetty) require coordinated edits elsewhere.
