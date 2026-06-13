# Scala Coding Standards

Kafka's `core` module (and some other modules) contain Scala code, formatted with **scalafmt**.
The canonical configuration is [`../checkstyle/.scalafmt.conf`](../checkstyle/.scalafmt.conf).
Formatting is enforced via the Spotless Gradle plugin
(`./gradlew spotlessScalaCheck` / `spotlessApply`).

## scalafmt configuration

From `.scalafmt.conf`:

```hocon
version = 3.10.3
runner.dialect = scala213
docstrings.style = Asterisk
docstrings.wrap = false
maxColumn = 120
continuationIndent.defnSite = 2
assumeStandardLibraryStripMargin = true
rewrite.rules = [SortImports, RedundantBraces, RedundantParens, SortModifiers]
```

What this means in practice:

- **scalafmt version `3.10.3`.** The version is pinned in the config. The Gradle build reads
  the same scalafmt version from [`../gradle/dependencies.gradle`](../gradle/dependencies.gradle)
  (`scalafmt: "3.10.3"`). **If you bump scalafmt, you must update BOTH** the `version` field in
  `.scalafmt.conf` and the `scalafmt` entry in `dependencies.gradle` — they must match.
- **Scala 2.13 dialect** (`runner.dialect = scala213`). Kafka builds against Scala
  `2.13.18` by default (see [`build-conventions.md`](build-conventions.md)). Do not use Scala 3
  syntax.
- **Max line width is 120 columns.** Break long lines.
- **Docstrings** use the asterisk (`*`) style and are not auto-wrapped.
- **`stripMargin`** is assumed to be the standard-library form (affects multi-line string
  indentation handling).
- **Rewrite rules** are applied automatically by `spotlessApply`:
  - `SortImports` — imports are sorted.
  - `RedundantBraces` — unnecessary braces are removed.
  - `RedundantParens` — unnecessary parentheses are removed.
  - `SortModifiers` — modifiers (e.g. `private`, `final`, `override`) are ordered consistently.

## Workflow

- Run `./gradlew spotlessApply` to auto-format Scala before committing.
- Run `./gradlew spotlessScalaCheck` (or the full `check`) to verify formatting in CI.
- Let scalafmt own formatting — do not hand-format Scala against these rules; run the tool.
- The ASF license header is required on every Scala file as well (Spotless enforces it).
