---
layout: default
title: Deployment
nav_order: 6
permalink: /deployment/
---

# Deployment Guide

OpenBQ runs as a Docker container. This guide covers how to run the backend service image directly, for servers, pipelines and custom setups. If you just want to assess a folder of images on your own machine, the [bqConnect CLI]({% link getting_started/bqconnectcli.md %}) does all of this for you.

## How the components fit together

| Component | What it is | Where it runs |
|---|---|---|
| **bqConnect CLI** (`pip install openbq`) | Lightweight Python launcher | Your machine |
| **bqCore-Service** | Docker image with the command-line service and all assessment engines | Docker container |
| **bqCore** | Python module that wraps the quality engines (OBQE, OFIQ, BIQT, NFIQ2, NISQA) | Inside the bqCore-Service image |

The CLI pulls the bqCore-Service image, mounts your input and output folders, and runs the assessment inside the container.

## System requirements

| Requirement | Details |
|---|---|
| Docker | Docker Engine (Linux) or Docker Desktop (macOS, Windows with WSL 2) |
| CPU architecture | `amd64` (x86-64) or `arm64` (Apple Silicon, AWS Graviton) |
| Memory | [MINIMUM RAM] minimum, [RECOMMENDED RAM] recommended |
| Disk | [DISK SPACE] free for the image, plus space for your datasets and results |
| Python (CLI only) | 3.10 to 3.14 |

## Get the image

Images are published to the GitHub Container Registry:

```
ghcr.io/open-source-biometric-quality-framework/bqcore-service
```

### Image tags

| Tag | Use |
|---|---|
| `latest` | Most recent release. The CLI uses this by default. |
| `vX.Y.Z` (e.g. `v0.1.3`) | A specific release. Use a fixed version for production and reproducible results. |

See all published versions on the [package page](https://github.com/Open-Source-Biometric-Quality-Framework/bqCore-Service/pkgs/container/bqcore-service) and the [release notes]({% link index/releasenotes.md %}).

```bash
docker pull ghcr.io/open-source-biometric-quality-framework/bqcore-service:latest
```

### Multi-architecture support

Each release is built for `linux/amd64` and `linux/arm64` and published as a single multi-arch image. Docker automatically pulls the right version for your machine, so no extra flags are needed.

To check which architecture you pulled:

```bash
docker image inspect --format '{{.Architecture}}' \
  ghcr.io/open-source-biometric-quality-framework/bqcore-service:latest
```

## Run the service

The image's entrypoint is `bash -l -c`, so the command you pass is a single quoted string. The working directory inside the container is `/app`.

### Verify the installation

```bash
docker run --rm -it --shm-size=8g \
  ghcr.io/open-source-biometric-quality-framework/bqcore-service:latest \
  "python3 -m openbq --benchmark"
```

This runs a benchmark on the sample data bundled in the image. To show engine and version info instead, use `"python3 -m openbq --info"`.

### Assess your own data

Mount your data folder into `/app` and point `--input` at it:

```bash
docker run --rm -it --shm-size=8g \
  -v "$(pwd)/data/face:/app/data/face" \
  ghcr.io/open-source-biometric-quality-framework/bqcore-service:latest \
  "python3 -m openbq --mode face --input data/face --report"
```

Results are written to a timestamped folder inside the input folder (e.g. `data/face/1730000000/`), so they appear on your machine when the container exits:

| File | Contents |
|---|---|
| `output_<mode>_<engine>_<timestamp>.csv` | Quality scores per file |
| `log_<mode>_<engine>_<timestamp>.json` | Run log |
| `report_<mode>_<engine>_<timestamp>.html` | Report (with `--report`) |

To write results somewhere else, mount a second folder and add `--output`:

```bash
docker run --rm -it --shm-size=8g \
  -v "$(pwd)/data/face:/app/data/face" \
  -v "$(pwd)/results:/app/results" \
  ghcr.io/open-source-biometric-quality-framework/bqcore-service:latest \
  "python3 -m openbq --mode face --input data/face --output results"
```

On Windows PowerShell, replace `$(pwd)` with `${PWD}`.

### Volume mounts

| Flag | Purpose |
|---|---|
| `-v <host-folder>:/app/<folder>` | Makes your input data available to the container. Use the same `<folder>` in `--input`. |
| `-v <host-folder>:/app/<folder>` | Optional separate output folder. Use the same `<folder>` in `--output`. |

The container runs as a non-root user (`assessor`). On Linux, the mounted output folder (or the input folder, if results are written there) must be writable by that user.

### Shared memory (`--shm-size`)

OpenBQ processes files in parallel, and the workers share data through shared memory (`/dev/shm`). Docker's default of 64 MB is too small and can cause failures or slow runs on large datasets.

| Setting | Value |
|---|---|
| Recommended | Half of the host's total RAM |
| Fallback | `8g` |

The bqConnect CLI sets this automatically to half of your RAM (falling back to 8 GB if it can't detect memory). To override it with the CLI:

```bash
openbq --shm 4096MB --mode face --input data/face
```

## Command options

Common options passed to `python3 -m openbq` inside the container (or directly to the `openbq` CLI):

| Option | Description | Default |
|---|---|---|
| `--mode`, `-M` | Modality: `fingerprint`, `face`, `iris`, `speech` | `face` |
| `--input`, `-I` | Input folder or CSV file | `data/` |
| `--output`, `-O` | Output folder | Timestamped folder in input |
| `--report`, `-R` | Generate a report | Off |
| `--engine`, `-E` | Face engine: `OBQE`, `OFIQ`, `BIQT`, `Fusion` | `OBQE` |
| `--limit`, `-L` | Maximum number of files to scan | No limit |
| `--filename`, `-F` | Filename pattern to match | `*` |
| `--type` | File types to process, e.g. `jpg,png` | All supported |
| `--benchmark`, `-B` | Run system benchmark | Off |
| `--info` | Show engine information | Off |
| `--debug` | Print runtime logs | Off |

Run `"python3 -m openbq --help"` for the full list.

## Using a specific version with the CLI

The bqConnect CLI uses the `latest` image by default. To pin a version or use a custom image:

```bash
openbq --tag ghcr.io/open-source-biometric-quality-framework/bqcore-service:v0.1.3 --mode face --input data/face
```

## Upgrading

```bash
# With the CLI
openbq --update

# With Docker
docker pull ghcr.io/open-source-biometric-quality-framework/bqcore-service:latest
```

To remove the CLI and the image: `openbq --uninstall`.
