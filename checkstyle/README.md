# checkstyle/

**Verbatim copies** of Kafka's static-analysis configuration from `visubobby/kafka/checkstyle/`.
These define the **enforced** Java style/quality rules, Scala formatting, and per-module import
layering. They are the source of truth behind the human-readable rules in
[`../conventions/`](../conventions/).

## How Devin should use this

- Before writing/modifying Java or Scala in `visubobby/kafka`, check the relevant file here so
  generated code passes the build's `checkstyle`/`spotless` tasks.
- When unsure whether something is allowed (an import, a `System.exit`, a `final`), the XML here
  is authoritative. The `conventions/` docs summarize it; this directory is the ground truth.

## Files

| File | Purpose |
| --- | --- |
| `checkstyle.xml` | Main Checkstyle ruleset (style, complexity, `dontUseSystemExit`, locale checks, etc.) |
| `suppressions.xml` | Per-file/per-pattern suppressions (e.g. `generated/` exemption, `FinalLocalVariable` scoping) |
| `.scalafmt.conf` | scalafmt config for Scala formatting (version `3.10.3`, Scala 2.13) |
| `java.header` | Required ASF license header for Java source files |
| `import-control.xml` | Base import layering for `org.apache.kafka` |
| `import-control-core.xml` | `core` (broker) module import rules |
| `import-control-metadata.xml` | `metadata` (KRaft) module import rules |
| `import-control-server.xml` / `import-control-server-common.xml` | server module import rules |
| `import-control-storage.xml` | storage module import rules |
| `import-control-group-coordinator.xml` / `import-control-coordinator-common.xml` / `import-control-transaction-coordinator.xml` | coordinator module import rules |
| `import-control-clients-integration-tests.xml` | clients integration test import rules |

## Key rules (summarized — see `../conventions/`)

- No direct `System.exit()` (use `Exit.exit`), except in a small allowlist of files.
- `FinalLocalVariable` is only enforced inside the `streams` package.
- Everything under `/generated/` is exempt from all checks.
- Imports are default-deny per module; cross-module imports must be explicitly allowed.
