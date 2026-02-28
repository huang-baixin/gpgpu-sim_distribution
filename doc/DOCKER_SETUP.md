# 在 Mac 上使用 Docker 运行 GPGPU-Sim

## 前提：安装 Docker Desktop

从 https://www.docker.com/products/docker-desktop/ 下载并安装 Docker Desktop for Mac。

安装后打开 Docker Desktop，等待引擎启动（菜单栏鲸鱼图标变为稳定状态）。

建议在 Docker Desktop → Settings → Resources 中分配：
- CPU: 至少 4 核（推荐 8 核，编译更快）
- Memory: 至少 8 GB（推荐 16 GB）
- Disk: 至少 30 GB

验证安装：
```bash
docker --version
docker run hello-world
```

---

## 方案一：使用项目官方 CI 镜像（推荐，开箱即用）

这是项目 GitHub Actions CI 实际使用的镜像，包含 Ubuntu 22.04 + CUDA 11.7 + 全部构建依赖。

### 第一步：拉取镜像

```bash
docker pull tgrogers/accel-sim_regress:Ubuntu-22.04-cuda-11.7
```

> 镜像较大（约几 GB），首次拉取需要一些时间。

### 第二步：启动容器

```bash
cd /path/to/gpgpu-sim_distribution

# 启动交互式容器，把本地源码挂载进去
docker run -it --platform linux/amd64 \
  -v $(pwd):/home/gpgpu-sim:rw \
  -w /home/gpgpu-sim \
  --name gpgpu-sim-dev \
  tgrogers/accel-sim_regress:Ubuntu-22.04-cuda-11.7 \
  /bin/bash
```

参数说明：
- `--platform linux/amd64` — 重要！Mac Apple Silicon (M1/M2/M3/M4) 需要此参数启用 x86 模拟
- `-v $(pwd):/home/gpgpu-sim:rw` — 把本地代码目录挂载到容器内
- `-w /home/gpgpu-sim` — 设置工作目录
- `--name gpgpu-sim-dev` — 命名容器，方便后续操作

### 第三步：在容器内编译

```bash
# 查看 CUDA 安装路径
ls /usr/local/cuda*

# 设置环境变量（镜像中通常已设置，如未设置手动指定）
export CUDA_INSTALL_PATH=/usr/local/cuda

# 配置构建环境
source setup_environment release

# 编译（-j 并行编译）
make -j$(nproc)
```

### 第四步：运行一个简单测试

```bash
# 创建测试目录
mkdir -p /tmp/test && cd /tmp/test

# 复制一个 GPU 配置（Volta V100）
cp /home/gpgpu-sim/configs/tested-cfgs/SM7_QV100/gpgpusim.config .
cp /home/gpgpu-sim/configs/tested-cfgs/SM7_QV100/config_volta_islip.icnt .

# 编写一个最简单的 CUDA 程序
cat > vectorAdd.cu << 'CUDA_EOF'
#include <stdio.h>

__global__ void vectorAdd(float *a, float *b, float *c, int n) {
    int i = blockIdx.x * blockDim.x + threadIdx.x;
    if (i < n) c[i] = a[i] + b[i];
}

int main() {
    int n = 256;
    size_t size = n * sizeof(float);
    float *h_a = (float*)malloc(size);
    float *h_b = (float*)malloc(size);
    float *h_c = (float*)malloc(size);

    for (int i = 0; i < n; i++) {
        h_a[i] = i;
        h_b[i] = i * 2;
    }

    float *d_a, *d_b, *d_c;
    cudaMalloc(&d_a, size);
    cudaMalloc(&d_b, size);
    cudaMalloc(&d_c, size);

    cudaMemcpy(d_a, h_a, size, cudaMemcpyHostToDevice);
    cudaMemcpy(d_b, h_b, size, cudaMemcpyHostToDevice);

    vectorAdd<<<1, 256>>>(d_a, d_b, d_c, n);

    cudaMemcpy(h_c, d_c, size, cudaMemcpyDeviceToHost);

    printf("c[0]=%f, c[1]=%f, c[255]=%f\n", h_c[0], h_c[1], h_c[255]);

    cudaFree(d_a); cudaFree(d_b); cudaFree(d_c);
    free(h_a); free(h_b); free(h_c);
    return 0;
}
CUDA_EOF

# 编译 CUDA 程序（动态链接 cudart）
nvcc -o vectorAdd vectorAdd.cu -lcudart

# 确保 LD_LIBRARY_PATH 指向 GPGPU-Sim（source setup_environment 已设置）
# 验证动态链接
ldd vectorAdd | grep cudart
# 应该指向 gpgpu-sim 的 lib 目录，而非 /usr/local/cuda/lib64

# 运行！GPGPU-Sim 会自动拦截并模拟
./vectorAdd
```

你会看到 GPGPU-Sim 的启动信息、GPU 配置加载、kernel 执行过程和统计输出。

### 日常使用

```bash
# 退出容器
exit

# 重新进入已有容器
docker start -i gpgpu-sim-dev

# 删除容器（代码在本地，不会丢失）
docker rm gpgpu-sim-dev
```

---

## 方案二：自定义 Dockerfile（更灵活）

如果官方镜像拉取困难或想自定义环境，可以自己构建。

