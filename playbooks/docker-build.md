# Playbook: Building & Testing Docker Images

Goal: build and test Kafka Docker images locally. Kafka ships two image types:

- **`jvm`** — JVM-based image (Dockerfile mirrored at
  [`../docker/jvm/Dockerfile`](../docker/jvm/Dockerfile)).
- **`native`** — GraalVM native image (Dockerfile mirrored at
  [`../docker/native/Dockerfile`](../docker/native/Dockerfile)).

All scripts live in the `docker/` directory of `visubobby/kafka`. Run them from there. They
require Python (install deps with `pip install -r docker/requirements.txt`) and Docker with
`buildx` enabled.

## Key scripts

- `docker/docker_build_test.py` — builds **and** tests an image. Shared build/test helpers live
  in `docker/common.py`.
- `docker/docker_release.py` — builds a multi-arch image and pushes it to a registry.
- `docker/docker_official_image_build_test.py` — builds/tests the Docker Official Image variant.

## Build + test from a published tarball URL

```bash
cd docker
# JVM image
python docker_build_test.py kafka/test --image-tag=3.6.0 --image-type=jvm \
  --kafka-url=https://archive.apache.org/dist/kafka/3.6.0/kafka_2.13-3.6.0.tgz

# Native image
python docker_build_test.py kafka/test --image-tag=3.8.0 --image-type=native \
  --kafka-url=https://archive.apache.org/dist/kafka/3.8.0/kafka_2.13-3.8.0.tgz
```

- First positional arg is the image name (`<namespace>/<name>`).
- `--image-tag` is the resulting tag; `--image-type` is `jvm` or `native`.
- By default the script **builds and tests**. Use `--build` / `-b` to only build, or
  `--test` / `-t` to only test an existing image.
- Run `python docker_build_test.py --help` for full options.

## Build + test from a local build archive

First produce a tarball from source (`./gradlew clean releaseTarGz`), then:

```bash
cd docker
python docker_build_test.py kafka/test --image-tag=local-build --image-type=jvm \
  --kafka-archive=/absolute/path/to/core/build/distributions/kafka_2.13-<version>.tgz
```

## Where tests live

The integration/sanity tests exercised by `docker_build_test.py` are under `docker/test/`
(driven via `common.py`). They start the built image and validate it can run Kafka. Inspect that
directory to understand or extend coverage.

## Notes

- `buildx` is required for multi-architecture builds; transient buildx errors can usually be
  fixed by retrying.
- Use the Scala 2.13 binary tarball (matches Kafka's default Scala version).
- The default in-container broker config is mirrored at
  [`../docker/server.properties`](../docker/server.properties).
