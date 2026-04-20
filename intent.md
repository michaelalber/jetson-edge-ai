# Jetson Edge AI — Intent

---

## Agent Architecture

**This project uses:** Coding harness

**Reason:** Discovery-phase learning project with a sole developer — human reviews every output before merging; no autonomous multi-session loop needed.

---

## Primary Goal

Build working, on-device implementations of each project sub-directory that run on the NVIDIA Jetson Orin Nano (arm64, JetPack 6.x), demonstrating real-world Edge AI and Industrial Automation capabilities end-to-end — from sensor input or camera frame through AI inference to MQTT telemetry or a live dashboard.

---

## Values (What We Optimize For)

1. **Correctness** — code that actually runs on the target hardware and produces valid output
2. **Learnability** — implementations that teach the underlying concepts, not just produce output
3. **Maintainability** — each project is independently readable, testable, and deployable
4. **Security** — no hardcoded credentials; safe defaults for OPC-UA and MQTT
5. **Performance** — GPU memory efficiency on the 8 GB unified memory constraint
6. **Speed of delivery** — last priority; this is a learning repo, not a production deadline

---

## Tradeoff Rules

| Conflict | Resolution |
|---|---|
| Speed vs. correctness | Correctness. Flag explicitly if a shortcut is proposed. |
| Completeness vs. brevity | Brevity unless depth is explicitly requested. |
| Learnability vs. production patterns | Prefer the pattern that teaches the concept; note production divergences in comments. |
| GPU feature vs. CPU fallback | Always provide a CPU fallback path for local dev; document which path is active. |
| New abstraction vs. direct implementation | Direct implementation first; extract only when two projects genuinely share it. |
| Polling vs. subscriptions (OPC-UA) | Always prefer subscriptions — polling is a deliberate exception with documented reasoning. |

---

## Decision Boundaries

### Decide Autonomously

- Formatting, naming, file structure within established project conventions
- Tool selection for read-only exploration (Glob, Grep, Read)
- Refactoring within the approved, scoped task
- Choosing `pytest-asyncio` mode (`asyncio_mode = "auto"` vs manual) per project
- Adding type stubs or dev-only dependencies without changing runtime behaviour

### Escalate to Human

- Any output intended for external distribution (docs, emails, published packages)
- Any irreversible action (delete files, push to remote, deploy to Jetson)
- Any request that contradicts a logged decision in `AGENTS.md` Persistent Decisions
- Scope changes: adding a new project sub-directory or changing the data flow architecture
- When acceptance criteria cannot be met within stated constraints
- Choosing a persistence layer (database, time-series store) — open loop, not yet decided
- Choosing a Dashboard ↔ Python API transport (REST vs gRPC) — open loop, not yet decided
- Any OPC-UA security mode change in non-dev environments

---

## What "Good" Looks Like

A good output for this project:

- Runs `pytest` (or `dotnet test`) clean on the first attempt with no skipped tests except hardware-gated ones
- Hardware-gated tests (GPU, real sensor, real PLC) are clearly marked with `pytest.mark.hardware` or equivalent and skipped automatically on non-Jetson machines
- Protocol configuration (NodeIds, MQTT topics, register addresses) lives in config files or Pydantic settings — never hardcoded
- Each project is independently runnable: `pip install -e ".[dev]" && pytest` works from the project directory with no side effects on other projects
- Code teaches the concept: a new reader can understand *why* the pattern was chosen, not just what it does

---

## Anti-Patterns (What Bad Looks Like)

- Generating code that requires a real Jetson, PLC, or sensor to run any test — unit tests must be hardware-free
- Hardcoding NodeIds, MQTT topic strings, or Modbus register addresses as inline string/int literals
- Cross-importing between `projects/` sub-directories (breaks independent deployability)
- GPU-only code paths with no CPU fallback or clear skip marker
- Committing model weights (`.pt`, `.onnx`, `.engine`, `.gguf`, `.bin`, `.safetensors`)
- Using `List`, `Optional`, `Dict` from `typing` instead of built-in generics
- Generating a new project scaffold that doesn't include a `tests/` directory and at least one passing test

---

## Persistent Decisions

| Date | Decision | Rationale |
|---|---|---|
| 2026-03-07 | Coding harness architecture | Discovery phase; sole developer; human reviews every change |
| 2026-03-07 | Learning intent takes priority over production-readiness | Repo purpose is education; production patterns are noted but not required |

---

## Open Loops

- [ ] Which project to scaffold first?
- [ ] Persistence layer for metrics (InfluxDB, SQLite, in-memory)?
- [ ] Dashboard ↔ Python API boundary: REST or gRPC?
- [ ] CI/CD: GitHub Actions for arm64 cross-build + test?
- [ ] SparkplugB for structured MQTT payloads — revisit after baseline MQTT is working
