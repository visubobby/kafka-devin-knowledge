# docker/

**Verbatim copies** of Kafka's Docker build inputs from `visubobby/kafka/docker/`. Kafka
publishes two image flavors: a **JVM** image and a GraalVM **native** image.

## How Devin should use this

- Reference these Dockerfiles when building/debugging Kafka container images or when reasoning
  about how the official images are constructed.
- The actual build/test/release **scripts** (`common.py`, `docker_build_test.py`,
  `docker_release.py`, etc.) live in `visubobby/kafka/docker/` — run them from there. See the
  step-by-step guide in [`../playbooks/docker-build.md`](../playbooks/docker-build.md).

## Files

| File | Purpose |
| --- | --- |
| `jvm/Dockerfile` | JVM-based Apache Kafka image |
| `native/Dockerfile` | GraalVM native Apache Kafka image |
| `server.properties` | Default broker config baked into / used by the container |

## Quick reference

```bash
# from visubobby/kafka/docker
python docker_build_test.py kafka/test --image-tag=<tag> --image-type=<jvm|native> \
  --kafka-url=<tarball-url>
```

Use `--build`/`-b` to only build, `--test`/`-t` to only test. Requires Docker with `buildx`.
