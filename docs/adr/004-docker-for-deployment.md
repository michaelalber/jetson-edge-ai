# 004: Docker for Project Containerization

**Status:** Accepted
**Date:** 2026-03-07

## Context

Each project has different runtime dependencies (CUDA version, Python packages, .NET runtime).
Reproducibility across development and the Jetson target device requires environment isolation.

## Decision

Each project ships a `Dockerfile` under `infrastructure/docker/<project-name>/`. Use
NVIDIA's L4T base images (`nvcr.io/nvidia/l4t-*`) for projects requiring GPU access.
Use `docker compose` for multi-container scenarios (e.g., inference pipeline + dashboard).

## Consequences

- Consistent runtime between development machine and Jetson (architecture caveat: dev is x86,
  Jetson is arm64 — use `--platform linux/arm64` and cross-compilation or build on-device)
- L4T base images are large (~2–5 GB); cache layers carefully in CI
- GPU access in containers requires `--runtime nvidia` or the NVIDIA Container Toolkit
- `docker compose` simplifies networking between services during local testing
