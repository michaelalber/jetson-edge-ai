# Industrial Gateway

OPC-UA client and MQTT publisher bridge on the Jetson Orin Nano. Reads process data from
industrial devices (PLCs, sensors) via OPC-UA or Modbus TCP and publishes it to an MQTT
broker for downstream consumers (anomaly detection, monitoring dashboard, cloud).

## Goal

Learn the core industrial automation protocols hands-on:
- Connect to an OPC-UA server (real PLC or simulator) and subscribe to node value changes
- Publish timestamped readings to MQTT with proper QoS and topic hierarchy
- Handle reconnection, LWT (Last Will and Testament), and graceful shutdown
- Simulate PLC data locally for development without hardware

## Stack

- Python 3.10+
- `asyncua` — async OPC-UA client
- `aiomqtt` — async MQTT client (wraps paho-mqtt)
- `pymodbus` — Modbus TCP client (legacy device support)
- Pydantic v2 — configuration and message schema validation
- `asyncio` — concurrent OPC-UA subscriptions and MQTT publishing

## Data Flow

```
OPC-UA Server (PLC/Simulator)
        |
        | asyncua subscription
        v
  GatewayService (Jetson)
        |
        | aiomqtt publish
        v
  MQTT Broker (Mosquitto)
        |
   +---------+----------+
   |         |          |
Dashboard  Anomaly   Cloud
           Detection
```

## MQTT Topic Convention

```
jetson/industrial-gateway/{device_id}/{measurement}

Examples:
  jetson/industrial-gateway/plc1/tank_level
  jetson/industrial-gateway/plc1/motor_rpm
  jetson/industrial-gateway/line1/temperature
```

## Status

Planning — not yet started.

## Planned Structure

```
industrial-gateway/
├── src/
│   └── industrial_gateway/
│       ├── opcua/          # OPC-UA client and subscription manager
│       ├── modbus/         # Modbus TCP client (legacy devices)
│       ├── mqtt/           # MQTT publisher with reconnect and LWT
│       ├── models/         # Pydantic schemas: ReadingEvent, GatewayConfig, NodeConfig
│       ├── simulator/      # PLC data simulator for local dev / testing
│       └── gateway.py      # Top-level orchestrator
├── tests/
├── config/
│   └── nodes.yaml          # OPC-UA NodeId / Modbus register map (not hardcoded)
├── pyproject.toml
└── README.md
```

## Key References

- [asyncua docs](https://asyncua.readthedocs.io/)
- [aiomqtt docs](https://aiomqtt.bo3hm.de/)
- [pymodbus docs](https://pymodbus.readthedocs.io/)
- [OPC Foundation Reference](https://reference.opcfoundation.org/)
- [MQTT 5.0 Specification](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html)
- [Mosquitto broker](https://mosquitto.org/)

## Local Development (no hardware)

Use the OPC-UA simulation server built into `asyncua`:

```bash
# Install asyncua simulation extras
pip install asyncua[all]

# Run the built-in simulation server (exposes demo nodes on opc.tcp://0.0.0.0:4840)
python -c "import asyncio; from asyncua.server.demo_server import DemoServer; ..."
```

Run Mosquitto broker locally via Docker:

```bash
docker run -d --name mosquitto -p 1883:1883 eclipse-mosquitto
```
