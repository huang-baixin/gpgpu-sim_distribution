# GPGPU-Sim 环境搭建实录：在 Mac 上完整跑通

本文档记录了在 macOS (Apple Silicon) 上通过 Docker 从零搭建 GPGPU-Sim 开发环境并成功运行仿真的完整过程。

## 环境信息

| 项目 | 详情 |
|------|------|
| 主机系统 | macOS (darwin 24.6.0), Apple Silicon (arm64) |
| Docker | Docker Desktop (via Homebrew) |
| 容器系统 | Ubuntu 22.04 (x86_64, 通过 Rosetta 模拟) |
| GCC | 11.4.0 |
| CUDA Toolkit | 11.7 (V11.7.64) |
| GPGPU-Sim | 4.2.0 |
| 容器内 CPU 核心 | 8 |

## 磁盘空间占用

| 项目 | 大小 |
|------|------|
| Docker Desktop 应用 | ~2 GB |
| Docker 镜像 (gpgpu-sim-env) | 3.36 GB |
| GPGPU-Sim 编译产物 | ~57 MB |
| Docker VM 总占用 | ~10 GB |
| **建议预留** | **15 GB** |

---

## 第一步：安装 Docker Desktop

```bash
# 使用 Homebrew 安装（需已安装 Homebrew）
brew install --cask docker

# 启动 Docker Desktop
open -a Docker

# 等待引擎就绪（首次启动需 30-60 秒）
# 验证安装
docker --version
docker info
```

> Docker Desktop 会在菜单栏显示鲸鱼图标，引擎就绪后图标停止动画。

### 为什么需要 Docker Desktop 而不是只装 CLI？

Docker 容器是 Linux 内核功能（cgroups + namespaces），macOS 内核不支持。
在 Mac 上无论用什么方案都需要一个 Linux 虚拟机来运行容器：

- **Docker Desktop** = Docker CLI + Linux VM + GUI 管理
- 替代方案：Colima（纯 CLI，更轻量）、OrbStack（更快）

---

## 第二步：构建 Docker 镜像

项目根目录下已有 `Dockerfile`，内容如下：

```dockerfile
FROM --platform=linux/amd64 ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV CUDA_INSTALL_PATH=/usr/local/cuda
ENV PATH=/usr/local/cuda/bin:$PATH

# 安装构建依赖
RUN apt-get update && apt-get install -y \
    build-essential xutils-dev bison zlib1g-dev flex \
    libglu1-mesa-dev git wget vim gdb \
    && rm -rf /var/lib/apt/lists/*

# 安装 CUDA Toolkit 11.7（仅 toolkit，不装驱动）
RUN wget -q https://developer.download.nvidia.com/compute/cuda/11.7.0/local_installers/cuda_11.7.0_515.43.04_linux.run \
    && sh cuda_11.7.0_515.43.04_linux.run --toolkit --silent --no-drm \
    && rm cuda_11.7.0_515.43.04_linux.run

WORKDIR /home/gpgpu-sim
```

构建镜像：

```bash
cd /path/to/gpgpu-sim_distribution

# 构建（Apple Silicon 上约 30 分钟，主要时间花在下载和安装 CUDA Toolkit）
docker build --platform linux/amd64 -t gpgpu-sim-env .
```

各步骤耗时参考：

| 步骤 | 内容 | 耗时 |
|------|------|------|
| Step 1 | 拉取 Ubuntu 22.04 基础镜像 | ~10 秒 |
| Step 2 | apt-get 安装构建依赖 | ~7 分钟 |
| Step 3 | 下载并安装 CUDA 11.7 Toolkit (~2.5 GB) | ~20 分钟 |
| Step 4 | 设置工作目录 | <1 秒 |
| 导出镜像 | 写入 Docker 存储 | ~3 分钟 |
| **总计** | | **~30 分钟** |

验证镜像：

```bash
docker images gpgpu-sim-env
# 预期输出：
# REPOSITORY      TAG       SIZE
# gpgpu-sim-env   latest    3.36GB
```

### 替代方案：使用官方 CI 镜像

如果网络允许，也可以直接拉取项目 GitHub Actions 使用的官方镜像（更大但包含更多预装工具）：

