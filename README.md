# eloq-docker

Docker images for EloqData development.

## Images

### `eloqdata/ubuntu-dev`

A lightweight Ubuntu 24.04 development image containing the build toolchain and libraries that
EloqData projects need, installed entirely via `apt` (compilers, CMake/Ninja, common C/C++ dev
libraries, Python 3, Node.js, Go, the JDK, and the Google Cloud CLI). It runs as a non-root user
`eloq` with passwordless `sudo`.

Pull it (Docker automatically selects `amd64` or `arm64` for your machine):

```sh
docker pull eloqdata/ubuntu-dev:24.04
```

Tags: `latest`, `24.04`, and `sha-<commit>`.

Build locally:

```sh
docker build -t eloqdata/ubuntu-dev ./ubuntu-dev
```

## CI

`.github/workflows/ubuntu-dev.yml` builds the image on native `amd64` and `arm64` runners:

- **Pull requests** build both architectures for validation (no login, no push).
- **Pushes to `main`** build and push each architecture, then merge them into a single multi-arch
  manifest published to DockerHub.

### Required repository secrets

| Secret | Value |
| --- | --- |
| `DOCKER_USERNAME` | DockerHub username with write access to `eloqdata/ubuntu-dev` |
| `DOCKER_PASSWORD` | That account's DockerHub password (or access token) |

Configure them in **Settings → Secrets and variables → Actions**. They are encrypted at rest and
automatically masked in build logs.
