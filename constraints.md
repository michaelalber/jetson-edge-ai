# Jetson Edge AI — Constraints

---

## Must Do

- Load and confirm context (`AGENTS.md`, `intent.md`, `constraints.md`) before every session.
- Confirm the active task with the user directly — no Jira/Confluence in this project.
- Write three verifiable acceptance criteria before starting any non-trivial implementation.
- Confirm with the user before any irreversible action (delete files, push to remote, deploy to Jetson).
- Provide a CPU-only fallback or a clearly-marked `pytest.mark.hardware` skip for every test that requires GPU, a real sensor, or a real PLC.
- Store all protocol configuration (OPC-UA NodeIds, MQTT topic strings, Modbus register addresses) in Pydantic settings models or YAML config files — never as inline literals.
- Pin all ML/AI dependency versions explicitly in `pyproject.toml`.
- Search `grounded-code-mcp` (`edge_ai`, `automation`, `python`, `dotnet` collections) before generating any non-trivial protocol integration or inference code.

---

## Must NOT Do

- Do not begin a task that has no verifiable acceptance criteria.
- Do not re-litigate decisions already logged in `AGENTS.md` or `intent.md` Persistent Decisions tables.
- Do not cross-import between `projects/` sub-directories — each project must be independently deployable.
- Do not commit model weights (`.pt`, `.onnx`, `.engine`, `.gguf`, `.bin`, `.safetensors`).
- Do not hardcode secrets, API keys, or device credentials — use environment variables or a secrets manager.
- Do not write code that requires a real Jetson, PLC, or physical sensor to pass any unit or integration test — simulate hardware in tests.
- Do not disable OPC-UA security (`SecurityPolicy.NoSecurity`) outside of a local dev context without explicit user approval.
- Do not add code to `shared/` unless two or more projects already use it — YAGNI.
- Do not use `List`, `Optional`, `Dict` from `typing` — use built-in generics and `from __future__ import annotations`.
- Do not move any task to "Done" — that is the developer's call only.

---

## Preferences

- Prefer subscriptions over polling for OPC-UA data — polling is a deliberate exception requiring documented justification.
- Prefer `asyncio`-native libraries (`asyncua`, `aiomqtt`) over synchronous alternatives in Python projects.
- Prefer brevity over completeness unless depth is explicitly requested.
- Prefer editing an existing file over creating a new one.
- Prefer flagging a problem before executing a workaround.
- Prefer asking one clarifying question over assuming and proceeding.
- Prefer `grounded-code-mcp` knowledge base over training data for protocol, ML, and framework idioms.
- Prefer direct implementation over abstractions until two or more projects share a need.

---

## Escalate Rather Than Decide

- Any choice of persistence layer (database type, time-series store) — open loop, not yet decided.
- Any choice of Dashboard ↔ Python API transport (REST vs gRPC) — open loop, not yet decided.
- Any action that conflicts with a logged Persistent Decision in `AGENTS.md`.
- Any request where acceptance criteria cannot be met within stated constraints (especially the 8 GB memory constraint).
- Any OPC-UA security mode change for non-dev environments.
- Any proposed change to the MQTT topic convention (`jetson/{project}/{device_id}/{measurement}`).
- Adding a new project sub-directory or changing the overall data flow architecture.

---

## Code Quality Gates

- **Test coverage (business logic):** ≥ 80% — `pytest --cov=src --cov-report=term-missing`
- **Test coverage (security-critical paths):** ≥ 95% (auth, protocol security, credential handling)
- **Cyclomatic complexity (per method):** < 10
- **Code duplication:** ≤ 3%
- **Commit format:** Conventional Commits — `feat:`, `fix:`, `refactor:`, `chore:`, `test:`, `docs:`
- **Commit scope:** Atomic — one logical change per commit; scope to the project name where unambiguous (e.g., `feat(sensor-pipeline): add BME280 driver`)
- **Python lint:** `ruff check src/ tests/` — zero errors
- **Python types:** `mypy src/` — zero errors
- **Security scan:** `snyk code test` — zero high or critical issues
- **Hardware-gated tests:** must be skipped automatically on non-Jetson machines with `pytest.mark.hardware`
