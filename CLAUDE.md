# Jetson Edge AI — Project Context

This is the project-level CLAUDE.md for Claude Code. It supplements the global CLAUDE.md — it does
not replace it. Global standards (coding style, security rules, quality gates) live in the global
file. This file contains only what is specific to this project.

---

## Project Overview

- **Name:** Jetson Edge AI
- **Purpose:** Learning repository for Edge AI and Industrial Automation on the NVIDIA Jetson Orin Nano — building practical, end-to-end projects covering inference pipelines, sensor integration, industrial protocol bridging, and real-time monitoring dashboards.
- **Phase:** Discovery
- **Jira project key:** N/A — no issue tracker; confirm active task with the user at session start.
- **Confluence space:** N/A — no wiki; ADRs live in `docs/adr/`.
- **Definition of success:** Each sub-project under `projects/` contains a working, tested implementation that runs on-device (arm64, JetPack 6.x) and demonstrates the named capability end-to-end.

---

## Technology Stack

- **Language(s):** Python 3.10+ (inference / sensor / automation); C# 13 / .NET 10 (monitoring dashboard)
- **Framework(s):** asyncua (OPC-UA), aiomqtt (MQTT), pymodbus (Modbus TCP), OpenCV, TensorRT Python API, Ollama Python client; .NET: Blazor Server + SignalR
- **Database:** None decided — metrics streamed in-memory; persistence is an open loop
- **UI library:** Blazor Server built-in components (no third-party component library chosen yet)
- **Test framework:** pytest + pytest-asyncio (Python); xUnit (.NET)
- **CI/CD:** None yet — local `pytest` / `dotnet test` only
- **Package manager:** pip via `pyproject.toml` (Python); NuGet (.NET)

---

## Architecture

- **Pattern:** Feature-per-project — each directory under `projects/` is a self-contained vertical slice with its own `pyproject.toml` or `.sln`. The .NET dashboard uses vertical slice feature folders under `Features/`.
- **Entry points:** `projects/<name>/src/<name>/main.py` (Python, not yet scaffolded); `projects/monitoring-dashboard/src/Dashboard/Program.cs` (.NET, not yet scaffolded)
- **Key directories:**
  - `projects/` — six independent sub-projects (see layout below)
  - `infrastructure/docker/` — per-project Dockerfiles (arm64 / L4T base images)
  - `infrastructure/scripts/` — Jetson provisioning and deploy scripts
  - `docs/adr/` — Architecture Decision Records; read before any new technology decision
  - `models/` — configs and conversion scripts only; weights are gitignored
  - `shared/` — empty; only add code here when two or more projects genuinely share it
- **Non-obvious constraints:**
  - GPU libraries (TensorRT, CUDA) are only available on-device or inside L4T containers — expect CPU-only fallbacks in local dev; skip GPU-dependent tests on non-Jetson machines.
  - 8 GB unified memory is shared between CPU and GPU — LLM models must leave headroom for OS and inference runtime.
  - Never cross-import between `projects/` sub-directories — each project must be independently deployable.
  - Blazor Server requires a persistent SignalR connection; acceptable for LAN-deployed dashboards only.

---

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
docs/adr/               # Architecture Decision Records
```

---

## Development Commands

### Python projects (run from within a project directory)

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

pytest                              # all tests
pytest tests/test_foo.py::test_bar  # single test
ruff check src/ tests/              # lint
mypy src/                           # type check
```

### .NET dashboard (`projects/monitoring-dashboard/`)

```bash
dotnet restore
dotnet build
dotnet test
dotnet test --filter "FullyQualifiedName~SomeTest"
dotnet run --project src/Dashboard
```

### Docker — building for Jetson (arm64)

```bash
# Enable arm64 emulation once on an x86 host
docker run --privileged --rm tonistiigi/binfmt --install arm64

docker buildx build \
  --platform linux/arm64 \
  -t jetson-edge-ai/<project-name>:latest \
  -f infrastructure/docker/<project-name>/Dockerfile \
  projects/<project-name>
```

