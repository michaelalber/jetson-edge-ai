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

These protocols are first-class learning targets in this repo. Understand the concepts before implementing; they have sharp edges that bite if misunderstood.

### OPC-UA (IEC 62541) — "REST for factory equipment"

OPC-UA is the industrial standard for secure, reliable, platform-independent data exchange between devices and software. It defines both a structured data model (the Address Space) and a transport layer.

**Core concepts:**
- **Address Space** — A tree of *Nodes* hosted by an OPC-UA Server. Every piece of data (a temperature value, a motor status) lives at a Node.
- **NodeId** — Globally unique identifier for a node: `ns=2;s=PLC1.Tank1.Temperature`
- **Namespace** — Logical scope for NodeIds (ns=0 is OPC-UA standard; ns=1+ are vendor/device-specific).
- **Services** — Read, Write, Browse, Subscribe, Call. A client calls services against a server.
- **Subscriptions / MonitoredItems** — The server pushes value changes to subscribed clients (avoids polling).
- **Security** — Built-in: message signing, encryption, X.509 certificates, user authentication. Do not disable security in production.
- **Transport** — Binary TCP (`opc.tcp://host:4840`) is standard. HTTPS variant also exists.

**Jetson role:** OPC-UA *client* — reads from PLCs/field devices and exposes processed results as an OPC-UA server to upstream SCADA/MES systems.

**Python library:** `asyncua` (async, actively maintained).
```python
from asyncua import Client
async with Client(url="opc.tcp://192.168.1.10:4840") as client:
    node = client.get_node("ns=2;s=PLC1.Temperature")
    value = await node.read_value()
```

**Reference:** OPC Foundation — https://reference.opcfoundation.org/

---

### MQTT (ISO/IEC 20922) — Lightweight pub/sub for IoT

MQTT is a publish-subscribe messaging protocol over TCP, designed for constrained devices and unreliable networks. A central *broker* routes messages between publishers and subscribers by *topic*.

**Core concepts:**
- **Broker** — Central message router. Mosquitto is the standard local broker (`mosquitto` package). Cloud options: HiveMQ, AWS IoT Core, Azure IoT Hub.
- **Topic** — Hierarchical UTF-8 string: `factory/line1/jetson/defect_count`. No registration required — publish to any topic.
  - Wildcard subscriptions: `+` (single level), `#` (multi-level)
- **QoS levels:**
  - `0` — Fire-and-forget. No delivery guarantee. Use for high-frequency telemetry.
  - `1` — At-least-once. Message arrives but may duplicate. Use for events.
  - `2` — Exactly-once. Highest overhead. Use for commands/control.
- **Retain** — Broker stores the last message on a topic; new subscribers receive it immediately.
- **LWT (Last Will and Testament)** — Client pre-registers a message the broker publishes on unexpected disconnect. Use for device health/status topics.

**Topic naming convention for this repo:** `jetson/{project}/{measurement}`
- e.g., `jetson/sensor-pipeline/temperature`, `jetson/visual-inspection/defect_count`

**Jetson role:** MQTT publisher for sensor readings, inference results, and device health. Subscriber for remote configuration updates.

**Python library:** `aiomqtt` (async wrapper around paho-mqtt — preferred for `asyncio` code).
```python
import aiomqtt
async with aiomqtt.Client("192.168.1.5") as client:
    await client.publish("jetson/sensor-pipeline/temperature", payload="42.3", qos=1)
```

---

### PLC Basics — What they are and how they talk

A **Programmable Logic Controller (PLC)** is a ruggedized industrial computer that reads inputs (sensors, switches, encoders) and drives outputs (motors, valves, alarms) based on deterministic programmed logic. Designed for 24/7 operation in harsh environments.

**Ladder Logic** — The primary IEC 61131-3 PLC language. Looks like a relay circuit diagram:
- Read it left-to-right, top-to-bottom. Each *rung* evaluates conditions (contacts) and energizes an output (coil).
- Contacts: `--[ ]--` (normally open), `--[/]--` (normally closed).
- Coil: `--( )--` (output energized when the rung is true).
- Execution is cyclic: Read Inputs → Execute Logic → Write Outputs → Repeat (typically 1–100 ms scan time).

**How PLCs communicate (integration layer options):**

| Protocol | Transport | Python Library | Notes |
|---|---|---|---|
| OPC-UA | TCP :4840 | `asyncua` | Modern standard; most new PLCs support natively |
| Modbus TCP | TCP :502 | `pymodbus` | Ubiquitous legacy protocol; simple register model |
| EtherNet/IP | TCP/UDP | `pycomm3` | Allen-Bradley / Rockwell PLCs |
| Siemens S7 | TCP :102 | `snap7` / `python-snap7` | Siemens S7-300/400/1200/1500 |

**Modbus register types** (most common legacy protocol):
- **Coils** (0x) — 1-bit read/write (digital outputs)
- **Discrete Inputs** (1x) — 1-bit read-only (digital inputs)
- **Input Registers** (3x) — 16-bit read-only (analog inputs)
- **Holding Registers** (4x) — 16-bit read/write (setpoints, parameters)

**Jetson role:** Consumer of PLC data — not programming or controlling PLCs directly. The Jetson reads process values via OPC-UA or Modbus TCP, runs AI inference, and publishes results via MQTT or back to the OPC-UA server.

**Key mental model:**
```
PLC (sensors/actuators) → OPC-UA/Modbus → Jetson (AI inference) → MQTT → Dashboard/Cloud
```
