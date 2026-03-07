# LLM Inference

On-device large language model inference using Ollama on the Jetson Orin Nano.

## Goal

Explore local LLM capabilities on resource-constrained hardware: conversational queries,
structured output extraction from sensor data, and basic RAG over local documents.

## Stack

- Python 3.10+
- Ollama — local LLM runtime and model management
- `ollama` Python client library
- FastAPI — optional REST wrapper for dashboard integration
- Pydantic v2 — prompt/response models

## Hardware Constraints

The Jetson Orin Nano has 8 GB unified LPDDR5 memory. Realistic model sizes:

| Model | Quant | Size | Notes |
|---|---|---|---|
| Llama 3.2 3B | Q4_K_M | ~2 GB | Good general performance |
| Phi-3 Mini 3.8B | Q4_K_M | ~2.3 GB | Strong reasoning, small footprint |
| Gemma 2B | Q4_K_M | ~1.5 GB | Fast inference |

## Status

Planning — not yet started.

## Planned Structure

```
llm-inference/
├── src/
│   └── llm_inference/
│       ├── client/         # Ollama client wrapper
│       ├── prompts/        # Versioned prompt templates
│       ├── rag/            # Document chunking, embedding, retrieval
│       └── api/            # FastAPI endpoint for dashboard queries
├── tests/
├── prompts/                # Prompt template files (not f-strings in code)
├── pyproject.toml
└── README.md
```
