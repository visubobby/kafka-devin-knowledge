# Playbook: Local Dev Setup

Goal: build Kafka from source and run its tests in a local clone of `visubobby/kafka`.

## Prerequisites

- A JDK that can target Java 17 (the `core`/server module needs 17; clients/streams target 11).
  Use JDK 17 or newer.
- The Gradle **wrapper** is committed — always invoke `./gradlew`, never a system `gradle`.
- Default Scala version is `2.13.18` (see
  [`../conventions/build-conventions.md`](../conventions/build-conventions.md)).

## Steps

1. Clone and enter the repo:
   ```bash
   git clone https://github.com/visubobby/kafka.git
   cd kafka
   ```
2. Build everything (compiles, runs Checkstyle/SpotBugs/Spotless, assembles jars):
   ```bash
   ./gradlew build
   ```
   To skip tests during a build: `./gradlew build -x test`.
3. Run the full test suite:
   ```bash
   ./gradlew test
   ```
   Run tests for a single module, e.g. clients or core:
   ```bash
   ./gradlew :clients:test
   ./gradlew :core:test
   ```
   Run a single test class/method:
   ```bash
   ./gradlew :clients:test --tests org.apache.kafka.common.utils.UtilsTest
   ```
4. Run static checks only (fast feedback before committing):
   ```bash
   ./gradlew checkstyleMain checkstyleTest    # Java import-control + style
   ./gradlew spotlessApply                    # auto-format Scala (scalafmt)
   ./gradlew spotlessScalaCheck               # verify Scala formatting
   ```
5. Build a runnable distribution (to start brokers locally):
   ```bash
   ./gradlew clean releaseTarGz
   ```
   Or use the `bin/` scripts directly against the build output for quick runs.

## Useful options

- Build against a specific Scala version: `./gradlew -PscalaVersion=2.13 build`.
- Override Checkstyle version: `-PcheckstyleVersion=...`.
- Clean: `./gradlew clean`.

## Conventions reminders

- Do not call `System.exit` — use `Exit.exit` (see
  [`../conventions/java-coding-standards.md`](../conventions/java-coding-standards.md)).
- Never edit files under `generated/`; regenerate via the build.
- Respect module import layering (see
  [`../conventions/module-dependency-rules.md`](../conventions/module-dependency-rules.md)).
- Dependency versions live only in
  [`../gradle/dependencies.gradle`](../gradle/dependencies.gradle).