```bash
docker pull --platform linux/amd64 tgrogers/accel-sim_regress:Ubuntu-22.04-cuda-11.7
```

---

## 第三步：启动容器

```bash
cd /path/to/gpgpu-sim_distribution

# 启动后台容器，挂载本地源码目录
docker run -d --platform linux/amd64 --privileged \
  -v $(pwd):/home/gpgpu-sim:rw \
  -w /home/gpgpu-sim \
  --name gpgpu-sim-dev \
  gpgpu-sim-env:latest \
  sleep infinity
```

参数说明：

| 参数 | 作用 |
|------|------|
| `-d` | 后台运行 |
| `--platform linux/amd64` | Apple Silicon 必须，指定 x86 平台 |
| `--privileged` | 允许 GDB 使用 ptrace（调试必须） |
| `-v $(pwd):/home/gpgpu-sim:rw` | 挂载本地源码到容器，双向同步 |
| `--name gpgpu-sim-dev` | 命名容器，方便后续操作 |
| `sleep infinity` | 保持容器运行 |

验证容器：

```bash
docker ps
# 应看到 gpgpu-sim-dev 状态为 Up

# 测试 CUDA 环境
docker exec gpgpu-sim-dev nvcc --version
# 预期输出：
# Cuda compilation tools, release 11.7, V11.7.64
```

---

## 第四步：编译 GPGPU-Sim

```bash
# 在容器内执行编译
docker exec gpgpu-sim-dev bash -c \
  'cd /home/gpgpu-sim && source setup_environment release && make -j$(nproc)'
```

编译输出关键信息：

```
GPGPU-Sim version 4.2.0 (build gpgpu-sim_git-commit-xxx) configured with AccelWattch.
setup_environment succeeded

        Building GPGPU-Sim version 4.2.0 ...

# ... flex/bison 生成解析器 ...
# ... g++ 编译各模块 ...
# ... 链接生成 libcudart.so ...
```

编译完成后，产物位于 `lib/gcc-11.4.0/cuda-11070/release/`：

```bash
ls -lh lib/gcc-11.4.0/cuda-11070/release/libcudart.so
# -rwxr-xr-x  57M  libcudart.so
```

> 编译耗时约 3 分钟（8 核并行），编译产物约 57 MB。

---

## 第五步：运行测试

### 准备测试文件

测试程序 `test/vectorAdd.cu` 已存在于项目中：

```cuda
__global__ void vectorAdd(float *a, float *b, float *c, int n) {
    int i = blockIdx.x * blockDim.x + threadIdx.x;
    if (i < n) c[i] = a[i] + b[i];
}
```

### 编译并运行

```bash
docker exec gpgpu-sim-dev bash -c '
  # 复制 GPU 配置文件（Volta V100）
  cd /home/gpgpu-sim/test
  cp /home/gpgpu-sim/configs/tested-cfgs/SM7_QV100/gpgpusim.config .
  cp /home/gpgpu-sim/configs/tested-cfgs/SM7_QV100/config_volta_islip.icnt .

  # 编译测试程序
  nvcc -o vectorAdd vectorAdd.cu -lcudart

  # 加载 GPGPU-Sim 环境（替换 LD_LIBRARY_PATH）
  cd /home/gpgpu-sim && source setup_environment release

  # 运行！
  cd /home/gpgpu-sim/test && ./vectorAdd
'
```

### 成功输出

程序启动后会看到：

```
*** GPGPU-Sim Simulator Version 4.2.0 ***

GPGPU-Sim PTX: simulation mode 0
```

然后是大量 GPU 配置参数（Volta V100 的完整微架构配置），接着是仿真过程。

最终输出：

```
gpgpu_simulation_time = 0 days, 0 hrs, 0 min, 1 sec (1 sec)
gpgpu_simulation_rate = 5376 (inst/sec)
gpgpu_simulation_rate = 5569 (cycle/sec)
gpgpu_silicon_slowdown = 203268x
Results: c[0]=0.0, c[1]=3.0, c[255]=765.0
Verification: PASSED (0 errors)
GPGPU-Sim: *** exit detected ***
```

关键信息解读：

