# cab/

**Change Advisory Board (CAB) risk-assessment kit** for pull requests against
[`visubobby/kafka`](https://github.com/visubobby/kafka).

This directory gives a CAB member (or Devin acting as one) everything needed to look at a PR
diff and assign a defensible **LOW / MEDIUM / HIGH** risk level, explain the blast radius,
and call out rollback / deployment constraints — without having to rediscover Kafka's
compatibility rules each time.

Everything here is **derived from the source repo** (`message/README.md`,
`MetadataVersion.java`, `InterfaceStability.java`, `gradle/dependencies.gradle`, and the
module layout). If anything here disagrees with `visubobby/kafka`, **the source repo wins** —
treat that as a signal to re-sync this kit.

## Files

| File | Purpose |
| --- | --- |
| [`risk-taxonomy.md`](risk-taxonomy.md) | The classification rules: which paths/changes are HIGH, MEDIUM, or LOW risk |
| [`module-criticality.md`](module-criticality.md) | Ranked list of every module with its criticality and blast radius |
| [`protocol-change-rules.md`](protocol-change-rules.md) | What counts as an incompatible (HIGH) vs. safe (LOW) protocol change |
| [`rollback-constraints.md`](rollback-constraints.md) | Scenarios where rollback is blocked or multi-step |
| [`api-stability-guide.md`](api-stability-guide.md) | How `@InterfaceStability` levels map to risk |
| [`pr-review-checklist.md`](pr-review-checklist.md) | The step-by-step CAB checklist to run on each PR |
| [`deployment-impact.md`](deployment-impact.md) | Deployment/upgrade choreography each change type forces |

## How a CAB member should use this

1. **Start with the diff.** Get the list of files the PR touches (`git diff --name-only` against
   the merge base, or the PR "Files changed" tab).
2. **Run the checklist.** Walk [`pr-review-checklist.md`](pr-review-checklist.md) top to bottom.
   It tells you which of the other docs to consult at each step.
3. **Assign a risk level.** Use [`risk-taxonomy.md`](risk-taxonomy.md) as the rulebook. The
   highest-matching rule wins — a PR that touches one HIGH path is HIGH overall, even if the
   rest is trivial.
4. **Write the justification.** For every PR, record: the risk level, which rule(s) triggered
   it, the affected modules and their blast radius, and any rollback/deployment constraints.
5. **Flag the blockers.** If [`rollback-constraints.md`](rollback-constraints.md) shows the
   change is irreversible (e.g. a `didMetadataChange=true` MetadataVersion entry, a RocksDB
   bump), say so explicitly and describe the required deployment path from
   [`deployment-impact.md`](deployment-impact.md).

## The one-paragraph version

If a PR touches the **wire protocol** (`message/*.json`), **MetadataVersion**, the
**KRaft/metadata/core/storage** critical path, a **`@Stable` client API**, **security code**,
or a **native compression / RocksDB / Jetty** dependency, it is **HIGH** risk — assume it
needs careful review and may not be cleanly reversible. Config defaults, `@Evolving` APIs,
coordinators, Connect, and Streams are **MEDIUM**. Test-only, tooling, docs, and `@Unstable`
changes are **LOW**. When in doubt, escalate one level and explain why.
