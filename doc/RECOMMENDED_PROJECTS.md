# 计算机系统学习：开源项目与课程推荐

涵盖计算机体系结构、操作系统、计算机网络三大方向，按难度分级整理。

---

## 一、计算机体系结构

### 1.1 从零开始：逻辑门 → 计算机

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **Nand2Tetris** | HDL/Jack | 3k+ | https://github.com/nand2tetris/projects | 从与非门开始，逐步搭建 ALU → CPU → 汇编器 → 编译器 → OS。配套 Coursera 课程《From Nand to Tetris》，**强烈推荐的第一个项目** |
| **一生一芯 (ysyx)** | Chisel/Verilog | — | https://ysyx.oscc.cc/ | 中科院计算所出品，从零写一个 RISC-V 处理器并流片，中文文档完善，有社区和导师指导 |

### 1.2 CPU 模拟器 / 仿真器

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **QtRVSim** | C++ | 550+ | https://github.com/cvut/qtrvsim | RISC-V CPU 图形化模拟器，能实时观察流水线、Cache、分支预测，**学流水线最直观** |
| **RISC-V Emulator Book** | Rust | 130+ | https://github.com/d0iasm/book.rvemu | 手把手教你用 Rust 从零写 RISC-V 模拟器，10 步完成，能跑 xv6。在线阅读：https://book.rvemu.app/ |
| **SimpleCPU** | Verilog | 100+ | https://github.com/SimpleCPU/SimpleCPU | 开源 CPU 设计验证平台，含 MIPS + RISC-V 实现 |
| **Tiny8** | Python | 500+ | https://github.com/sql-hkr/tiny8 | 极简 CPU 模拟器，Python 实现，入门友好 |

### 1.3 专业级体系结构研究模拟器

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **gem5** | C++/Python | 1.5k+ | https://github.com/gem5/gem5 | 业界标准 CPU 体系结构模拟器，支持 ARM/x86/RISC-V，建模乱序执行/Cache 一致性/内存系统，**学 CPU 微架构的终极工具** |
| **GPGPU-Sim** | C++ | 800+ | https://github.com/gpgpu-sim/gpgpu-sim_distribution | GPU 体系结构模拟器，周期精确模拟 NVIDIA GPU，学 GPU 微架构 |
| **gem5-gpu** | C++ | — | https://github.com/gem5-gpu/gem5 | gem5 + GPGPU-Sim 集成，研究 CPU-GPU 异构系统 |
| **Sniper** | C++ | 400+ | https://github.com/snipersim/snipersim | 多核 CPU 模拟器，速度比 gem5 快，适合大规模实验 |

### 1.4 推荐学习路线

```
Nand2Tetris（建立直觉，2-3 周）
  → QtRVSim（理解流水线和 Cache，1 周）
  → RISC-V Emulator Book（自己写模拟器，2 周）
  → gem5（专业级 CPU 微架构，持续学习）
  → GPGPU-Sim（GPU 微架构）
```

### 1.5 推荐论文

- **Accel-Sim: An Extensible Simulation Framework for Validated GPU Modeling** (ISCA 2020)
  - GPGPU-Sim 4.0 核心论文
  - 本地副本：`doc/papers/Accel-Sim_ISCA2020.pdf`

- **Analyzing Machine Learning Workloads Using a Detailed GPU Simulator** (arXiv 2019)
  - GPGPU-Sim 对 ML 工作负载的支持
  - 本地副本：`doc/papers/Analyzing_ML_Workloads_GPU_Simulator.pdf`

---

## 二、操作系统

### 2.1 入门级：跟课程学

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **xv6-riscv** | C | 7k+ | https://github.com/mit-pdos/xv6-riscv | MIT 6.S081 课程配套，RISC-V 上的微型 Unix，代码仅 ~8000 行，配套教材。**操作系统入门首选** |
| **Pintos** | C | — | https://pintos-os.org/ | 斯坦福 CS140 课程配套，练习线程/用户程序/虚拟内存/文件系统，Lab 设计精良 |
| **eduOS** | C | 200+ | https://github.com/RWTH-OS/eduOS | 分阶段学习：bootloader → 多任务 → 同步 → 抢占式调度，每个阶段一个 Git branch |

配套课程：
- MIT 6.S081: https://pdos.csail.mit.edu/6.828/2025/schedule.html
- xv6 教材: https://pdos.csail.mit.edu/6.828/2025/xv6/book-riscv-rev4.pdf

