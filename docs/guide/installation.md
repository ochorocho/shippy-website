---
title: Installation
description: Install Shippy with Homebrew, go install, a prebuilt binary, or the Docker image.
---

# Installation

Shippy is a single static binary. Pick whichever channel fits your machine or pipeline — they all
produce the same `shippy` executable.

## Using Homebrew

```bash
brew tap ochorocho/shippy https://github.com/ochorocho/shippy
brew trust ochorocho/shippy
brew install shippy
```

To upgrade later:

```bash
brew upgrade shippy
```

The formula installs a prebuilt binary for your architecture, so there is no Go toolchain involved.

## From Source

```bash
git clone https://github.com/ochorocho/shippy.git
cd shippy
go build -o shippy
sudo mv shippy /usr/local/bin/
```

## Using Go Install

```bash
go install github.com/ochorocho/shippy@latest
```

## Prebuilt binaries

Every [release](https://github.com/ochorocho/shippy/releases) ships binaries with a matching
`.checksum` file for macOS and Linux, on both `amd64` and `arm64`:

| File | Platform |
| --- | --- |
| `shippy-darwin-arm64` | macOS, Apple Silicon |
| `shippy-darwin-amd64` | macOS, Intel |
| `shippy-linux-arm64` | Linux, arm64 |
| `shippy-linux-amd64` | Linux, x86_64 |

```bash
curl -sSL -o shippy \
  https://github.com/ochorocho/shippy/releases/latest/download/shippy-linux-amd64
chmod +x shippy
sudo mv shippy /usr/local/bin/
```

## Docker

A prebuilt image ships `shippy` on `PATH`, which is the easiest way to run it inside a pipeline:

```bash
docker run --rm ghcr.io/ochorocho/shippy:latest shippy --help
```

The image is published to both GitHub Container Registry (`ghcr.io/ochorocho/shippy`) and Docker Hub
(`ochorocho/shippy`). See [CI/CD](./ci-cd) for pipeline examples.

## Verify the installation

```bash
shippy --help
```

Then continue with the [Quick Start](./quick-start).
