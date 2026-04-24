# Triton‑Ascend Development Container

## Prerequisites

- Docker installed on the host machine.
- Ascend NPU drivers and devices available.
- **Your own local copy of the `triton-ascend` repository** (recommended branch: `release/3.5.x`).  
  Place it anywhere on your host, e.g. `/home/your_username/Projects/triton-ascend`.

## Base image

The Docker image `triton-ascend-llvm-22-cann-9.0.0_v3.2:latest` is **already available on the server**.  
You do **not** need to build it yourself – just use it directly in the `docker run` command.

- If docker images is not preseneted. Run **Dockerfile**

## Run command to build docker

```bash
    docker build -t triton-ascend-llvm-22-cann-9.0.0_v3.3:latest .
```

## 1. Prepare your local source code

- Make sure your `triton-ascend` folder contains the `python` subdirectory.
- Create the `install_triton.sh` script (see full content below) inside that folder.

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
    -v /etc/ascend_install.info:/etc/ascend_install.info:ro \
    -v /var/log/npu/:/usr/slog \
    -v /usr/local/sbin/npu-smi:/usr/local/sbin/npu-smi:ro \
    -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
    -v /<path_to_workspace>/triton_workspace/:/workspace \
    -v /<path_to_local_trito>/triton-ascend/:/workspace/triton-ascend \
    triton-ascend-llvm-22-cann-9.0.0_v3.3:latest
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