### 2.2 中级：用 Rust 写 OS

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **rCore-Tutorial-v3** | Rust | 1.8k+ | https://github.com/rcore-os/rCore-Tutorial-v3 | 清华大学出品，RISC-V + Rust，9 章教程从裸机到文件系统，**中文社区最好的 OS 教程** |
| **Writing an OS in Rust (blog_os)** | Rust | 15k+ | https://github.com/phil-opp/blog_os | 极其流行的博客系列，从 bootloader 到内存管理到异步，x86_64。在线阅读：https://os.phil-opp.com/ |
| **xv6-rust** | Rust | — | https://github.com/Gogomoe/xv6-rust | 用 Rust 重写 xv6，适合同时学 Rust 和 OS |

配套资源：
- rCore 教程在线阅读: https://rcore-os.github.io/rCore-Tutorial-Book-v3/
- 开源操作系统训练营: https://github.com/LearningOS

### 2.3 高级：深入真实内核

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **Linux 0.11** | C | 1k+ | https://github.com/yuan-xy/Linux-0.11 | Linus 最初的 Linux 内核，~10000 行 C，配合《Linux 内核完全注释》阅读 |
| **Linux 内核** | C | — | https://github.com/torvalds/linux | 真实世界的终极参考，建议按子系统阅读（mm/、fs/、kernel/、net/） |

### 2.4 推荐学习路线

```
xv6-riscv + MIT 6.S081 Labs（最经典的 OS 入门，4-6 周）
  → rCore-Tutorial-v3（Rust 重写，加深理解，3-4 周）
  → Writing an OS in Rust（更现代的视角，按需）
  → Linux 0.11 源码阅读（理解真实内核，持续）
```

---

## 三、计算机网络

### 3.1 动手实现协议栈

#### 入门：跟课程做 Lab

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **Stanford CS144** | C++ | 5k+ | https://github.com/CS144/minnow | **计算机网络入门首选**。从零搭建完整 TCP/IP 协议栈：字节流 → 重组器 → TCP 收发 → 路由器，8 个 Lab 循序渐进 |
| **MIT 6.1810 Net Lab** | C | — | https://pdos.csail.mit.edu/6.1810/2025/labs/net.html | 在 xv6 里写 E1000 网卡驱动 + 实现 Ethernet/IP/UDP 协议栈，偏底层驱动视角 |

配套资源：
- CS144 官网: https://cs144.stanford.edu
- CS144 讲座视频: YouTube 搜索 "Stanford CS144"

#### 中级：用户态 TCP/IP 协议栈

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **level-ip** | C | 2k+ | https://github.com/saminiir/level-ip | 用户态 TCP/IP 栈，用 Linux TAP 设备收发真实网络包，能让 `curl` 通过你自己的协议栈上网。配套系列博客逐层讲解 |
| **tapip** | C | 650+ | https://github.com/chobits/tapip | 基于 TAP 的用户态 TCP/IP，实现了 ARP/IP/ICMP/TCP/Socket，代码简洁可读，**中文作者** |
| **Build Your Own Internet** | 多语言 | — | https://github.com/build-your-own-internet/byoi | 从零构建互联网的教程项目 |

#### 高级：工业级协议栈

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **lwIP** | C | — | https://savannah.nongnu.org/projects/lwip/ | 业界广泛使用的轻量级 TCP/IP 栈，嵌入式标配，代码质量高 |
| **smoltcp** | Rust | 3.8k+ | https://github.com/smoltcp-rs/smoltcp | Rust 实现的嵌入式 TCP/IP 栈，支持 IPv4/IPv6/TCP/UDP/DHCP，现代且安全 |
| **Linux 内核网络栈** | C | — | https://github.com/torvalds/linux/tree/master/net | 真实世界的终极参考，配合《Linux Networking Internals》阅读 |

### 3.2 网络模拟与仿真

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **Mininet** | Python | 5k+ | https://github.com/mininet/mininet | SDN 网络模拟器，在笔记本上创建含主机/交换机/控制器的虚拟网络拓扑，**学 SDN 和网络实验必备** |
| **ns-3** | C++/Python | — | https://www.nsnam.org/ | 专业级离散事件网络模拟器，可模拟 WiFi/LTE/TCP 等各种协议，学术论文标配 |
| **GNS3** | Python | 2k+ | https://github.com/GNS3/gns3-server | 网络设备模拟器，能模拟 Cisco/Juniper 路由器，适合学网络运维和路由协议 |

### 3.3 网络工具与协议分析

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **Wireshark** | C/C++ | 7k+ | https://github.com/wireshark/wireshark | 最知名的抓包工具，源码也是学习协议解析的绝佳材料 |
| **Scapy** | Python | 10k+ | https://github.com/secdev/scapy | Python 交互式数据包操作库，能构造/发送/嗅探/解析任意协议包，**网络协议瑞士军刀** |
| **Beej's Guide** | C | — | https://beej.us/guide/bgnet/ | 网络编程圣经级教程，讲 Socket API，免费在线阅读 |

