# Models

Model configuration files and conversion scripts. Model weights are excluded from version
control (they are large binary files — use model registries or download scripts instead).

## Structure

```
models/
├── configs/            # Model architecture configs and hyperparameters
├── conversion/         # Scripts to convert ONNX -> TensorRT engine
└── download/           # Scripts to pull weights from Hugging Face / NVIDIA NGC
```

## Policy

- **Include:** config files (`.yaml`, `.json`), conversion scripts (`.py`), `requirements.txt`
  for conversion environments
- **Exclude:** weight files (`.pt`, `.onnx`, `.engine`, `.gguf`, `.bin`) — add these patterns
  to `.gitignore` as projects are created
