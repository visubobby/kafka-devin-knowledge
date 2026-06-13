# API Stability Guide

How the three `@InterfaceStability` levels map to CAB risk. These annotations are defined in
`clients/src/main/java/org/apache/kafka/common/annotation/InterfaceStability.java` and tell users
how much they can rely on a package, class, or method **not changing over time**. The annotation
on a changed public API directly determines the risk of a **breaking change** to it.

## The three levels

### `@Stable` — breaking change = **HIGH** risk
> "Compatibility is maintained in major, minor and patch releases with one exception: compatibility
> may be broken in a major release (i.e. 0.m) for APIs that have been deprecated for at least one
> major/minor release cycle. In cases where the impact of breaking compatibility is significant,
> there is also a minimum deprecation period of one year."

`@Stable` APIs carry the strongest guarantee. Users depend on these across upgrades, so breaking a
`@Stable` signature (removing/renaming a method, changing parameters or return type, removing a
field) is **HIGH** risk and generally requires deprecation first.

### `@Evolving` — breaking change = **MEDIUM** risk
> "Compatibility may be broken at minor release (i.e. m.x)."

`@Evolving` APIs are allowed to change at minor releases. Breaking one is real but expected churn,
so it maps to **MEDIUM** risk.

### `@Unstable` — breaking change = **LOW** risk
> "No guarantee is provided as to reliability or stability across any level of release
> granularity."

`@Unstable` APIs offer no compatibility guarantee at all. Breaking one is **LOW** risk.

## Unannotated public APIs default to `@Stable`

Per the source: *"This is the default stability level for public APIs that are not annotated."*
So a **public** API in `clients/` with **no** `@InterfaceStability` annotation must be treated as
`@Stable` — i.e. **breaking it is HIGH risk**. Do not assume "no annotation = no guarantee"; the
default is the strongest guarantee.

## How to apply this in review

1. Find the changed public type/method in `clients/`.
2. Look upward for an `@InterfaceStability.Stable` / `.Evolving` / `.Unstable` annotation on the
   method, its class, or its package (`package-info.java`).
3. **No annotation on a public API ⇒ treat as `@Stable`.**
4. Decide whether the change is **breaking** (signature/behavioral contract change) vs. **additive**
   (new method/overload, new optional behavior). Additive changes are much lower risk than breaking
   changes at every level.
5. Map breaking changes: `@Stable` → HIGH, `@Evolving` → MEDIUM, `@Unstable` → LOW, per
   [`risk-taxonomy.md`](risk-taxonomy.md).

## Quick reference

| Annotation | Compatibility guarantee | Breaking-change risk |
|---|---|---|
| `@Stable` (and unannotated public APIs) | Held across major/minor/patch (break only on major, after deprecation) | **HIGH** |
| `@Evolving` | May break at minor release | **MEDIUM** |
| `@Unstable` | No guarantee at any granularity | **LOW** |
