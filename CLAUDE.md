# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

Learning repository for Edge AI and Industrial Automation on the NVIDIA Jetson Orin Nano (arm64, 8 GB unified memory, JetPack 6.x / L4T r36). Projects are self-contained and developed independently.

## Project Layout

```
projects/
  visual-inspection/    # Python — camera + TensorRT defect detection
  sensor-pipeline/      # Python — I2C/SPI/GPIO sensor ingestion
  anomaly-detection/    # Python — statistical anomaly detection on sensor streams
  llm-inference/        # Python — Ollama on-device LLM and RAG
  monitoring-dashboard/ # .NET 10 Blazor Server — live metrics UI
infrastructure/
  docker/               # Per-project Dockerfiles (L4T base images for GPU projects)
  scripts/              # Jetson provisioning and deploy scripts
models/                 # Configs and conversion scripts only — weights are gitignored
docs/adr/               # Architecture Decision Records (read before making new decisions)
```

Each Python project is self-contained with its own `pyproject.toml` and `.venv`. The .NET project has its own `.sln` and `.csproj`.

## Development Commands

### Python projects (run from within a project directory)

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

pytest                          # all tests
pytest tests/test_foo.py::test_bar  # single test
ruff check src/ tests/          # lint
mypy src/                       # type check
```

### .NET dashboard (run from `projects/monitoring-dashboard/`)

```bash
dotnet restore
dotnet build
dotnet test
dotnet test --filter "FullyQualifiedName~SomeTest"   # single test
dotnet run --project src/Dashboard
```

### Docker — building for Jetson (arm64)

```bash
# Enable arm64 emulation once on an x86 host
docker run --privileged --rm tonistiigi/binfmt --install arm64

# Build a project image
docker buildx build \
  --platform linux/arm64 \
  -t jetson-edge-ai/<project-name>:latest \
  -f infrastructure/docker/<project-name>/Dockerfile \
  projects/<project-name>
```

## Architecture Decisions (ADRs)

Before making a new technology or pattern decision, read `docs/adr/`. Active decisions:

- **ADR-001** Python 3.10+ with `pyproject.toml` for all inference/sensor pipelines
- **ADR-002** .NET 10 Blazor Server (SignalR) for the monitoring dashboard
- **ADR-003** Ollama for on-device LLM; GGUF quantized models sized to fit in ~6 GB
- **ADR-004** Docker with NVIDIA L4T base images; `--platform linux/arm64` for cross-builds

## Key Constraints

- **Memory:** 8 GB unified — CPU and GPU share it. LLM models must leave headroom for OS and inference runtime.
- **Architecture:** Jetson is `linux/arm64`. GPU libraries (TensorRT, CUDA) are only available on-device or in L4T containers.
- **Python GPU libs:** On a dev machine, expect CPU-only fallbacks or skip GPU-dependent tests.
- **Model weights:** Never commit weights (`.pt`, `.onnx`, `.engine`, `.gguf`, `.bin`, `.safetensors` are gitignored). Use download scripts under `models/download/`.
- **`shared/`:** Only add code here when two or more projects genuinely share it — not speculatively.

## Python Standards (for this repo)

- Type hints on all function signatures; Pydantic v2 for validation and settings
- `pathlib.Path` over `os.path`
- `async def` / `asyncio` for I/O-bound work (sensor polling, HTTP calls)
- Prompt templates live in versioned files under a `prompts/` directory — not inline f-strings
- Pin all ML/AI dependency versions explicitly in `pyproject.toml`

## .NET Standards (for this repo)

- .NET 10, nullable reference types enabled, warnings as errors
- Blazor Server with SignalR for real-time metric streaming
- Vertical slice feature folders under `Features/` — not layer folders
- `CancellationToken` propagated through all async call chains
