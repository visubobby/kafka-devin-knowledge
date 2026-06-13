# gradle/

**Verbatim copy** of `visubobby/kafka/gradle/dependencies.gradle` — the **single source of
truth** for dependency and toolchain versions used across the Kafka build.

## How Devin should use this

- Look up any dependency version here (JUnit, Mockito, Jackson, Jetty, Log4j2, RocksDB, SLF4J,
  lz4, zstd, Scala, Checkstyle, scalafmt, SpotBugs, …) rather than guessing.
- Never hardcode a version in a build file — versions are centralized here and referenced via
  `versions[...]`.
- This file also encodes **coordinated-bump invariants** in comments (e.g. bumping lz4/zstd
  requires checking `CompressionType.java`; bumping scalafmt requires updating
  `../checkstyle/.scalafmt.conf`; bumping zstd requires updating the native-image resource
  config). See [`../conventions/build-conventions.md`](../conventions/build-conventions.md) for
  the full list before upgrading anything.

## Files

| File | Purpose |
| --- | --- |
| `dependencies.gradle` | Centralized dependency/version definitions for the Gradle build |

## Selected pinned versions (verify against the file)

Scala `2.13.18` · Checkstyle `12.3.1` · scalafmt `3.10.3` · SpotBugs `4.9.8` ·
JUnit `5.14.3` · Mockito `5.23.0` · Jackson `2.21.2` · Jetty `12.0.34` · Log4j2 `2.25.4` ·
RocksDB `10.1.3` · SLF4J `1.7.36` · lz4 `1.10.2` · zstd `1.5.6-10`.
