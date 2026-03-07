# Visual Inspection

Real-time camera-based defect detection pipeline running on the Jetson Orin Nano.

## Goal

Capture frames from a USB or CSI camera, run an object detection / classification model
via TensorRT, and stream annotated results to the monitoring dashboard.

## Stack

- Python 3.10+
- OpenCV — camera capture and frame preprocessing
- TensorRT / ONNX Runtime — optimized inference
- FastAPI — lightweight REST API for result streaming
- Pydantic v2 — data validation and settings

## Status

Planning — not yet started.

## Planned Structure

```
visual-inspection/
├── src/
│   └── visual_inspection/
│       ├── capture/        # Camera capture and frame buffering
│       ├── inference/      # TensorRT engine loading and inference
│       ├── postprocess/    # Bounding box decode, NMS, annotation
│       └── api/            # FastAPI result streaming endpoint
├── tests/
├── models/                 # Model configs (weights in models/ at repo root)
├── pyproject.toml
└── README.md
```

## Key References

- [Jetson Inference Library](https://github.com/dusty-nv/jetson-inference)
- [TensorRT Python API](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/)