在项目根目录创建 `Dockerfile`：

```dockerfile
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV CUDA_INSTALL_PATH=/usr/local/cuda
ENV PATH=$CUDA_INSTALL_PATH/bin:$PATH

# 安装构建依赖
RUN apt-get update && apt-get install -y \
    build-essential \
    xutils-dev \
    bison \
    zlib1g-dev \
    flex \
    libglu1-mesa-dev \
    git \
    wget \
    doxygen \
    graphviz \
    vim \
    gdb \
    && rm -rf /var/lib/apt/lists/*

# 安装 CUDA Toolkit 11.7（仅 toolkit，不装驱动）
RUN wget -q https://developer.download.nvidia.com/compute/cuda/11.7.0/local_installers/cuda_11.7.0_515.43.04_linux.run \
    && sh cuda_11.7.0_515.43.04_linux.run --toolkit --silent --no-drm \
    && rm cuda_11.7.0_515.43.04_linux.run

WORKDIR /home/gpgpu-sim
```

构建和使用：

```bash
# 构建镜像（首次约 10-20 分钟）
docker build --platform linux/amd64 -t gpgpu-sim-env .

# 启动容器
docker run -it --platform linux/amd64 \
  -v $(pwd):/home/gpgpu-sim:rw \
  --name gpgpu-sim-dev \
  gpgpu-sim-env /bin/bash

# 容器内编译
source setup_environment release
make -j$(nproc)
```

---

## Apple Silicon (M1/M2/M3/M4) 注意事项

GPGPU-Sim 和 CUDA Toolkit 都是 x86_64 架构，在 Apple Silicon Mac 上通过 Docker 的
Rosetta 2 模拟层运行，需要注意：

1. **必须加 `--platform linux/amd64`** — 否则会拉取 ARM 镜像，CUDA toolkit 不可用

2. **性能开销** — x86 模拟大约有 2-5 倍的性能损失，编译和仿真都会比原生 Linux 慢。
   编译约需 5-15 分钟（取决于 CPU 核数），仿真速度也会相应降低。

3. **启用 Rosetta** — Docker Desktop → Settings → General → 勾选
   "Use Rosetta for x86_64/amd64 emulation on Apple Silicon"，性能会显著改善。

4. **内存建议调大** — Docker Desktop → Settings → Resources → Memory 设为 8-16 GB

---

## 常用操作速查

### 编译相关

```bash
# Release 编译
source setup_environment release && make -j$(nproc)

# Debug 编译（可 gdb 调试）
source setup_environment debug && make -j$(nproc)

# 清理
make clean

# 生成 doxygen 文档
make docs
```

### 切换 GPU 配置

```bash
# 可用配置（在 configs/tested-cfgs/ 下）:
#   SM2_GTX480        — Fermi GTX 480
#   SM3_KEPLER_TITAN  — Kepler Titan
#   SM6_TITANX        — Pascal Titan X
#   SM7_QV100         — Volta V100 (推荐)
#   SM7_TITANV        — Volta Titan V
#   SM75_RTX2060      — Turing RTX 2060
#   SM86_RTX3070      — Ampere RTX 3070

# 复制配置到工作目录
cp configs/tested-cfgs/SM7_QV100/* /your/working/dir/
```

### GDB 调试 GPGPU-Sim

```bash
# 用 debug 模式编译
source setup_environment debug
make -j$(nproc)

# 用 gdb 启动你的 CUDA 应用
cd /your/working/dir
gdb --args ./your_cuda_app

# 常用断点
(gdb) break gpgpu_sim::cycle
(gdb) break shader_core_ctx::issue
(gdb) break ldst_unit::cycle
(gdb) run
```

### 运行项目自带的回归测试

```bash
# 在容器内
export CONFIG=QV100
export CUDA_INSTALL_PATH=/usr/local/cuda
export GPUAPPS_ROOT=/path/to/benchmarks

source setup_environment
make -j$(nproc)

git clone https://github.com/accel-sim/accel-sim-framework.git
./accel-sim-framework/util/job_launching/run_simulations.py \
  -C $CONFIG -B rodinia_2.0-ft -N regress -l local
./accel-sim-framework/util/job_launching/monitor_func_test.py \
  -v -N regress -j procman
```

---

## 故障排查

### 问题：`source setup_environment` 报错找不到 CUDA

确保 `CUDA_INSTALL_PATH` 正确指向容器内的 CUDA 目录：
```bash
ls /usr/local/cuda*/bin/nvcc
export CUDA_INSTALL_PATH=/usr/local/cuda  # 或具体版本如 /usr/local/cuda-11.7
```

### 问题：ldd 显示 cudart 指向系统而非 GPGPU-Sim

重新 source 环境：
```bash
source setup_environment release
echo $LD_LIBRARY_PATH  # 确认包含 gpgpu-sim 的 lib 路径
ldd ./your_app | grep cudart
```

### 问题：Apple Silicon 上 docker pull 失败

显式指定平台：
```bash
docker pull --platform linux/amd64 tgrogers/accel-sim_regress:Ubuntu-22.04-cuda-11.7
```

### 问题：编译报 bison/flex 错误

确保容器内已安装：
```bash
apt-get update && apt-get install -y bison flex
```
