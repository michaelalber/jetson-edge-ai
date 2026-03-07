# 002: .NET Blazor for Monitoring Dashboard

**Status:** Accepted
**Date:** 2026-03-07

## Context

Monitoring dashboards and administrative interfaces require a structured UI framework.
The developer has existing .NET/C# expertise and wants to apply it to this project alongside
Python AI work.

## Decision

Use .NET 10 Blazor Server for the monitoring dashboard (`projects/monitoring-dashboard`).
Blazor Server's SignalR-based model suits real-time metric streaming without a separate
WebSocket server. Target .NET 10 with nullable reference types enabled and warnings as errors.

## Consequences

- Blazor Server requires a persistent connection; acceptable for a LAN-deployed Jetson dashboard
- Keeps UI logic in C# — no TypeScript/React context switching
- Real-time updates via SignalR are a natural fit for live sensor and inference metrics
- Dashboard communicates with Python services via REST or gRPC; a defined API boundary is needed
