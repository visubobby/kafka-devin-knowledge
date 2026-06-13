# Java Coding Standards

Rules derived from [`../checkstyle/checkstyle.xml`](../checkstyle/checkstyle.xml),
[`../checkstyle/suppressions.xml`](../checkstyle/suppressions.xml), and the
`import-control*.xml` files in [`../checkstyle/`](../checkstyle/). These are enforced by
Checkstyle during the Gradle build (`./gradlew checkstyleMain checkstyleTest`). When in doubt,
the canonical rule is whatever the XML in this repo's `checkstyle/` directory says.

## No direct `System.exit()`

Calling `System.exit(...)` directly is **banned**. The Checkstyle rule `dontUseSystemExit`
(a `Regexp` module matching `System\.exit` in `checkstyle.xml`) fails the build with:

> 'System.exit': Should not directly call System.exit, but Exit.exit instead.

Use `org.apache.kafka.common.utils.Exit.exit(...)` instead. `Exit` indirects through a
swappable procedure so tests can intercept exits instead of killing the JVM.

The ban is suppressed (i.e., `System.exit` IS allowed) only in these files, per
`suppressions.xml`:

- `Exit.java` — the designated wrapper that legitimately calls `System.exit`.
- `MessageGenerator.java` — code-generation entry point.
- Tools/verifiers run as standalone processes: `VerifiableConsumer.java`,
  `VerifiableProducer.java`, `VerifiableShareConsumer.java` (also `DumpLogSegmentsTest.java`).
- Streams demo/example apps: `PageViewTypedDemo.java`, `PipeDemo.java`, `TemperatureDemo.java`,
  `WordCountDemo.java`, `WordCountProcessorDemo.java`, `WordCountTransformerDemo.java`.

Do **not** add new `System.exit` calls elsewhere, and do not add new suppression entries
without strong justification.

## `FinalLocalVariable` — only enforced in the `streams` package

The `FinalLocalVariable` check is **globally suppressed except** for code under the Kafka
Streams package. From `suppressions.xml`:

```xml
<!-- suppress FinalLocalVariable outside of the streams package. -->
<suppress checks="FinalLocalVariable"
          files="^(?!.*[\\/]org[\\/]apache[\\/]kafka[\\/]streams[\\/].*$)"/>
```

Practical implications:

- In `org.apache.kafka.streams.*` code: local variables that are never reassigned **must** be
  declared `final`.
- Everywhere else: marking locals `final` is optional (neither required nor flagged).
- A few specific Streams files additionally suppress `FinalLocalVariable`
  (e.g. `Murmur3.java`, `Murmur3Test.java`, and some assignor/join classes) — do not rely on
  these; treat `final` as required when adding new Streams code.

## Generated code is exempt from all Checkstyle checks

Anything under a `/generated/` directory is fully exempt. From `suppressions.xml`:

```xml
<suppress checks="[a-zA-Z0-9]*" files="[\\/]generated[\\/]"/>
```

Never hand-edit files under `generated/` to satisfy Checkstyle (or for any other reason) —
they are produced by the message/code generators. Regenerate via the build instead.

## Import layering is enforced per module (`ImportControl`)

The `ImportControl` check enforces which packages each module may import, using a separate XML
per module:

- `import-control.xml` — base/default layering for `org.apache.kafka`
- `import-control-core.xml` — `core` (broker) module
- `import-control-metadata.xml` — `metadata` (KRaft) module
- `import-control-server.xml`, `import-control-server-common.xml` — server modules
- `import-control-storage.xml` — storage module
- `import-control-group-coordinator.xml`, `import-control-coordinator-common.xml`,
  `import-control-transaction-coordinator.xml` — coordinator modules
- `import-control-clients-integration-tests.xml` — clients integration tests

Cross-module imports must be **explicitly allowed** via `<allow .../>` entries; everything
else is denied. For example, the base `import-control.xml` declares `<disallow pkg="kafka" />`
("no one depends on the server"). When adding a new dependency direction between modules, you
must add an explicit `allow` entry to the relevant XML — and you should think hard about
whether the layering actually permits it. See
[`module-dependency-rules.md`](module-dependency-rules.md) for the layering model.

A handful of files suppress `ImportControl` individually (e.g. `RequestConvertToJson.java`,
`MetadataRequestTest.java`); do not add new suppressions casually.

## Other notable Checkstyle rules

- **Locale-sensitive methods**: `String.toLowerCase()` / `toUpperCase()` with no `Locale`
  argument are banned (`Regexp` for `\.to(Lower|Upper)Case\(\)`). Always pass an explicit
  `Locale` (e.g. `Locale.ROOT`).
- **Complexity limits**: `MethodLength` (max 170 lines), `ParameterNumber`, `CyclomaticComplexity`,
  `NPathComplexity`, `ClassFanOutComplexity`, `ClassDataAbstractionCoupling`, etc. are enforced.
  Specific legacy classes are suppressed in `suppressions.xml`; prefer refactoring over adding a
  new suppression.
- **Java header**: every source file must start with the ASF license header
  ([`../checkstyle/java.header`](../checkstyle/java.header)).
