# Triton-Ascend Docker Images

<!-- [![Docker Pulls](https://img.shields.io/docker/pulls/yourusername/triton-ascend)](https://hub.docker.com/r/yourusername/triton-ascend)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue)](LICENSE) -->

This repository provides ready‑to‑use Docker images for **Triton-Ascend** – the Ascend NPU backend for the Triton compiler.  
Images are optimized for **Python 3.12**, **CANN 9.0.0**, **LLVM 22**, **torch_npu**, and **triton-ascend** (release 3.5.x).

Regional variants are available to speed up package downloads by using local mirrors (APT, PyPI, LLVM, Git).

## 🔧 Development with local Triton-Ascend source

The Docker image **does not** bundle the `triton-ascend` source code. To develop, debug, or modify Triton‑Ascend, you **must** mount your local clone of the repository.

### Step 1: Clone the repository on your host

```bash
git clone https://gitee.com/ascend/triton-ascend.git /path/to/local/triton-ascend
```

## 🧩 Using a local CANN installation

The Docker image does **not** bundle the Ascend CANN kernel operators (`.opp` files) because they are host‑specific and depend on the NPU driver version. To make the container work with your NPU, you must mount the host’s CANN directory into the container.

---

## 🚀 Quick Start

### 1. Copy neaded Dockerfile and build

### 2. Run the container (with NPU devices)

```bash
docker run -d --rm \
    --privileged \
    --cap-add=SYS_PTRACE \
    --device=/dev/davinci0 --device=/dev/davinci1 \
    --device=/dev/davinci2 --device=/dev/davinci3 \
    --device=/dev/davinci4 --device=/dev/davinci5 \
    --device=/dev/davinci6 --device=/dev/davinci7 \
    --device=/dev/davinci_manager \
    --device=/dev/devmm_svm  \
    --device=/dev/hisi_hdc \
    --net=host \
    --name <container_name> \
    -v /usr/local/dcmi:/usr/local/dcmi:ro \
    -v /usr/local/Ascend:/usr/local/Ascend:ro \
    -v /usr/local/Ascend/driver:/usr/local/Ascend/driver:ro \
    -v /etc/ascend_install.info:/etc/ascend_install.info:ro \
    -v /var/log/npu/:/usr/slog \
    -v /usr/local/sbin/npu-smi:/usr/local/sbin/npu-smi:ro \
    -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
    -v /<your_mspti_optional>/mspti/:/workspace/mspti \
    -v /<your_working_directory_optional>/triton_workspace/:/workspace \
    -v /<path_to_your_triton-ascend_folder>/triton-ascend/:/workspace/triton-ascend \
    <your_image_name>:latest
```

## 📄 License
This project is distributed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).