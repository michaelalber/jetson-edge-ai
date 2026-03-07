# Local Development Environment Setup

This guide covers setting up a development machine to work with this repository.
Most code can be developed and tested locally, then deployed to the Jetson.

## Requirements

| Tool | Version | Purpose |
|---|---|---|
| Python | 3.10+ | Inference pipeline development |
| .NET SDK | 10.x | Blazor dashboard development |
| Docker | 24+ | Containerized builds and local testing |
| Git | 2.x | Version control |

## Python Setup

Each project is self-contained. Example for `visual-inspection`:

```bash
cd projects/visual-inspection
python3 -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
pytest                          # run tests
```

Note: GPU-accelerated libraries (TensorRT, CUDA) only run on the Jetson or a CUDA-capable
host. CPU fallbacks are used for local development where available.

## .NET Setup

```bash
# Install .NET 10 SDK
# https://dotnet.microsoft.com/download/dotnet/10.0

cd projects/monitoring-dashboard
dotnet restore
dotnet build
dotnet test
dotnet run --project src/Dashboard
```

## Docker — Cross-platform Builds for arm64

The Jetson is `linux/arm64`. Build multi-arch images from an x86 host:

```bash
# Enable arm64 emulation (once)
docker run --privileged --rm tonistiigi/binfmt --install arm64

# Build for Jetson target
docker buildx build \
  --platform linux/arm64 \
  -t jetson-edge-ai/visual-inspection:latest \
  -f infrastructure/docker/visual-inspection/Dockerfile \
  projects/visual-inspection
```

For fast iteration, build directly on the Jetson instead.

## Connecting to the Jetson

```bash
# SSH (set up SSH key auth on first boot)
ssh user@jetson-orin-nano.local

# Copy a built image
docker save jetson-edge-ai/visual-inspection:latest | ssh user@jetson-orin-nano.local docker load

# Or use a private registry and pull on-device
```
