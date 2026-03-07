# Sensor Pipeline

I2C, SPI, UART, and GPIO sensor data ingestion on the Jetson Orin Nano.

## Goal

Read data from industrial sensors (temperature, pressure, vibration, proximity) via the
Jetson's 40-pin header, apply calibration, and publish timestamped readings to downstream
consumers (anomaly detection, dashboard).

## Stack

- Python 3.10+
- `smbus2` / `spidev` / `pyserial` — hardware protocol drivers
- Pydantic v2 — sensor reading validation and configuration
- `asyncio` — concurrent polling of multiple sensors

## Status

Planning — not yet started.

## Planned Structure

```
sensor-pipeline/
├── src/
│   └── sensor_pipeline/
│       ├── drivers/        # Protocol-specific drivers (I2C, SPI, GPIO)
│       ├── calibration/    # Sensor calibration and scaling
│       ├── models/         # Pydantic reading/config models
│       └── publisher/      # Data publishing (file, queue, REST)
├── tests/
├── pyproject.toml
└── README.md
```

## Key References

- [Jetson Orin Nano 40-Pin Header Pinout](https://jetsonhacks.com/nvidia-jetson-orin-nano-gpio-header-pinout/)
- [smbus2 docs](https://smbus2.readthedocs.io/)
