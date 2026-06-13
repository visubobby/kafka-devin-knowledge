# conventions/

Human-readable coding and build conventions for `visubobby/kafka`, **derived from** the actual
enforced configuration in [`../checkstyle/`](../checkstyle/) and
[`../gradle/dependencies.gradle`](../gradle/dependencies.gradle). These explain the *intent*
behind the rules; the XML/gradle files are the ground truth.

## How Devin should use this

- Read the relevant file here **before** writing or modifying code in `visubobby/kafka`, so your
  changes pass `checkstyle`, `spotless`, and the build on the first try.
- If a convention here ever conflicts with the source `checkstyle/`/`gradle/` files, the source
  files win — update these docs to match.

## Files

| File | Purpose |
| --- | --- |
| `java-coding-standards.md` | Java rules: `System.exit` ban + allowlist, `FinalLocalVariable` scoping, `generated/` exemption, import control, locale/complexity rules |
| `scala-coding-standards.md` | Scala formatting via scalafmt (`3.10.3`, Scala 2.13) and its rewrite rules |
| `module-dependency-rules.md` | The per-module import-control layering model (clients can't depend on server, etc.) |
| `build-conventions.md` | Gradle/toolchain conventions, pinned versions, coordinated version-bump invariants |

## The highlights

- **No `System.exit()`** — use `Exit.exit`; only allowed in `Exit.java`, `MessageGenerator.java`,
  the `Verifiable*` tools, and the Streams demo apps.
- **`FinalLocalVariable`** is enforced **only** in the `streams` package.
- **`generated/`** code is exempt from all checks — never hand-edit it.
- **Imports** are default-deny per module; new cross-module imports need explicit `allow` entries.
- **Versions** are centralized in `gradle/dependencies.gradle`; several bumps require coordinated
  edits elsewhere (lz4/zstd → `CompressionType.java`; scalafmt → `.scalafmt.conf`; Jetty →
  SLF4J 1.7.x compatibility).
