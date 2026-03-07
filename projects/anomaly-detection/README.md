# Anomaly Detection

Statistical anomaly detection on sensor time-series data from the sensor pipeline.

## Goal

Consume timestamped sensor readings, detect outliers and drift, and emit alerts with
severity classification to the monitoring dashboard.

## Stack

- Python 3.10+
- Statistical methods: Z-score, IQR, EWMA, Isolation Forest
- Pydantic v2 — reading and alert models
- `asyncio` — streaming consumption

## Status

Planning — not yet started.

## Planned Structure

```
anomaly-detection/
├── src/
│   └── anomaly_detection/
│       ├── detectors/      # Pluggable detector implementations
│       ├── calibration/    # Baseline computation from historical data
│       ├── alerts/         # Alert classification and emission
│       └── models/         # Pydantic input/output models
├── tests/
├── pyproject.toml
└── README.md
```
