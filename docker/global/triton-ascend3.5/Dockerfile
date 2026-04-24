FROM docker.m.daocloud.io/library/ubuntu:24.04

ARG CHIP_TYPE=A2
ARG CHIP_NAME=910b
ARG CANN_VERSION=9.0.0-beta.2
ARG CANN_URL_VERSION=9.0.T511
ARG LLVM_COMMIT="fad3272286528b8a491085183434c5ad4b59ab92"
ENV DEBIAN_FRONTEND=noninteractive

ENV SSL_CERT_FILE=/dev/null

RUN apt-get update && apt-get install -y \
    gnupg \
    ca-certificates \
    lsb-release \
    gcc \
    libbz2-dev \
    libffi-dev \
    liblzma-dev \
    zlib1g-dev \
    build-essential \
    wget \
    curl \
    git \
    libreadline-dev \
    libssl-dev \
    libsqlite3-dev \
    && rm -rf /var/lib/apt/lists/*

# Установка pyenv
RUN wget --no-check-certificate -O /tmp/Python-3.11.9.tar.xz \
        "https://mirrors.tuna.tsinghua.edu.cn/python/3.11.9/Python-3.11.9.tar.xz" && \
    tar -xf /tmp/Python-3.11.9.tar.xz -C /tmp && \
    cd /tmp/Python-3.11.9 && \
    ./configure --enable-optimizations --prefix=/usr/local && \
    make -j$(nproc) && \
    make altinstall && \
    cd / && rm -rf /tmp/Python-3.11.9 /tmp/Python-3.11.9.tar.xz

# Создание символических ссылок (опционально)
RUN ln -sf /usr/local/bin/python3.11 /usr/bin/python3 && \
    ln -sf /usr/local/bin/python3.11 /usr/bin/python && \
    ln -sf /usr/local/bin/pip3.11 /usr/bin/pip3 && \
    ln -sf /usr/local/bin/pip3.11 /usr/bin/pip

# Создание виртуального окружения (рекомендуется)
RUN python3.11 -m venv /opt/venv/triton_env
ENV PATH="/opt/venv/triton_env/bin:$PATH"

# Проверка (опционально)
RUN python --version && pip --version


# Install system dependencies and Python 3.11
RUN apt-get update && apt-get install -y --no-install-recommends \
    patchelf ccache git \
    clang-15 lld-15 libssl-dev libbz2-dev libffi-dev liblzma-dev file \
    software-properties-common && \
    apt-get update && \
    apt-get clean



# 2. Создаем пользователя HwHiAiUser, как в руководстве
RUN groupadd HwHiAiUser && \
    useradd -g HwHiAiUser -d /home/HwHiAiUser -m HwHiAiUser -s /bin/bash


# Создание рабочей директории для установочных файлов
WORKDIR /tmp/packages

# Проверка и установка CANN Toolkit
RUN TOOLKIT_FILE="Ascend-cann_${CANN_VERSION}_linux-aarch64.run"; \
    if [ -f "$TOOLKIT_FILE" ]; then \
        echo "✅ Using local $TOOLKIT_FILE"; \
        cp "$TOOLKIT_FILE" /tmp/; \
    else \
        echo "📦 Downloading CANN toolkit..."; \
        wget --no-check-certificate -O /tmp/"$TOOLKIT_FILE" \
            "https://ascend-repo.obs.cn-east-2.myhuaweicloud.com/CANN/CANN%20${CANN_URL_VERSION}/Ascend-cann_${CANN_VERSION}_linux-aarch64.run"; \
    fi && \
    chmod +x /tmp/"$TOOLKIT_FILE" && \
    /tmp/"$TOOLKIT_FILE" --install --quiet

# Проверка и установка Kernel-пакетов
RUN KERNEL_FILE="Ascend-cann-${CHIP_NAME}-ops_${CANN_VERSION}_linux-aarch64.run"; \
    if [ -f "$KERNEL_FILE" ]; then \
        echo "✅ Using local $KERNEL_FILE"; \
        cp "$KERNEL_FILE" /tmp/; \
    else \
        echo "📦 Downloading kernel package..."; \
        wget --no-check-certificate -O /tmp/"$KERNEL_FILE" \
            "https://ascend-repo.obs.cn-east-2.myhuaweicloud.com/CANN/CANN%20${CANN_URL_VERSION}/Ascend-cann-${CHIP_NAME}-ops_${CANN_VERSION}_linux-aarch64.run"; \
    fi && \
    chmod +x /tmp/"$KERNEL_FILE" && \
    /tmp/"$KERNEL_FILE" --install --quiet

# Переменные окружения CANN
ENV ASCEND_INSTALL_PATH=/usr/local/Ascend
ENV ASCEND_OPP_PATH=/usr/local/Ascend/opp
ENV ASCEND_AICPU_PATH=/usr/local/Ascend/ascend-toolkit
ENV LD_LIBRARY_PATH=/usr/local/Ascend/driver/lib64:/usr/local/Ascend/ascend-toolkit/latest/lib64:/usr/local/Ascend/cann-9.0.0/aarch64-linux/lib64:/usr/local/Ascend/cann-9.0.0/aarch64-linux/lib64/device/lib64:/usr/local/Ascend/cann-9.0.0/aarch64-linux/devlib/linux/aarch64:${LD_LIBRARY_PATH}
ENV PATH=/usr/local/Ascend/ascend-toolkit/latest/bin:${PATH}

# Настройка окружения
ENV ASCEND_HOME=/usr/local/Ascend
ENV LD_LIBRARY_PATH=$ASCEND_HOME/driver/lib64:$ASCEND_HOME/ascend-toolkit/latest/lib64:$LD_LIBRARY_PATH
ENV PATH=$ASCEND_HOME/ascend-toolkit/latest/bin:$PATH

# RUN python3 -c "import acl;print(acl.get_soc_name())"

# Update CMake
RUN pip install --break-system-packages cmake==3.28.1 ninja pybind11
RUN ln -sf /usr/bin/python3.11 /usr/bin/python3
ENV PATH="/usr/local/bin:${PATH}"
RUN git config --global http.sslVerify false
RUN if [ -f /opt/llvm-22/bin/llvm-config ] && [ -x /opt/llvm-22/bin/llvm-config ]; then \
    echo "✅ LLVM 22 already installed at /opt/llvm-22. Skipping build."; \
    else \
    echo "🔨 Building LLVM 22 with MLIR (30–40 min)..."; \
    LLVM_PREFIX="/opt/llvm-22"; \
    update-alternatives --install /usr/bin/clang clang /usr/bin/clang-15 100 && \
    update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-15 100 && \
    update-alternatives --install /usr/bin/lld lld /usr/bin/lld-15 100 && \
    if [ ! -d /tmp/llvm-project ]; then \
    # git clone --progress --no-checkout https://mirrors.tuna.tsinghua.edu.cn/git/llvm-project.git /tmp/llvm-project; \
    git clone --progress --no-checkout https://gitcode.com/GitHub_Trending/ll/4-project.git /tmp/llvm-project; \
    else \
    echo "Using existing /tmp/llvm-project (cache)"; \
    fi && \
    cd /tmp/llvm-project && git checkout $LLVM_COMMIT && \
    wget -c --show-progress -O fad3272.patch https://raw.gitcode.com/Ascend/triton-ascend/blobs/2b0a06eb21438359d6d0576b622e3bb5e0292d17/fad3272.patch && \
    git apply fad3272.patch && \
    mkdir -p build && cd build && \
    cmake ../llvm -G Ninja \
    -DCMAKE_C_COMPILER=/usr/bin/clang-15 \
    -DCMAKE_CXX_COMPILER=/usr/bin/clang++-15 \
    -DCMAKE_LINKER=/usr/bin/lld-15 \
    -DCMAKE_BUILD_TYPE=Release \
    -DLLVM_ENABLE_ASSERTIONS=ON \
    -DLLVM_ENABLE_PROJECTS="mlir;clang;lld;llvm" \
    -DLLVM_TARGETS_TO_BUILD="host;NVPTX;AMDGPU" \
    -DLLVM_ENABLE_LLD=ON \
    -DCMAKE_INSTALL_PREFIX=$LLVM_PREFIX && \
    ninja install && \
    if [ -f bin/FileCheck ]; then cp bin/FileCheck $LLVM_PREFIX/bin/; fi; \
    fi

ENV LLVM_SYSPATH=/opt/llvm-22
ENV PATH="/opt/llvm-22/bin:${PATH}"
ENV LD_LIBRARY_PATH="/opt/llvm-22/lib:${LD_LIBRARY_PATH}"

RUN mkdir -p /opt/scripts && \
    printf '%s\n' \
    '#!/bin/bash' \
    'set -e' \
    '# ------------------------------------------------------------------' \
    '# 1. Install CMake 3.28+ (required by AscendNPU-IR)' \
    '# ------------------------------------------------------------------' \
    'current_cmake_version=$(cmake --version | head -n1 | cut -d" " -f3)' \
    'required_cmake_version="3.28.1"' \
    'apt-get update' \
    'apt-get install -y vim libboost-dev zlib1g-dev zlib1g' \
    'apt-get remove -y ninja-build' \
    'pip install ninja pyyaml numpy' \
    'pip install --upgrade setuptools wheel' \
    'apt-get update && apt-get install -y nlohmann-json3-dev' \
    'export JSON_SYSPATH=/usr/include/nlohmann' \
    'source /usr/local/Ascend/cann-9.0.0-beta.2/set_env.sh' \
    'export PATH=/usr/local/bin:$PATH' \
    'if [ "$(printf '\''%s\n'\'' "$required_cmake_version" "$current_cmake_version" | sort -V | head -n1)" = "$required_cmake_version" ]; then' \
    '    echo "✅ CMake version $current_cmake_version >= $required_cmake_version, skipping upgrade."' \
    'else' \
    '    apt-get remove -y cmake 2>/dev/null || true' \
    '    echo "🔧 Upgrading CMake from $current_cmake_version to $required_cmake_version..."' \
    '    CMAKE_VERSION="3.28.1"' \
    '    python3 -m pip install cmake==$CMAKE_VERSION' \
    '    if [ ! -f /usr/local/bin/cmake ]; then' \
    '        ln -sf $(python3 -c "import sys; print(sys.prefix)")/bin/cmake /usr/local/bin/cmake' \
    '    fi' \
    '    export PATH=/usr/local/bin:$PATH' \
    '    cmake --version' \
    'fi' \
    '' \
    '' \
    '# ------------------------------------------------------------------' \
    '# 3. Install torch and torch_npu (if needed)' \
    '# ------------------------------------------------------------------' \
    'TORCH_URL="https://download.pytorch.org/whl/cpu/torch-2.10.0%2Bcpu-cp311-cp311-manylinux_2_28_aarch64.whl"' \
    'TORCH_FILE=$(basename "$TORCH_URL")' \
    'wget --no-check-certificate --progress=bar:force -O "/tmp/$TORCH_FILE" "$TORCH_URL"' \
    'pip3 install /tmp/torch-2.10.0%2Bcpu-cp311-cp311-manylinux_2_28_aarch64.whl' \
    'rm "/tmp/$TORCH_FILE"' \
    'echo "=== 3. Installing torch_npu ==="' \
    'TORCH_NPU_URL="https://gitcode.com/Ascend/pytorch/releases/download/v26.0.0-beta.1-pytorch2.10.0/torch_npu-2.10.0rc3-cp311-cp311-manylinux_2_28_aarch64.whl"' \
    'wget --no-check-certificate --progress=bar:force  -O /tmp/torch_npu.whl "$TORCH_NPU_URL"' \
    'if file /tmp/torch_npu.whl | grep -q "Zip archive data"; then' \
    '    FILENAME=$(basename "$TORCH_NPU_URL" | cut -d "?" -f1)' \
    '    mv /tmp/torch_npu.whl "/tmp/$FILENAME"' \
    '    python3 -m pip install "/tmp/$FILENAME"' \
    '    rm "/tmp/$FILENAME"' \
    'else' \
    '    echo "ERROR: not a valid wheel"' \
    '    exit 1' \
    'fi' \
    'echo "=== 4. Building triton-ascend from /workspace/triton-ascend ==="' \
    'if [ ! -d "/workspace/triton-ascend/python" ]; then' \
    '    echo "ERROR: /workspace/triton-ascend not mounted"' \
    '    exit 1' \
    'fi' \
    '# ------------------------------------------------------------------' \
    '# 4. Build Triton-Ascend from source (mounted at /workspace/triton-ascend)' \
    '# ------------------------------------------------------------------' \
    'TRITON_SRC="/workspace/triton-ascend"' \
    'if [ ! -d "$TRITON_SRC/python" ]; then' \
    '    echo "ERROR: Triton-Ascend source not found at $TRITON_SRC"' \
    '    exit 1' \
    'fi' \
    'cd $TRITON_SRC' \
    'GCC_VERSION=$(gcc -dumpversion | cut -d. -f1)' \
    'if [ "$GCC_VERSION" -lt 9 ]; then' \
    '    echo "GCC version <9 detected. Patching CMakeLists.txt to link stdc++fs..."' \
    '    sed -i '\''/if (NOT WIN32 AND NOT APPLE)/,/endif()/ s/^#//'\'' CMakeLists.txt' \
    'fi' \
    'export LLD_DIR=/opt/llvm-22/lib/cmake/lld' \
    'export TRITON_APPEND_CMAKE_ARGS="-DTRITON_BUILD_UT=OFF -DLLD_DIR=$LLD_DIR"' \
    'echo "LLD_DIR = $LLD_DIR"' \
    'echo "TRITON_APPEND_CMAKE_ARGS = $TRITON_APPEND_CMAKE_ARGS"' \
    'mkdir -p /root/.triton/llvm/llvm-fad32722-ubuntu-arm64/bin' \
    'ln -sf /opt/llvm-22/bin/FileCheck /root/.triton/llvm/llvm-fad32722-ubuntu-arm64/bin/FileCheck' \
    'export TRITON_BUILD_PROTON=ON' \
    'export TRITON_CODEGEN_BACKENDS="ascend;nvidia;amd"' \
    'LLVM_SYSPATH=$LLVM_PREFIX' \
    'TRITON_BUILD_WITH_CCACHE=true' \
    'TRITON_BUILD_WITH_CLANG_LLD=true' \
    'TRITON_BUILD_PROTON=ON' \
    'TRITON_WHEEL_NAME="triton-ascend"' \
    'TRITON_APPEND_CMAKE_ARGS="-DTRITON_BUILD_UT=OFF -DLLD_DIR=/opt/llvm-22/lib/cmake/lld  -DTRITON_BUILD_PROTON=ON -DTRITON_CODEGEN_BACKENDS=ascend;nvidia;amd"' \
    'python3 setup.py install' \
    'source /usr/local/Ascend/ascend-toolkit/set_env.sh' \
    'echo "✅ Triton-Ascend installed successfully"' \
    'echo "=== Fixing triton._C module ==="' \
    'C_DIR=$(python -c "import triton; print(triton.__path__[0] + '\''/_C'\'')" 2>/dev/null || echo "/usr/local/lib/python3.11/dist-packages/triton/_C")' \
    'if [ -d "$C_DIR" ] && [ ! -f "$C_DIR/__init__.py" ]; then' \
    '    echo "from .libtriton import *" > "$C_DIR/__init__.py"' \
    'fi' \
    'export LD_PRELOAD=/usr/local/Ascend/cann-9.0.0-beta.2/aarch64-linux/lib64/libmspti.so' \
    'echo "=== 5. Verification ==="' \
    'cd ~' \
    'python3 -c "import torch; print(\"PyTorch:\", torch.__version__)"' \
    'python3 -c "import torch_npu; print(\"torch_npu:\", torch_npu.__version__)"' \
    'python3 -c "import triton; print(\"Triton:\", triton.__version__)"' \
    'echo "=== All done ==="' \
    > /opt/scripts/build.sh && chmod +x /opt/scripts/build.sh


WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
