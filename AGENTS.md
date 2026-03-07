# AGENTS.md

This file provides guidance to AI coding agents (such as opencode) when working with code in this repository.

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
  industrial-gateway/   # Python — OPC-UA client + MQTT publisher bridge
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
- **ADR-005** OPC-UA (`asyncua`) + MQTT (`aiomqtt`) as industrial integration protocols

## Key Constraints

- **Memory:** 8 GB unified — CPU and GPU share it. LLM models must leave headroom for OS and inference runtime.
- **Architecture:** Jetson is `linux/arm64`. GPU libraries (TensorRT, CUDA) are only available on-device or in L4T containers.
- **Python GPU libs:** On a dev machine, expect CPU-only fallbacks or skip GPU-dependent tests.
- **Model weights:** Never commit weights (`.pt`, `.onnx`, `.engine`, `.gguf`, `.bin`, `.safetensors` are gitignored). Use download scripts under `models/download/`.
- **`shared/`:** Only add code here when two or more projects genuinely share it — not speculatively.

## Python Standards (for this repo)

- Use built-in generic types — `list[str]`, `type | None` — not `List`, `Optional` from `typing`. Use `from __future__ import annotations` for forward references.
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

## Industrial Protocols

These protocols are first-class learning targets. An agent working in this repo must understand them conceptually before generating code that touches them.

### OPC-UA (IEC 62541) — "REST for factory equipment"

OPC-UA is the industrial standard for secure, reliable, platform-independent data exchange between devices and software.

**Core concepts:**
- **Address Space** — A tree of *Nodes* hosted by an OPC-UA Server. Every data point lives at a Node.
- **NodeId** — Unique identifier: `ns=2;s=PLC1.Tank1.Temperature`
- **Namespace** — Logical scope: ns=0 is OPC-UA standard; ns=1+ are vendor/device-specific.
- **Subscriptions / MonitoredItems** — Server pushes changes to subscribed clients; avoid polling.
- **Security** — Built-in signing, encryption, X.509 certs, user auth. Never disable in production.
- **Transport** — Binary TCP `opc.tcp://host:4840` (standard).

**Python library:** `asyncua` — always `async`/`await`.
```python
from asyncua import Client
async with Client(url="opc.tcp://192.168.1.10:4840") as client:
    node = client.get_node("ns=2;s=PLC1.Temperature")
    value = await node.read_value()
```

**Agent rules:**
- Always use subscriptions over polling for real-time data.
- Never hardcode NodeIds — put them in a Pydantic settings model or config file.
- Always propagate `CancellationToken`-equivalent (`asyncio.Event` / task cancellation) for clean shutdown.

---

### MQTT (ISO/IEC 20922) — Lightweight pub/sub

A broker-mediated publish-subscribe protocol. A central broker routes messages by topic.

**Core concepts:**
- **Broker** — Mosquitto for local dev. Cloud: HiveMQ, AWS IoT Core, Azure IoT Hub.
- **Topic** — Hierarchical string: `factory/line1/jetson/defect_count`. Wildcards: `+` (one level), `#` (multi-level).
- **QoS:** 0=fire-and-forget, 1=at-least-once, 2=exactly-once. Default to QoS 1 for events.
- **Retain** — Broker stores last message; new subscribers receive it immediately. Use for device status.
- **LWT** — Pre-registered disconnect message. Always configure for device health topics.

**Topic convention for this repo:** `jetson/{project}/{measurement}`
- e.g., `jetson/sensor-pipeline/temperature`, `jetson/visual-inspection/defect_count`

**Python library:** `aiomqtt` (async, preferred).
```python
import aiomqtt
async with aiomqtt.Client("192.168.1.5") as client:
    await client.publish("jetson/sensor-pipeline/temperature", payload="42.3", qos=1)
```

**Agent rules:**
- Topic strings belong in constants or config — not scattered as string literals.
- QoS must be a conscious choice — document the reasoning in code comments.
- Always configure a LWT for device online/offline status.

---

### PLC Basics

A **PLC** is a ruggedized industrial computer: Read Inputs → Execute Logic → Write Outputs → Repeat (1–100 ms scan cycle). Deterministic, fault-tolerant.

**Ladder Logic** (IEC 61131-3) — Relay diagram metaphor:
- Contacts `--[ ]--` (conditions) on the left energize a Coil `--( )--` (output) on the right.
- Scan is cyclic — logic executes top-to-bottom every scan.

**Integration protocols (Jetson as consumer only):**

| Protocol | Port | Python Library | Use |
|---|---|---|---|
| OPC-UA | TCP 4840 | `asyncua` | Modern PLCs; preferred |
| Modbus TCP | TCP 502 | `pymodbus` | Legacy/ubiquitous |
| EtherNet/IP | TCP/UDP | `pycomm3` | Rockwell/Allen-Bradley |
| Siemens S7 | TCP 102 | `python-snap7` | Siemens S7 family |

**Modbus register types:** Coils (1-bit R/W), Discrete Inputs (1-bit R), Input Registers (16-bit R), Holding Registers (16-bit R/W).

**Data flow mental model:**
```
PLC (field devices) → OPC-UA / Modbus TCP → Jetson AI → MQTT → Dashboard / Cloud
```

**Agent rules:**
- The Jetson is always a *consumer* of PLC data — never a PLC programmer.
- Modbus register addresses and OPC-UA NodeIds are configuration, not constants in code.
- Simulate PLC data in tests — never require real hardware for unit tests.
