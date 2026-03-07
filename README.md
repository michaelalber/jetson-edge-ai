# jetson-edge-ai

Edge AI and Industrial Automation on the NVIDIA Jetson Orin Nano — a learning repository
exploring real-time inference, sensor integration, and .NET-based monitoring dashboards.

## Goal

Build practical, end-to-end Edge AI projects using:
- **Python** — inference pipelines, sensor drivers, Ollama LLM integration
- **.NET / C# (Blazor)** — monitoring dashboards and administrative interfaces
- **TensorRT / ONNX** — model optimization for on-device inference
- **Docker** — reproducible, containerized deployments on the Jetson

## Repository Structure

```
jetson-edge-ai/
├── docs/
│   ├── adr/                    # Architecture Decision Records
│   └── setup/                  # Jetson and local dev environment guides
├── projects/
│   ├── visual-inspection/      # Camera-based defect detection (Python + TensorRT)
│   ├── sensor-pipeline/        # I2C / SPI / GPIO data ingestion (Python)
│   ├── anomaly-detection/      # Statistical anomaly detection on sensor streams (Python)
│   ├── llm-inference/          # On-device LLM with Ollama (Python)
│   └── monitoring-dashboard/   # Real-time monitoring UI (.NET Blazor)
├── shared/                     # Utilities shared across 2+ projects (added when needed)
├── infrastructure/
│   ├── docker/                 # Container definitions per project
│   └── scripts/                # Provisioning and deployment scripts
└── models/                     # Model configs and conversion scripts (weights excluded)
```

## Projects

| Project | Description | Stack |
|---|---|---|
| [visual-inspection](projects/visual-inspection/) | Real-time defect detection via camera | Python, OpenCV, TensorRT |
| [sensor-pipeline](projects/sensor-pipeline/) | Sensor data ingestion and streaming | Python, I2C/SPI/GPIO |
| [anomaly-detection](projects/anomaly-detection/) | Outlier detection on sensor time-series | Python, statistical models |
| [llm-inference](projects/llm-inference/) | Local LLM chat and RAG experiments | Python, Ollama |
| [monitoring-dashboard](projects/monitoring-dashboard/) | Live metrics and device management UI | .NET 10, Blazor |

## Getting Started

See [docs/setup/jetson-setup.md](docs/setup/jetson-setup.md) for Jetson Orin Nano provisioning
and [docs/setup/development.md](docs/setup/development.md) for local development environment setup.

## Hardware

- **Device:** NVIDIA Jetson Orin Nano
- **JetPack SDK:** 6.x (L4T r36)
- **Documentation:** [Jetson Orin Nano Developer Guide](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/)
