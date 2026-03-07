# Monitoring Dashboard

Real-time monitoring and device management UI for the Jetson Orin Nano projects.

## Goal

Provide a live view of inference results, sensor readings, anomaly alerts, and device health
metrics. Allow basic administrative actions (start/stop pipelines, adjust thresholds).

## Stack

- .NET 10 Blazor Server
- SignalR — real-time metric streaming to the browser
- REST/gRPC clients — consume Python service APIs

## Status

Planning — not yet started.

## Planned Structure

```
monitoring-dashboard/
├── src/
│   └── Dashboard/
│       ├── Features/           # Vertical slice feature folders
│       │   ├── Inference/      # Visual inspection results
│       │   ├── Sensors/        # Sensor readings and alerts
│       │   └── DeviceHealth/   # Jetson system metrics
│       ├── Shared/             # Shared Blazor components
│       └── Program.cs
├── tests/
│   └── Dashboard.Tests/
└── Dashboard.sln
```

## Key References

- [Blazor Server documentation](https://learn.microsoft.com/en-us/aspnet/core/blazor/)
- [SignalR with Blazor](https://learn.microsoft.com/en-us/aspnet/core/blazor/tutorials/signalr-blazor)
