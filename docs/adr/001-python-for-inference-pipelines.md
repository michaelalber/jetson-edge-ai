# 001: Python for Inference Pipelines

**Status:** Accepted
**Date:** 2026-03-07

## Context

The Jetson Orin Nano runs JetPack SDK which ships CUDA, TensorRT, and cuDNN libraries with
first-class Python bindings. All major ML frameworks (PyTorch, TensorFlow, ONNX Runtime)
provide GPU-accelerated Python wheels optimized for the Jetson L4T platform.

## Decision

Use Python 3.10+ for all Edge AI inference pipelines, sensor drivers, and data processing
scripts. Use `pyproject.toml` (not `setup.py`) for dependency management in each project.

## Consequences

- Broad ecosystem support: OpenCV, NumPy, TensorRT Python API, Pydantic, FastAPI all
  available without cross-compilation
- JetPack-provided Python wheels avoid library compatibility issues
- Each project manages its own virtual environment and `pyproject.toml`
- .NET handles UI concerns; Python handles inference — clear separation of responsibilities