---

## Key Files

| File | Why It Matters |
|---|---|
| `docs/adr/` | Five active ADRs — read before any new technology or pattern decision |
| `docs/adr/001-python-for-inference-pipelines.md` | Rationale for Python 3.10+ and pyproject.toml per project |
| `docs/adr/002-dotnet-blazor-for-dashboard.md` | Rationale for .NET 10 Blazor Server + SignalR; API boundary with Python services |
| `docs/adr/003-ollama-for-local-llm.md` | Ollama model sizing constraints (must fit in ~6 GB headroom) |
| `docs/adr/005-industrial-protocol-integration.md` | OPC-UA + MQTT as primary protocols; topic convention; QoS defaults |
| `docs/setup/jetson-setup.md` | Jetson Orin Nano provisioning guide |
| `docs/setup/development.md` | Local dev environment setup (cross-platform) |

---

## Persistent Decisions

| Date | Decision | Rationale |
|---|---|---|
| 2026-03-07 | Python 3.10+ with pyproject.toml for all inference/sensor pipelines | JetPack ships first-class Python ML bindings; broad ecosystem support (ADR-001) |
| 2026-03-07 | .NET 10 Blazor Server (SignalR) for monitoring dashboard | Real-time push without a separate WebSocket server; C# throughout UI (ADR-002) |
| 2026-03-07 | Ollama for on-device LLM; GGUF quantized models ≤ ~6 GB | Keeps GPU memory headroom for OS + inference runtime (ADR-003) |
| 2026-03-07 | Docker with NVIDIA L4T base images; --platform linux/arm64 for cross-builds | Reproducible arm64 builds from x86 dev machines (ADR-004) |
| 2026-03-07 | OPC-UA (asyncua) + MQTT (aiomqtt) as primary industrial protocols; Modbus TCP as legacy fallback | Async-native libraries; OPC-UA is the modern standard (ADR-005) |
| 2026-03-07 | MQTT topic convention: `jetson/{project}/{device_id}/{measurement}` | Supports multiple physical devices per project (ADR-005) |
| 2026-03-07 | Jetson is always a consumer of PLC data — never a PLC programmer | Safety boundary: AI inference only; no write-back to field devices |
| 2026-03-07 | No cross-imports between projects/ sub-directories | Each project must deploy independently |

---

## Open Loops

- [ ] Which project to scaffold first — `sensor-pipeline`, `visual-inspection`, or `industrial-gateway`?
- [ ] Persistence layer: where do processed metrics get stored — InfluxDB, SQLite, in-memory only?
- [ ] CI/CD: GitHub Actions for arm64 cross-build + test?
- [ ] Dashboard ↔ Python API boundary: REST or gRPC? (ADR-002 flags this as needed)
- [ ] SparkplugB (MQTT payload schema): worth adding once baseline MQTT is working (flagged in ADR-005)?

---

## Team

| Name | Role | Notes |
|---|---|---|
| Michael K. Alber | Sole developer | Reviewer and decision-maker on all changes |

---

## Available Tools

- `mcp__grounded-code-mcp__search_knowledge` — Search the local RAG knowledge base. Use before generating non-trivial protocol integration or inference code. Collections relevant to this project: `edge_ai`, `automation`, `python`, `dotnet`, `robotics`, `architecture`.
- `mcp__grounded-code-mcp__search_code_examples` — Fetch code examples from the knowledge base. Use alongside `search_knowledge` when generating code.

---

## Project Boot Ritual

At the start of every session:

1. Read this file (`CLAUDE.md`), `intent.md`, and `constraints.md`.
2. No Jira/Confluence — confirm the active task with the user directly.
3. Confirm context — state: current phase (Discovery), active task (if any), top 3 constraints, open loops.
4. Do NOT begin work until context is confirmed.

---

## Python Standards (Project-Specific)

