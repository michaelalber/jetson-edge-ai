# Docker

Container definitions for each project. Named to match the project directory:

```
infrastructure/docker/
├── visual-inspection/
│   └── Dockerfile
├── sensor-pipeline/
│   └── Dockerfile
├── anomaly-detection/
│   └── Dockerfile
├── llm-inference/
│   └── Dockerfile
└── monitoring-dashboard/
    └── Dockerfile
```

## Base Images

| Use case | Base image |
|---|---|
| Python + GPU (inference) | `nvcr.io/nvidia/l4t-cuda:36.2-runtime` |
| Python (no GPU) | `python:3.10-slim` |
| .NET dashboard | `mcr.microsoft.com/dotnet/aspnet:10.0` |

## Building for arm64 (Jetson target)

```bash
# From repo root — example for visual-inspection
docker buildx build \
  --platform linux/arm64 \
  -t jetson-edge-ai/visual-inspection:latest \
  -f infrastructure/docker/visual-inspection/Dockerfile \
  projects/visual-inspection
```

See [docs/setup/development.md](../../docs/setup/development.md) for cross-compilation setup.
