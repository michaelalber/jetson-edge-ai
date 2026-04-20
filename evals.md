# Jetson Edge AI — Evals

---

## Eval Philosophy

Evals are safety infrastructure — write them before the agent starts. A passing test suite ≠ done;
tests verify code correctness, evals verify output is actually good relative to project intent.

A passing eval is measurable, repeatable, and would survive scrutiny from the developer
(Michael K. Alber) reviewing it against the project's learning goals and hardware constraints.

Evals answer: *"Is the output actually good?"* — not *"does it look reasonable?"*

---

## Test Cases

### Test Case 1: New Python Project Scaffold

- **Input / Prompt:** "Scaffold the `sensor-pipeline` project with a BME280 I2C temperature/humidity driver."
- **Known-Good Output:** A `projects/sensor-pipeline/` directory containing `pyproject.toml`, `src/sensor_pipeline/` with type-annotated source, `tests/` with at least one passing unit test using a mock I2C bus, `ruff.toml` or inline ruff config, and a `README.md` stub.
- **Pass Criteria:**
  - [ ] `pip install -e ".[dev]" && pytest` passes from `projects/sensor-pipeline/` with zero real hardware required
  - [ ] `ruff check src/ tests/` — clean
  - [ ] `mypy src/` — clean
  - [ ] No `List`, `Optional`, `Dict` from `typing` in generated code
  - [ ] `pyproject.toml` pins all runtime dependencies with exact versions
  - [ ] Hardware-dependent tests (real I2C bus) marked with `pytest.mark.hardware` and skipped by default
  - [ ] No cross-imports to other `projects/` sub-directories
- **Last Run:** — | **Result:** —
- **Notes:** —

---

### Test Case 2: Industrial Protocol Integration

- **Input / Prompt:** "Add an OPC-UA subscription to `industrial-gateway` that reads `PLC1.Tank1.Level` at ns=2 and publishes to MQTT."
- **Known-Good Output:** Async Python module using `asyncua` subscriptions (not polling), MQTT publish via `aiomqtt` with QoS 1, NodeId loaded from a Pydantic settings model, MQTT topic as a module-level constant following `jetson/{project}/{device_id}/{measurement}` convention, and unit tests using a mock OPC-UA client and mock MQTT broker.
- **Pass Criteria:**
  - [ ] `pytest` passes with zero real PLC or broker required
  - [ ] NodeId `"ns=2;s=PLC1.Tank1.Level"` does not appear as an inline literal — it comes from config
  - [ ] MQTT topic string is a module-level constant, not an inline literal
  - [ ] OPC-UA code uses `subscribe_data_change` / `DataChangeNotif` — not a polling loop
  - [ ] LWT configured for the device connection
  - [ ] `ruff check` and `mypy` — clean
- **Last Run:** — | **Result:** —
- **Notes:** —

---

### Test Case 3: Model Optimization / TensorRT Conversion Script

- **Input / Prompt:** "Write a TensorRT conversion script for `visual-inspection` that converts a YOLOv8 ONNX model to a `.engine` file."
- **Known-Good Output:** Python script under `models/convert/` using `tensorrt` Python API, with a `--input` / `--output` CLI interface, no hardcoded file paths, `.engine` output excluded from git via `.gitignore`, and a `--dry-run` flag for testing without GPU.
- **Pass Criteria:**
  - [ ] Script imports guarded: `try: import tensorrt` with a clear error message if not available
  - [ ] No model weight paths hardcoded — all paths via CLI args or `pathlib.Path` config
  - [ ] `.engine` and `.onnx` not committed to git (verified by checking `.gitignore`)
  - [ ] `--dry-run` flag skips GPU calls and returns exit code 0 — allows CI testing without hardware
  - [ ] `ruff check` and `mypy` — clean on the script file
- **Last Run:** — | **Result:** —
- **Notes:** —

---

### Test Case 4: Blazor Dashboard Feature Slice

- **Input / Prompt:** "Add a `SensorMetrics` feature to `monitoring-dashboard` that displays live temperature readings via SignalR."
- **Known-Good Output:** `Features/SensorMetrics/` directory with `SensorMetricsHub.cs`, `SensorMetricsService.cs`, a Blazor component `SensorMetrics.razor`, and an xUnit test for the service. No cross-feature dependencies. `CancellationToken` propagated through all async methods.
- **Pass Criteria:**
  - [ ] `dotnet test` passes — zero real MQTT broker or Jetson required
  - [ ] `dotnet build` — zero warnings (warnings-as-errors is on)
  - [ ] No cross-feature imports — `Features/SensorMetrics/` depends only on shared interfaces, not other features
  - [ ] `CancellationToken` present in all `async` method signatures in the feature
  - [ ] No nullable reference warnings
- **Last Run:** — | **Result:** —
- **Notes:** —

---

## Taste Rules (Encoded Rejections)

| # | Pattern to Reject | Why It Fails | Rule |
|---|---|---|---|
| 1 | Output that "looks right" but isn't grounded in project context | Generic output wastes time — requires cleanup that defeats delegation | Always anchor to a specific fact from `AGENTS.md` or the active task spec |
| 2 | Unit tests that require a real Jetson, PLC, or sensor to run | Defeats the CI gate; blocks development on non-Jetson machines | All tests must pass with simulated hardware; mark real-hardware tests with `pytest.mark.hardware` |
| 3 | NodeIds or MQTT topic strings as inline literals | Hardcoded config breaks multi-device deployments and makes tests fragile | Config belongs in Pydantic settings or YAML; topics are module-level constants |
| 4 | `from typing import List, Optional, Dict` in new Python code | Project standard requires built-in generics | Use `list`, `X \| None`, `dict` and `from __future__ import annotations` |
| 5 | Cross-imports between `projects/` sub-directories | Breaks independent deployability | Each project is a deployment unit; shared code goes in `shared/` only when 2+ projects need it |

---

## CI Gate

- **Build (Python):** `pip install -e ".[dev]"` — zero errors
- **Build (.NET):** `dotnet build` — zero errors, zero warnings
- **Tests (Python):** `pytest` — all pass; hardware tests skipped automatically
- **Tests (.NET):** `dotnet test` — all pass
- **Coverage:** ≥ 80% — `pytest --cov=src --cov-report=term-missing`
- **Security scan:** `snyk code test` — zero high or critical issues
- **Lint (Python):** `ruff check src/ tests/` — clean
- **Types (Python):** `mypy src/` — clean

> Append CI gate results as a sub-item of each Test Case entry on every run.

---

## Rejection Log

<!-- Append entries here as outputs are rejected. Never delete entries.
     Review periodically to extract new Taste Rules above. -->