- Use built-in generic types: `list[str]`, `dict[str, int]`, `X | None` — never `List`, `Optional`, `Dict` from `typing`. Use `from __future__ import annotations` for forward references.
- Type hints on all function signatures; Pydantic v2 for settings and data validation.
- `pathlib.Path` over `os.path`.
- `async def` / `asyncio` for all I/O-bound work (sensor polling, HTTP calls, protocol clients).
- Prompt templates live in versioned files under a `prompts/` directory — not inline f-strings.
- Pin all ML/AI dependency versions explicitly in `pyproject.toml`.

---

## .NET Standards (Project-Specific)

- .NET 10, nullable reference types enabled, warnings as errors.
- Blazor Server with SignalR for real-time metric streaming.
- Vertical slice feature folders under `Features/` — not layer folders.
- `CancellationToken` propagated through all async call chains.

---

## Industrial Protocol Reference

### OPC-UA (IEC 62541)

**Core concepts:**
- **Address Space** — Tree of Nodes. Every data point lives at a Node.
- **NodeId** — `ns=2;s=PLC1.Tank1.Temperature` — namespace + identifier.
- **Subscriptions / MonitoredItems** — Server pushes changes; avoids polling.
- **Security** — Built-in signing, encryption, X.509 certs, user auth. Never disable in production.
- **Transport** — Binary TCP `opc.tcp://host:4840`.

**Agent rules:**
- Always use subscriptions over polling for real-time data.
- NodeIds belong in Pydantic settings or `config/nodes.yaml` — never hardcoded.
- Always handle `asyncio` task cancellation for clean shutdown.
- Security mode `SignAndEncrypt` is the target for production; `None` is acceptable in local dev only.

```python
from asyncua import Client
async with Client(url="opc.tcp://192.168.1.10:4840") as client:
    node = client.get_node("ns=2;s=PLC1.Temperature")
    value = await node.read_value()
```

### MQTT (ISO/IEC 20922)

**Core concepts:**
- **Broker** — Mosquitto for local dev; HiveMQ / AWS IoT Core / Azure IoT Hub for cloud.
- **Topic** — `jetson/{project}/{device_id}/{measurement}`. Wildcards: `+` (one level), `#` (multi-level).
- **QoS:** 0 = high-frequency telemetry, 1 = events (default), 2 = commands/setpoints.
- **LWT** — Pre-registered disconnect message; always configure for device status topics.

**Agent rules:**
- Topic strings are module-level constants — never inline string literals.
- QoS is a conscious choice — document reasoning inline.
- Always configure LWT for device online/offline status topics.

```python
import aiomqtt
async with aiomqtt.Client("192.168.1.5") as client:
    await client.publish("jetson/industrial-gateway/plc1/tank_level", payload="42.3", qos=1)
```

### PLC Integration

**Data flow:**
```
Field Devices (PLCs, sensors)
    | OPC-UA subscription / Modbus TCP poll
    v
industrial-gateway (Jetson)
    | MQTT publish  QoS 1
    v
MQTT Broker (Mosquitto)
    +--- monitoring-dashboard (subscriber)
    +--- anomaly-detection (subscriber)
```

| Protocol | Port | Python Library | Use |
|---|---|---|---|
| OPC-UA | TCP 4840 | `asyncua` | Modern PLCs; preferred |
| Modbus TCP | TCP 502 | `pymodbus` | Legacy/ubiquitous |
| EtherNet/IP | TCP/UDP | `pycomm3` | Rockwell/Allen-Bradley |
| Siemens S7 | TCP 102 | `python-snap7` | Siemens S7 family |

**Modbus register types:** Coils (1-bit R/W), Discrete Inputs (1-bit R), Input Registers (16-bit R), Holding Registers (16-bit R/W).

**Agent rules:**
- Jetson is always a *consumer* of PLC data — never a PLC programmer.
- Modbus register addresses and OPC-UA NodeIds are configuration, not code constants.
- Simulate PLC data in tests — never require real hardware for unit or integration tests.
