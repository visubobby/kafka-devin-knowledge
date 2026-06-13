# Module Dependency Rules (Import Layering)

Kafka enforces a strict module/package layering through Checkstyle's `ImportControl` check.
Each module has its own `import-control*.xml` in [`../checkstyle/`](../checkstyle/). The base
file is [`../checkstyle/import-control.xml`](../checkstyle/import-control.xml), which opens with
a deliberate warning:

> THINK HARD ABOUT THE LAYERING OF THE PROJECT BEFORE CHANGING THIS FILE

The model is **default-deny**: an import is rejected unless an enclosing scope explicitly
`<allow>`s it. Adding a new cross-module dependency therefore requires adding an explicit
`allow` entry — which forces a conscious decision about whether the layering permits it.

## The core layering principle

Lower layers must not depend on higher layers. In particular:

- **No one depends on the server (`core`).** The base `import-control.xml` declares
  `<disallow pkg="kafka" />`. The Scala broker code lives under the `kafka.*` package, and
  nothing else in the project may import it.
- **`clients` is a foundational layer.** It must not depend on `server`, `core`, `metadata`,
  `coordinator`, or `storage`. Server-side modules depend on `clients`, not the other way
  around.
- **Public API packages are broadly usable.** `import-control.xml` allows anyone to use
  selected public packages, e.g.:
  ```xml
  <allow pkg="org.apache.kafka.common" exact-match="true" />
  <allow pkg="org.apache.kafka.common.security" />
  <allow pkg="org.apache.kafka.common.serialization" />
  <allow pkg="org.apache.kafka.common.utils" />
  <allow pkg="org.apache.kafka.common.errors" exact-match="true" />
  ```
- **Common cannot depend on clients.** Inside the `common` subpackage, `import-control.xml`
  has `<disallow pkg="org.apache.kafka.clients" />` (with a few narrowly allowed exact classes
  such as `ConsumerRecord` and `NodeApiVersions`). This keeps `common` below `clients`.
- **Widely allowed third-party/JDK packages** are declared once at the top (e.g. `java`,
  `org.slf4j`, `org.junit`, `org.mockito`, `javax.management`, security packages).

## The per-module files

| File | Module / scope |
| --- | --- |
| `import-control.xml` | Base layering for all of `org.apache.kafka` (clients/common/public APIs) |
| `import-control-core.xml` | `core` broker module |
| `import-control-metadata.xml` | `metadata` (KRaft controller, cluster metadata) |
| `import-control-server.xml` | `server` module |
| `import-control-server-common.xml` | `server-common` shared server utilities |
| `import-control-storage.xml` | `storage` (log/tiered storage) |
| `import-control-group-coordinator.xml` | group coordinator |
| `import-control-coordinator-common.xml` | shared coordinator code |
| `import-control-transaction-coordinator.xml` | transaction coordinator |
| `import-control-clients-integration-tests.xml` | clients integration tests |

(The source repo has additional ones, e.g. `import-control-share-coordinator.xml`,
`import-control-jmh-benchmarks.xml`, `import-control-examples.xml`; the ones mirrored here are
the most commonly referenced.)

## How to add a dependency the right way

1. Identify the module doing the importing → open its `import-control-<module>.xml` (or the
   base `import-control.xml` for clients/common).
2. Add a scoped `<allow pkg="..."/>` or `<allow class="..." exact-match="true"/>` in the
   correct `<subpackage>` block — keep it as narrow as possible (prefer `class` + `exact-match`
   over a broad `pkg`).
3. Confirm you are not introducing an upward dependency (e.g. clients → server). If you are,
   the design is wrong, not the import-control file.
4. Run `./gradlew checkstyleMain checkstyleTest` to verify.

Per-file `ImportControl` suppressions exist in `suppressions.xml` for a small number of files
(e.g. `RequestConvertToJson.java`, `MetadataRequestTest.java`, `BrokerRegistrationRequestTest.java`).
Do not add new ones unless absolutely necessary.
