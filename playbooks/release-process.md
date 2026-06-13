# Playbook: Release Process (Overview)

Goal: orient Devin on how Apache Kafka releases are produced. The tooling lives in the
`release/` directory of `visubobby/kafka`. This is a high-level overview — the authoritative,
full instructions are the upstream
[Kafka Release Process wiki](https://cwiki.apache.org/confluence/display/KAFKA/Release+Process)
referenced from `release/README.md`.

## Requirements (from `release/README.md`)

- Python `3.12`
- `git`
- `gpg` `2.4` (releases are GPG-signed)

## Setup

```bash
cd release
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Point `PUSH_REMOTE_NAME` at the git remote for the canonical repo:

```bash
export PUSH_REMOTE_NAME=$(git remote -v | grep -w 'github.com' \
  | grep -w 'apache/kafka' | grep -w '(push)' | awk '{print $1}')
```

## Running a release

```bash
source .venv/bin/activate
python release.py
```

The `release/` directory is organized as:

- `release.py` — the main driver script.
- `runtime.py`, `preferences.py`, `git.py`, `gpg.py`, `svn.py`, `notes.py`, `templates.py`,
  `textfiles.py` — supporting modules the driver uses (git tagging, GPG signing, SVN upload to
  the Apache dist area, release notes generation, templating).
- `requirements.txt` — Python dependencies for the release tooling.

## Recovery notes

- The script saves previously entered inputs in `release/.release-settings.json`, so you can
  correct values on re-run.
- If the script is interrupted, you may need to manually delete the tag named after the release
  candidate and the branch named after the release version before retrying.

## Related

- Docker release images are built/pushed separately — see
  [`docker-build.md`](docker-build.md) and `docker/docker_release.py`.
