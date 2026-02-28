# GPGPU-Sim 学习路径规划

## 前置知识（1-2 周）

在阅读源码之前，建议先掌握以下背景知识：

### GPU 架构基础
- SM（Streaming Multiprocessor）结构
- Warp 与 SIMT 执行模型
- 内存层次：Register → Shared Memory → L1 → L2 → DRAM
- GPU 线程层次：Grid → Block (CTA) → Warp → Thread

### CUDA 编程模型
- `cudaMalloc` / `cudaMemcpy` / `cudaLaunch` 基本 API
- Grid / Block / Thread 的映射关系
- Shared Memory 和同步原语（`__syncthreads`）

### PTX 指令集
- NVIDIA 虚拟 ISA（Parallel Thread Execution）
- 基本指令：`.reg`、`ld`、`st`、`add`、`bar.sync`
- 参考：[PTX ISA Reference](https://docs.nvidia.com/cuda/parallel-thread-execution/)

### 推荐论文
1. **Accel-Sim: An Extensible Simulation Framework for Validated GPU Modeling** (ISCA 2020)
   - Mahmoud Khairy, Zhesheng Shen, Tor M. Aamodt, Timothy G Rogers
   - GPGPU-Sim 4.0 核心论文，介绍了仿真框架的整体设计
   - https://arxiv.org/abs/1811.08933 (扩展版)

2. **Analyzing Machine Learning Workloads Using a Detailed GPU Simulator** (arXiv 2019)
   - Jonathan Lew, Deval Shah, Suchita Pati 等
   - 介绍 GPGPU-Sim 对 ML 工作负载的支持，含 CuDNN/PyTorch 集成
   - https://arxiv.org/abs/1811.08933

---

## 阶段 1：理解仿真入口（3-5 天）

**目标**：弄清"一个 CUDA 程序是怎么进入模拟器的"

### 阅读顺序

```
libcuda/cuda_runtime_api.cc  →  libcuda/gpgpu_context.h
    →  src/stream_manager.h/cc  →  src/gpgpusim_entrypoint.cc
```

### 重点文件

| 文件 | 关注点 |
|------|--------|
| `libcuda/cuda_runtime_api.cc` | `cudaMalloc`、`cudaLaunch` 如何被拦截和重定向 |
| `libcuda/gpgpu_context.h` | `gpgpu_context` 全局上下文结构，持有所有子系统的指针 |
| `libcuda/cuda_api_object.h` | `_cuda_device_id`、`CUctx_st`、`CUstream_st` 等模拟器对象 |
| `src/stream_manager.h/cc` | `stream_operation` 类型（kernel launch / memcpy）和流调度 |
| `src/gpgpusim_entrypoint.cc` | `gpgpu_sim_thread_concurrent()` 仿真线程主循环 |

### 关键问题
- [ ] `LD_LIBRARY_PATH` 替换机制如何让应用调用到模拟器的 `libcudart.so`？
- [ ] `cudaLaunch` 如何变成 `stream_operation` 被推入 `stream_manager`？
- [ ] 仿真线程如何从 `stream_manager` 取出操作并驱动 GPU 仿真？

---

## 阶段 2：PTX 功能仿真（1 周）

**目标**：理解"PTX 代码是怎么被解析和执行的"

### 阅读顺序

```
src/cuda-sim/ptx_loader.cc  →  src/cuda-sim/ptx_parser.h
    →  src/cuda-sim/ptx_ir.h  →  src/cuda-sim/cuda-sim.cc
    →  src/cuda-sim/instructions.cc
```

### 重点文件

| 文件 | 关注点 |
|------|--------|
| `src/cuda-sim/ptx_loader.cc` | PTX 从 fatbin 中提取和加载 |
| `src/cuda-sim/ptx_ir.h` | `ptx_instruction`、`function_info`、`operand_info` IR 定义 |
| `src/cuda-sim/ptx_sim.h` | `ptx_thread_info` 线程状态、`ptx_reg_t` 寄存器值 |
| `src/cuda-sim/cuda-sim.cc` | `functionalCoreSim::execute()` 功能仿真入口 |
| `src/cuda-sim/instructions.cc` | 各 PTX 操作码的 `*_impl` 实现（如 `add_impl`、`ld_impl`） |
| `src/abstract_hardware_model.h` | `warp_inst_t`、`kernel_info_t`、`mem_access_t` 核心抽象类型 |

### 关键问题
- [ ] PTX 文本如何经过 flex/bison 解析成 `ptx_instruction` IR？
- [ ] `function_info` 如何构建基本块和控制流图？
- [ ] `ptx_thread_info::ptx_exec_inst()` 的执行流程是什么？
- [ ] 功能仿真（`functionalCoreSim`）和时序仿真的关系是什么？

---

## 阶段 3：Shader Core 时序模型（1-2 周）⭐ 核心重点

**目标**：理解 GPU 核心的周期精确流水线模型

### 阅读顺序

```
src/gpgpu-sim/gpu-sim.cc (gpgpu_sim::cycle)
    →  src/gpgpu-sim/shader.h (shader_core_ctx 类定义)
    →  src/gpgpu-sim/shader.cc (流水线各阶段实现)
```

### 重点文件

| 文件 | 关注点 |
|------|--------|
| `src/gpgpu-sim/gpu-sim.h/cc` | `gpgpu_sim::cycle()` 顶层调度，四个时钟域（Core/L2/DRAM/ICNT） |
| `src/gpgpu-sim/shader.h` | `shader_core_ctx` 完整类定义，所有流水线阶段声明 |
| `src/gpgpu-sim/shader.cc` | `fetch()` → `decode()` → `issue()` → `execute()` → `writeback()` |
| `src/gpgpu-sim/scoreboard.h/cc` | `Scoreboard` 寄存器记分板，WAR/WAW 冒险检测 |
| `src/gpgpu-sim/stack.h/cc` | `simt_stack` PDOM 重汇聚栈，处理分支分歧 |

### 子模块深入

| 子模块 | 文件位置 | 说明 |
|--------|----------|------|
| Warp 调度器 | `shader.h` 中 `scheduler_unit` | LRR / GTO / Two-Level / SWL 多种策略 |
| 操作数收集器 | `shader.h` 中 `opndcoll_rfu_t` | 寄存器 bank 仲裁和操作数收集 |
| 功能单元 | `shader.h` 中 `simd_function_unit` | SP / SFU / DP / INT / Tensor Core |
| Load/Store 单元 | `shader.cc` 中 `ldst_unit` | L1D/L1C/L1T/SharedMem 访问 |
| CTA 调度 | `shader.h` 中 `simt_core_cluster` | Block 到 SM 的分配 |

### 关键问题
- [ ] `gpgpu_sim::cycle()` 如何驱动各时钟域？
- [ ] `shader_core_ctx` 的流水线各阶段如何交互？
- [ ] Warp 调度器如何选择下一个执行的 warp？
- [ ] SIMT 栈如何处理分支分歧和重汇聚？
- [ ] CTA 是如何被分配到 SM 上的？

---

## 阶段 4：内存层次（1-2 周）

**目标**：理解从 L1 到 DRAM 的完整内存层次模型

### 阅读顺序

```
shader.cc 中 ldst_unit  →  gpu-cache.h/cc (L1)
    →  l2cache.h/cc (L2 + 分区)  →  dram.h/cc (DRAM 控制器)
    →  mem_fetch.h/cc (请求包)  →  addrdec.h/cc (地址解码)
```

### 重点文件

| 文件 | 关注点 |
|------|--------|
| `src/gpgpu-sim/gpu-cache.h/cc` | `baseline_cache`、`l1_cache`、`l2_cache`、`tex_cache` 实现 |
| `src/gpgpu-sim/l2cache.h/cc` | `memory_partition_unit`、`memory_sub_partition` 分区管理 |
| `src/gpgpu-sim/dram.h/cc` | `dram_t` DRAM 控制器，bank 状态机 |
| `src/gpgpu-sim/dram_sched.h/cc` | `frfcfs_scheduler` FR-FCFS 调度算法 |
| `src/gpgpu-sim/mem_fetch.h/cc` | `mem_fetch` 内存请求/响应包 |
| `src/gpgpu-sim/addrdec.h/cc` | 线性地址 → 物理地址（chip/bank/row/col）解码 |

### 数据流路径

```
ldst_unit (L1 miss)
  → mem_fetch 创建
  → Interconnect 路由
  → memory_sub_partition (icnt_L2_queue)
  → L2 Cache 查找
  → L2 miss → L2_dram_queue
  → dram_t (frfcfs_scheduler)
  → 响应: dram_L2_queue → L2_icnt_queue → Interconnect → Shader Core
```

### 关键问题
- [ ] Cache 的 `access()` / `cycle()` / `fill()` 接口如何工作？
- [ ] MSHR（Miss Status Holding Register）如何合并请求？
- [ ] `memory_sub_partition` 中的多级 FIFO 队列如何连接 L2 和 DRAM？
- [ ] FR-FCFS 调度如何平衡行命中率和公平性？
- [ ] 地址解码如何决定请求去往哪个 memory partition？

---

## 阶段 5：互连网络（按需，3-5 天）

**目标**：理解 SM 与 Memory 之间的片上网络模型

### 重点文件

| 文件 | 关注点 |
|------|--------|
| `src/gpgpu-sim/icnt_wrapper.h/cc` | 与 GPU 仿真的接口封装 |
| `src/intersim2/interconnect_interface.hpp/cpp` | `Push` / `Pop` / `Advance` 桥接接口 |
| `src/intersim2/gputrafficmanager.hpp/cpp` | GPU 特定的流量管理 |
| `src/intersim2/routers/iq_router.hpp/cpp` | 输入排队路由器 |
| `src/intersim2/networks/` | 拓扑实现（Mesh / Torus / FatTree 等） |
| `src/gpgpu-sim/local_interconnect.h/cc` | 本地 Crossbar 互连 |

---

## 阶段 6：功耗模型与配置（按需，2-3 天）

### 功耗模型
- `src/accelwattch/` — AccelWattch 功耗模型
- `src/gpgpu-sim/power_interface.h/cc` — 功耗模型接口

### GPU 配置
- `configs/tested-cfgs/SM7_QV100/gpgpusim.config` — Volta V100 配置
- 理解各配置参数的含义（shader 数量、cache 大小、DRAM 时序等）

---

## 实践建议

### 1. 跟踪一个简单 kernel 的完整执行路径
编写最简单的 vectorAdd，用 debug 模式编译 GPGPU-Sim，加断点跟踪：
```
cudaMalloc → cudaMemcpy → cudaLaunch → stream_manager
  → gpgpu_sim::cycle → shader_core_ctx::fetch/decode/issue/execute/writeback
  → ldst_unit → cache → interconnect → DRAM
```

### 2. 善用调试输出
配置文件中开启：
```
-gpgpu_runtime_stat 100    # 每 100 个周期输出一次统计
-gpgpu_ptx_instruction_classification 1
```

### 3. 生成源码文档
```bash
make docs    # 生成 doxygen HTML 文档到 doc/doxygen/html/
```

### 4. 从 cycle() 出发
`gpgpu_sim::cycle()` 是整个时序仿真的心跳函数。理解了它就理解了全局调度逻辑。

### 5. 利用 GPU 配置文件
对照 `gpgpusim.config` 中的参数和代码中的 `gpgpu_sim_config` 类，能快速理解各模块的可配置项。

---

## 项目架构图

参见 `doc/gpgpu-sim-architecture.png` 和 `doc/gpgpu-sim-architecture.drawio`。
