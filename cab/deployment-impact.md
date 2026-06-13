# Deployment Impact

What deployment / upgrade choreography each change type forces. The CAB uses this to tell
operators *how* a change must be rolled out and in what order — which is part of the risk picture,
not just an afterthought. Pair this with [`rollback-constraints.md`](rollback-constraints.md) for
the reverse direction.

---

## Rolling restart required
The brokers must be bounced one at a time (preserving availability) to pick up the change.
Applies to:
- **Broker config changes** that are not dynamically updatable.
- **New feature flags** / MetadataVersion feature bumps.
- **Most `core/` changes** (broker request path, replication, partition leadership).

These are the common case for broker-side code/config PRs.

## Config-only (no restart)
Some broker configs are **dynamic** and can be changed at runtime via `kafka-configs.sh` without a
restart. If a PR only adjusts a dynamically updatable config, the rollout is config-only.
Confirm the specific config is in the dynamic set before assuming no restart.

## Offline upgrade required
The cluster (or the affected component) cannot do this as a clean rolling change and needs a more
disruptive, carefully sequenced upgrade. Applies to:
- **MetadataVersion changes with `didMetadataChange = true`** — crossing a metadata-format boundary
  is not downgradable; the upgrade is one-way (see
  [`rollback-constraints.md`](rollback-constraints.md) §1).
- **RocksDB format changes in Streams** — downgrade needs local state-store deletion, so treat the
  upgrade as format-breaking.

## Two-phase rolling upgrade
Two passes over the fleet are required. The canonical example is the **Streams serialization format
change (KIP-904 pattern)**: downgrading 3.5+ → 3.4 needs **two rolling bounces** with the
`upgrade.from` config to negotiate the older format during the transition. Format-negotiated
Streams upgrades/downgrades follow this two-bounce shape.

## Client upgrade before broker
When **old protocol API versions are removed**, clients that still speak only the removed versions
can't connect. Clients must be upgraded **before** the brokers that drop those versions, or they
will be stranded (see [`protocol-change-rules.md`](protocol-change-rules.md) and
[`rollback-constraints.md`](rollback-constraints.md) §5).

## Broker upgrade before client
When **new server-side features are added that clients opt into**, the brokers must support the
feature **before** clients try to use it. Upgrade brokers first, then roll out clients that depend
on the new server capability.

---

## Mapping change types → deployment path

| Change in the PR | Deployment path |
|---|---|
| Non-dynamic broker config / most `core/` code | Rolling restart |
| Dynamically updatable config only | Config-only via `kafka-configs.sh` |
| New feature flag / MetadataVersion bump | Rolling restart (offline upgrade if `didMetadataChange=true`) |
| MetadataVersion `didMetadataChange=true` | Offline / one-way upgrade (no metadata downgrade) |
| RocksDB version bump (Streams) | Offline upgrade; downgrade needs state-store deletion |
| Streams serialization change (KIP-904) | Two-phase rolling upgrade with `upgrade.from` |
| Protocol API version removal | Upgrade clients before brokers |
| New opt-in server-side feature | Upgrade brokers before clients |