### 3.4 特定协议方向

#### HTTP / Web

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **Tiny HTTPd** | C | 11k+ | https://github.com/EZLippi/Tinyhttpd | 500 行 C 实现的 HTTP 服务器，麻雀虽小五脏俱全 |
| **build-your-own-x** | 多语言 | 300k+ | https://github.com/codecrafters-io/build-your-own-x | 从零实现各种系统的教程合集（HTTP 服务器、DNS、Redis 等） |

#### DNS

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **Implement DNS in a Weekend** | Python | — | https://implement-dns.wizardzines.com/ | 一个周末从零实现 DNS 解析器，Julia Evans 出品 |

### 3.5 推荐学习路线

```
Stanford CS144（协议栈实现，4-6 周，首选）
  → level-ip 或 tapip（用户态真实网络包收发，2-3 周）
  → Mininet + Scapy（网络实验和抓包分析，按需）
  → smoltcp 或 lwIP（工业级协议栈阅读，按需）
  → ns-3（网络仿真研究，按需）
```

---

## 四、全栈学习路线

### 路线 A：体系结构为主线

```
Nand2Tetris（硬件基础）
  → 一生一芯 ysyx（自己设计 RISC-V CPU）
  → xv6-riscv（在 CPU 上跑 OS）
  → CS144（网络协议栈）
  → GPGPU-Sim / gem5（研究级微架构）
```

### 路线 B：操作系统为主线

```
xv6-riscv + MIT 6.S081（OS 基础）
  → rCore-Tutorial-v3（Rust OS）
  → CS144 + MIT Net Lab（网络栈 + 网卡驱动）
  → Linux 0.11 / Linux 内核（真实内核）
```

### 路线 C：网络为主线

```
Beej's Guide（Socket 编程基础）
  → CS144（TCP/IP 协议栈实现）
  → level-ip / tapip（用户态完整实现）
  → Mininet（SDN 实验）
  → Linux 内核 net/（真实网络栈）
```

### 三个方向各选一个的最小组合

| 方向 | 首选项目 | 理由 |
|------|---------|------|
| 体系结构 | **gem5** 或 **GPGPU-Sim** | 研究级模拟器，覆盖完整微架构 |
| 操作系统 | **xv6-riscv** | 全球数十所大学在用，教材和 Lab 最完善 |
| 计算机网络 | **Stanford CS144** | 课程设计精良，最终产出完整 TCP/IP 栈 |

---

## 五、C++ 语言学习

### 5.1 系统化教程与路线

| 项目 | Stars | 链接 | 说明 |
|------|-------|------|------|
| **awesome-modern-cpp-2025** | — | https://github.com/0voice/awesome-modern-cpp-2025 | 2025 最新一站式指南：18 周学习路线 + 可运行教程 + 面试题，从零基础到面试通关 |
| **小彭老师现代 C++ 大典** | — | https://142857.red/book/ | 中文社区权威指南，倒序教学（C++23 → C++98），含 CMake、STL 精讲、设计模式 |
| **C++ For Yourself** | — | https://github.com/cpp-for-yourself | 配套 YouTube 视频的结构化 C++ 课程 |
| **Learn Modern C++ Tutorial** | 130+ | https://github.com/cpp-tutor/learnmoderncpp-tutorial | 完整可运行的现代 C++ 示例程序集，配套 learnmoderncpp.com |
| **modern-cpp-tutorial** | — | https://github.com/changkun/modern-cpp-tutorial | 欧长坤《现代 C++ 教程》，覆盖 C++11/14/17/20，中英双语 |

### 5.2 实战练手项目（按难度递进）

#### 入门级

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **Tiny HTTPd** | C | 11k+ | https://github.com/EZLippi/Tinyhttpd | 500 行实现 HTTP 服务器，学网络编程入门 |
| **json** | C++ | 43k+ | https://github.com/nlohmann/json | 阅读源码学习现代 C++ 技巧（模板、SFINAE、迭代器） |
| **MyTinySTL** | C++ | 11k+ | https://github.com/Alinshans/MyTinySTL | 用 C++11 重写 STL，**学 STL 源码和模板元编程最佳项目** |

#### 中级

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **TinyRenderer** | C++ | 20k+ | https://github.com/ssloy/tinyrenderer | 500 行裸 C++ 实现软件渲染器，无需任何图形库，学 3D 图形学基础 |
| **Ray Tracing in One Weekend** | C++ | 9k+ | https://raytracing.github.io/ | 一个周末写一个光线追踪器，免费电子书系列，代码优雅 |
| **sylar** | C++ | 4k+ | https://github.com/sylar-yin/sylar | 高性能 C++ 服务器框架，含协程/IO调度/Hook/HTTP/RPC，学系统编程 |
| **muduo** | C++ | 14k+ | https://github.com/chenshuo/muduo | 陈硕的多线程网络库，学 Reactor 模式和高性能网络编程经典之作 |
| **leveldb** | C++ | 37k+ | https://github.com/google/leveldb | Google 出品的 KV 存储引擎，代码仅 ~15000 行，**公认最值得阅读的 C++ 项目** |

