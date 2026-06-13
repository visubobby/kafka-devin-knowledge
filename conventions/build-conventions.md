# Build Conventions

Kafka builds with **Gradle** (use the wrapper: `./gradlew`). Dependency versions are pinned
centrally in [`../gradle/dependencies.gradle`](../gradle/dependencies.gradle) — treat that file
as the single source of truth and never hardcode versions elsewhere.

## Language/toolchain versions

- **Scala version: `2.13.18`** by default (`defaultScala213Version` in `dependencies.gradle`),
  configurable via `-PscalaVersion` (e.g. `./gradlew -PscalaVersion=2.13 build`). The default is
  also set in `gradle.properties` (`scalaVersion=2.13.18`).
- **Java**: the server/broker (`core`) targets **Java 17**, while the client-facing libraries
  (`clients`, `streams`) target **Java 11** for broad compatibility. Build with a JDK that can
  target these release levels (JDK 17+). Do not use APIs newer than the module's target release.
- **Checkstyle version: `12.3.1`** (overridable via `-PcheckstyleVersion`).
- **scalafmt version: `3.10.3`** — see "Coordinated version bumps" below.
- **SpotBugs version: `4.9.8`**.

## Key pinned dependency versions

From [`../gradle/dependencies.gradle`](../gradle/dependencies.gradle):

| Dependency | Version |
| --- | --- |
| JUnit (Jupiter) | `5.14.3` |
| Mockito | `5.23.0` |
| Jackson | `2.21.2` |
| Jetty | `12.0.34` |
| Log4j2 | `2.25.4` |
| RocksDB (`rocksdbjni`) | `10.1.3` |
| SLF4J | `1.7.36` |
| lz4 | `1.10.2` |
| zstd (`zstd-jni`) | `1.5.6-10` |

## Coordinated version bumps (read before upgrading)

`dependencies.gradle` contains comments encoding cross-file invariants. When upgrading these
dependencies, you must also update the related location:

- **lz4 or zstd** → verify the compression levels in
  `org.apache.kafka.common.record.internal.CompressionType` (`CompressionType.java`) are still
  valid for the new version. Additionally, bumping **zstd** requires updating
  `docker/native/native-image-configs/resource-config.json`.
- **scalafmt** → also update the `version` field in
  [`../checkstyle/.scalafmt.conf`](../checkstyle/.scalafmt.conf). The Gradle version and the
  config-file version must stay in sync.
- **Jetty** → verify the new Jetty release does **not** use the SLF4J 2.x fluent logging API.
  Kafka is on **SLF4J 1.7.x** (`1.7.36`); a Jetty version requiring SLF4J 2.x would break
  logging.

## Common Gradle tasks

- `./gradlew build` — compile, run Checkstyle/SpotBugs/Spotless, run tests, assemble.
- `./gradlew jar` / `./gradlew :clients:jar` — build jars (optionally per module).
- `./gradlew test` — run all tests; `./gradlew :core:test` for one module.
- `./gradlew checkstyleMain checkstyleTest` — run Checkstyle only.
- `./gradlew spotlessApply` / `spotlessScalaCheck` — apply/check Scala formatting.
- `./gradlew clean` — clean build outputs.

See [`../playbooks/local-dev-setup.md`](../playbooks/local-dev-setup.md) for full setup steps.
