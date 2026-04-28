# Triton‑Ascend Development Container

### Based on ascendai/cann:9.0.0-beta.2-910b-ubuntu22.04-py3.11 - https://github.com/Ascend/cann-container-image/blob/main/cann/9.0.0-beta.2-910b-ubuntu22.04-py3.11/Dockerfile

## Prerequisites

- Docker installed on the host machine.
- Ascend NPU drivers and devices available.
- CANN 9.0.0beta.2 installed in container.
- **Your own local copy of the `triton-ascend` repository** (recommended branch: `release/3.5.x`).  
  Place it anywhere on your host, e.g. `/home/your_username/Projects/triton-ascend`.

- Run docker build **Dockerfile**

## Run command to build docker

```bash
    docker build -t triton-ascend-llvm-22-cann-9.0.0-beta.2_v1.0:latest .
```

## 1. Prepare your local source code

- Make sure your `triton-ascend` folder contains the `python` subdirectory.

## 2. Run the container

Adjust the paths and device names according to your environment, then run:

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
    -v /usr/local/Ascend/driver:/usr/local/Ascend/driver:ro \
     -v /usr/local/Ascend/driver/version.info:/usr/local/Ascend/driver/version.info:ro \
    -v /etc/ascend_install.info:/etc/ascend_install.info:ro \
    -v /var/log/npu/:/usr/slog \
    -v /usr/local/sbin/npu-smi:/usr/local/sbin/npu-smi:ro \
    -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
    -v /<your_mspti_optional>/mspti/:/workspace/mspti \
    -v /<your_working_directory_optional>/triton_workspace/:/workspace \
    -v /<path_to_your_triton-ascend_folder>/triton-ascend/:/workspace/triton-ascend \
    triton-ascend-llvm-22-cann-9.0.0-beta.2_v1.0:latest
```

### Important

- Replace /home/your_username/... with your actual paths.

- If you have fewer davinci devices, remove the corresponding lines.

## 3. Run the container

```bash
  docker exec -u root -it <container_name> /bin/bash
```

### Inside the container

```bash
/opt/scripts/build.sh
```

### (Optional) If you need to use proton with mspti

```bash
export LD_PRELOAD=/usr/local/Ascend/cann-9.0.0-beta.2/aarch64-linux/lib64/libmspti.so
```