#### 高级

| 项目 | 语言 | Stars | 链接 | 说明 |
|------|------|-------|------|------|
| **CMU 15-445 BusTub** | C++ | — | https://github.com/cmu-db/bustub | CMU 数据库课程，用 C++ 实现存储引擎/B+树/查询执行/并发控制 |
| **Awesome C/C++ Projects** | C/C++ | — | https://github.com/0voice/Awesome_c-cpp_Projects | 500+ 高质量开源项目索引，覆盖网络框架/图形引擎/系统组件 |
| **LLVM** | C++ | 30k+ | https://github.com/llvm/llvm-project | 学编译器和现代 C++ 工程实践的终极项目 |

### 5.3 专项知识

#### 现代 C++ 特性（C++11/14/17/20/23）

| 资源 | 链接 | 说明 |
|------|------|------|
| **C++ Reference** | https://en.cppreference.com/ | C++ 标准库官方参考，查 API 必备 |
| **C++ Core Guidelines** | https://github.com/isocpp/CppCoreGuidelines | Bjarne Stroustrup 主导的 C++ 编码规范 |
| **Effective Modern C++** | — | Scott Meyers 的经典书籍，42 条现代 C++ 最佳实践 |

#### 模板与泛型编程

| 资源 | 链接 | 说明 |
|------|------|------|
| **MyTinySTL** | https://github.com/Alinshans/MyTinySTL | 通过重写 STL 学模板 |
| **C++ Templates: The Complete Guide** | — | 模板编程圣经（书籍） |

#### 并发与多线程

| 资源 | 链接 | 说明 |
|------|------|------|
| **C++ Concurrency in Action** | — | Anthony Williams 的多线程编程权威书籍 |
| **muduo** | https://github.com/chenshuo/muduo | 多线程网络库实战 |

#### 性能优化

| 资源 | 链接 | 说明 |
|------|------|------|
| **Google Benchmark** | https://github.com/google/benchmark | 微基准测试框架 |
| **perf / Valgrind / AddressSanitizer** | — | 性能分析和内存检测工具链 |

### 5.4 推荐学习路线

```
现代 C++ 语法（awesome-modern-cpp-2025 或小彭老师大典，3-4 周）
  → MyTinySTL（重写 STL，学模板和数据结构，2-3 周）
  → leveldb 源码阅读（工业级代码品味，2-3 周）
  → 选一个方向深入：
      ├── 图形方向 → TinyRenderer → Ray Tracing
      ├── 网络方向 → muduo → sylar
      ├── 数据库方向 → CMU 15-445 BusTub
      └── 系统方向 → GPGPU-Sim / gem5（你已经在这里了）
```

### 5.5 C++ 与本项目（GPGPU-Sim）的关联

GPGPU-Sim 本身就是学习 C++ 的极好素材，它大量使用了：

| C++ 特性 | 在 GPGPU-Sim 中的应用 |
|----------|----------------------|
| 继承与多态 | `cache_t` → `baseline_cache` → `l1_cache` / `l2_cache` / `tex_cache` |
| 模板 | `fifo_pipeline<T>` 延迟队列、`register_set` |
| STL 容器 | `std::map` / `std::vector` / `std::list` 遍布全项目 |
| 虚函数 | `scheduler_unit` 的多种调度策略（LRR/GTO/Two-Level） |
| 运算符重载 | `mem_fetch`、`warp_inst_t` |
| 指针与内存管理 | 手动管理的大量对象（模拟器出于性能考虑未使用智能指针） |

---

## 六、通用学习资源

| 资源 | 链接 | 说明 |
|------|------|------|
| **CS 自学指南 (csdiy.wiki)** | https://csdiy.wiki/ | 北大学生整理的计算机自学课程合集，中文，覆盖所有方向 |
| **build-your-own-x** | https://github.com/codecrafters-io/build-your-own-x | 从零实现各种系统的教程索引，300k+ Stars |
| **OSTEP** | https://pages.cs.wisc.edu/~remzi/OSTEP/ | 《Operating Systems: Three Easy Pieces》，免费在线 OS 教材 |
| **Computer Networking: A Top-Down Approach** | — | 计算机网络经典教材（自顶向下方法），配合 Wireshark 实验 |