| 指标 | 值 | 含义 |
|------|-----|------|
| simulation_time | 1 sec | 仿真壁钟时间 |
| simulation_rate | 5376 inst/sec | 每秒仿真的 GPU 指令数 |
| simulation_rate | 5569 cycle/sec | 每秒仿真的 GPU 时钟周期数 |
| silicon_slowdown | 203268x | 比真实 GPU 慢约 20 万倍（正常） |
| Verification | PASSED | 计算结果正确 |

---

## 日常使用

### 进入容器交互式终端

```bash
docker exec -it gpgpu-sim-dev bash

# 容器内：
source /home/gpgpu-sim/setup_environment release  # 或 debug
cd /home/gpgpu-sim/test
./vectorAdd
```

### 开发工作流

```
Mac 上用 Cursor 编辑代码
        ↓  (文件自动同步，通过 -v 挂载)
容器内编译和运行
        ↓
docker exec gpgpu-sim-dev bash -c 'cd /home/gpgpu-sim && source setup_environment release && make -j$(nproc)'
docker exec gpgpu-sim-dev bash -c 'cd /home/gpgpu-sim && source setup_environment release && cd test && ./vectorAdd'
```

### 容器管理

```bash
# 停止容器（不删除）
docker stop gpgpu-sim-dev

# 重新启动
docker start gpgpu-sim-dev

# 删除容器（源码在本地，不会丢失）
docker rm -f gpgpu-sim-dev

# 重建容器（如需要）
docker run -d --platform linux/amd64 --privileged \
  -v $(pwd):/home/gpgpu-sim:rw \
  -w /home/gpgpu-sim \
  --name gpgpu-sim-dev \
  gpgpu-sim-env:latest \
  sleep infinity
```

### GDB 调试

```bash
# 用 debug 模式重新编译
docker exec gpgpu-sim-dev bash -c \
  'cd /home/gpgpu-sim && source setup_environment debug && make clean && make -j$(nproc)'

# 进入容器，启动 GDB
docker exec -it gpgpu-sim-dev bash
cd /home/gpgpu-sim && source setup_environment debug
cd test
gdb --args ./vectorAdd

# GDB 中设置断点
(gdb) break gpgpu_sim::cycle
(gdb) break ptx_thread_info::ptx_exec_inst
(gdb) run
```

### 切换 GPU 配置

```bash
# 可选配置（在 configs/tested-cfgs/ 下）：
# SM2_GTX480, SM6_TITANX, SM7_QV100, SM7_TITANV,
# SM75_RTX2060, SM86_RTX3070

# 示例：切换到 RTX 3070
cp configs/tested-cfgs/SM86_RTX3070/* test/
```

---

## 故障排查

### Docker 引擎未启动

```bash
# 症状：docker: Cannot connect to the Docker daemon
open -a Docker   # 启动 Docker Desktop
# 等待 30-60 秒
```

### 容器名冲突

```bash
# 症状：The container name "/gpgpu-sim-dev" is already in use
docker rm -f gpgpu-sim-dev   # 强制删除旧容器
```

### 编译报错

```bash
# 清理后重新编译
docker exec gpgpu-sim-dev bash -c \
  'cd /home/gpgpu-sim && make clean && source setup_environment release && make -j$(nproc)'
```

### ldd 显示 cudart 未指向 GPGPU-Sim

```bash
# 确认 LD_LIBRARY_PATH
docker exec gpgpu-sim-dev bash -c \
  'cd /home/gpgpu-sim && source setup_environment release && echo $LD_LIBRARY_PATH && ldd test/vectorAdd | grep cudart'
# 应输出：libcudart.so => /home/gpgpu-sim/lib/gcc-11.4.0/cuda-11070/release/libcudart.so
```

---

## 相关文档

| 文档 | 路径 | 内容 |
|------|------|------|
| 项目架构分析 | `doc/gpgpu-sim-architecture.png` | 六层架构图 |
| 学习路径规划 | `doc/LEARNING_PATH.md` | 6 阶段源码阅读计划 |
| 推荐项目与课程 | `doc/RECOMMENDED_PROJECTS.md` | 体系结构/OS/网络/C++ 学习资源 |
| Accel-Sim 论文 | `doc/papers/Accel-Sim_ISCA2020.pdf` | GPGPU-Sim 4.0 核心论文 |
