# eloq-docker

Docker images for EloqData development.

## Images

### `eloqdata/ubuntu-dev`

An Ubuntu 24.04 development image containing the build toolchain and libraries that
EloqData projects need: compilers, CMake/Ninja, common C/C++ dev libraries, Python 3,
Python 2.7 for legacy MongoDB 4.0 build tooling, Node.js, Go, the JDK, and the Google Cloud CLI.
It also includes the Python test environments and RustFS 1.0.0 for local S3 tests.
It runs as a non-root user `eloq` with passwordless `sudo`.

RustFS is installed at `/usr/local/bin/rustfs` from the upstream amd64/arm64 release
archives, with pinned SHA256 checksums. Consumers can start `rustfs server <data-dir>`
without downloading or extracting it during tests; the image does not start a server
automatically or include credentials. Its Apache-2.0 license is included at
`/usr/local/share/licenses/rustfs/LICENSE`; the source is
[RustFS 1.0.0](https://github.com/rustfs/rustfs/tree/1.0.0).

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
