# Jetson Orin Nano — Initial Setup

## Prerequisites

- Jetson Orin Nano Developer Kit
- microSD card (64 GB+ UHS-I recommended) or NVMe SSD
- USB-C power supply (5V/3A minimum)
- HDMI display and USB keyboard/mouse for initial boot

## 1. Flash JetPack

1. Download and install [NVIDIA SDK Manager](https://developer.nvidia.com/sdk-manager) on a host Ubuntu machine
2. Connect Jetson in recovery mode (hold RECOVERY button, then press POWER)
3. Flash JetPack 6.x via SDK Manager — includes L4T, CUDA, cuDNN, TensorRT

Alternatively, use the pre-flashed SD card image from the [Jetson Download Center](https://developer.nvidia.com/embedded/downloads).

## 2. First Boot Configuration

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Set power mode (check options with: sudo nvpmodel --query)
sudo nvpmodel -m 0   # 15W mode (maximum performance)
sudo jetson_clocks   # lock clocks at maximum
```

## 3. Install Docker with NVIDIA Container Toolkit

```bash
# Docker (if not pre-installed)
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# NVIDIA Container Toolkit
distribution=$(. /etc/os-release; echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list \
  | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt update && sudo apt install -y nvidia-container-toolkit
sudo systemctl restart docker

# Verify GPU access in containers
docker run --rm --runtime nvidia nvcr.io/nvidia/l4t-base:r36.2 nvidia-smi
```

## 4. Install Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh

# Pull a small model to verify
ollama pull llama3.2:3b-instruct-q4_K_M
ollama run llama3.2:3b-instruct-q4_K_M "Hello from Jetson"
```

## 5. Install Python Environment Tools

```bash
sudo apt install -y python3-pip python3-venv
pip3 install --upgrade pip

# Each project manages its own venv; example for a project:
cd projects/visual-inspection
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
```

## 6. Verify CUDA and TensorRT

```bash
python3 -c "import torch; print(torch.cuda.is_available()); print(torch.version.cuda)"
python3 -c "import tensorrt; print(tensorrt.__version__)"
```

## References

- [JetPack 6 Release Notes](https://developer.nvidia.com/embedded/jetpack-sdk-60)
- [Jetson Orin Nano Developer Guide](https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/)
- [NVIDIA L4T Docker Hub](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/l4t-base)
