# 003: Ollama for On-Device LLM

**Status:** Accepted
**Date:** 2026-03-07

## Context

On-device LLM inference requires a runtime that manages model loading, GPU memory allocation,
and a simple API surface. The Jetson Orin Nano has 8 GB of unified LPDDR5 memory shared
between CPU and GPU.

## Decision

Use Ollama as the local LLM runtime. Ollama provides a REST API, handles model lifecycle,
and supports GGUF quantized models that fit within the Jetson's memory constraints.
Interact with Ollama via the Python `ollama` client library.

## Consequences

- Model selection is constrained to GGUF variants that fit in ~6 GB (leaving headroom for OS)
- Suitable models: Llama 3.2 3B Q4, Phi-3 Mini Q4, Gemma 2B Q4
- Ollama's REST API means the dashboard can also query it directly if needed
- No fine-tuning on-device — Ollama is for inference only
