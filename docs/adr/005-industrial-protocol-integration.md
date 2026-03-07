# ADR-005: Industrial Protocol Integration — OPC-UA + MQTT

**Status:** Accepted
**Date:** 2026-03-07

## Context

The repository's scope includes Physical AI applications for Industrial Automation on the Jetson Orin Nano. Industrial devices (PLCs, sensors, drives) communicate via specialized protocols — not REST or gRPC. We need to decide which protocols to prioritize as learning targets and integration layers.

The three protocols most relevant to modern industrial automation are:

| Protocol | Role | Maturity |
|---|---|---|
| OPC-UA (IEC 62541) | Structured device communication, security, subscriptions | Modern standard, wide PLC support |
| MQTT (ISO/IEC 20922) | Lightweight pub/sub telemetry for IoT/edge | Ubiquitous in IIoT |
| Modbus TCP | Legacy register-based device access | 40+ year old protocol, still dominant in field devices |

## Decision

**Primary integration protocols:** OPC-UA (client) and MQTT (publisher).

**Secondary protocol:** Modbus TCP for legacy device support via `pymodbus`.

**Python libraries selected:**

| Role | Library | Rationale |
|---|---|---|
| OPC-UA client | `asyncua` | Fully async, actively maintained, supports OPC-UA 1.04 |
| MQTT client | `aiomqtt` | Async-native wrapper around paho-mqtt; consistent with asyncio model |
| Modbus TCP | `pymodbus` | De-facto standard Python Modbus library |

## Data Flow Pattern

```
Field Devices (PLCs, sensors)
    |
    | OPC-UA subscription / Modbus TCP poll
    v
industrial-gateway (Jetson)
    |
    | MQTT publish  QoS 1, topic: jetson/{project}/{device}/{measurement}
    v
MQTT Broker (Mosquitto local / cloud)
    |
    +--- monitoring-dashboard (subscriber)
    +--- anomaly-detection (subscriber)
    +--- cloud / historian (subscriber)
```

## Topic Naming Convention

```
jetson/{project}/{device_id}/{measurement}
```

Examples:
- `jetson/industrial-gateway/plc1/tank_level`
- `jetson/sensor-pipeline/imu1/acceleration_x`
- `jetson/visual-inspection/camera1/defect_count`

## MQTT QoS Defaults

| Data type | QoS | Reason |
|---|---|---|
| High-frequency telemetry (>10 Hz) | 0 | Volume; occasional loss acceptable |
| Process events, state changes | 1 | At-least-once delivery sufficient |
| Commands / setpoints | 2 | Exactly-once; avoid duplicate actuation |
| Device online/offline (LWT) | 1 | At-least-once is sufficient for status |

## Consequences

- All industrial I/O code is async (`asyncio`); blocking calls are forbidden.
- NodeIds (OPC-UA) and register addresses (Modbus) are configuration, not constants — stored in `config/nodes.yaml` or equivalent.
- Topic strings are constants in a dedicated module, not inline string literals.
- Tests use simulators — no real PLC required for unit or integration tests.
- LWT must be configured for every device connection (detects unexpected disconnects).
- Security: OPC-UA security mode `SignAndEncrypt` is the target for production; `None` is acceptable in local dev only.

## Alternatives Considered

**REST/HTTP from PLCs** — Some modern PLCs expose REST APIs, but coverage is narrow and latency is higher. OPC-UA is the standard choice.

**DDS (Data Distribution Service)** — Used in robotics (ROS2). Out of scope for industrial PLC integration.

**AMQP (RabbitMQ)** — More complex broker setup; MQTT is simpler and purpose-built for IoT telemetry.

**SparkplugB (MQTT extension)** — Standardized MQTT payload schema for IIoT. Worth revisiting once baseline MQTT is working.
