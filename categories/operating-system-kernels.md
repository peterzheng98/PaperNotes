# 📦 Operating System Kernels

## [\[OSDI 2022\] Application Informed Kernel Synchronization Primitives](../conference/osdi/park.md)
作者：Sujin Park, Georgia Tech; Diyu Zhou and Yuchen Qian, EPFL; Irina Calciu, Graft; Taesoo Kim, Georgia Tech; Sanidhya Kashyap, EPFL

## \[Oakland 2023\] μSWITCH: Fast Kernel Context Isolation with Implicit Context Switches
作者：Dinglan Peng∗, Congyu Liu∗, Tapti Palit∗, Pedro Fonseca∗, Anjo Vahldiek-Oberwagner†, Mona Vij†；∗Purdue University，†Intel Labs

### 描述
- 隔离程序是一种保护敏感数据的方法，先前工作主要聚焦于提出新的内核抽象、CPU的新指令集（MPK）、基于软件的错误隔离（WebAssembly）。这些工作忽略了高上下文切换或共享内核资源场景下的性能开销。
- 本文主要关注通过细粒度domain设计的**内核资源隔离**，该隔离仍然可以保证安全性。
- 减少上下文切换的方法：允许用户态的切换而非trap进入内核（使用MPK切换权限），真正内核资源的切换由下一次syscall的时候完成。

### Key Points: 将Context Switch转化为用户态的切换
- 由于原来Context隔离依赖于进程抽象，必须上下文切换。现在可以通过一个uContext将多个Context运行在一个process中实现intra-context切换。（Fig 2）
- 额外增加一个Context Descriptor用于表达Context，并且直接通过memory sharing的方法在用户态和内核共享。每个Context Descriptor都代表了一组Kernel Resources。
- 真正进入内核的时候检查内核的Context Descriptor是否与用户态一致（一致说明用户态没有切换过，不一致说明切换过，Fig 4）。

### 核心
- **减少上下文切换如何实现？**Context之间的切换不进入内核，只有真正的syscall时才进入内核。
- **核心Insight**：1. 对内核资源的访问一定需要一次syscall。2. 对内核资源的切换在每次context切换的时候不是必须的，只需要在真正访问到内核资源的时候才需要切换/检查。（纤程思想？）

### 威胁模型
- 内核安全，App不安全
- 内核中不包括直接泄露信息给App的代码，不违反µSwitch的安全保证。
- 不考虑侧信道

### 思考
- 主要方法似乎和协程接近，如何保护内存？保护CPU？

## \[APSys 2022\] Towards Isolated Execution at the Machine Level
作者：Shu Anzai, The University of Tokyo; Masanori Misono, Technical University of Munich; Ryo Nakamura, Yohei Kuga, and Takahiro Shinagawa, The University of Tokyo

### 描述
- 隔离执行是缓解大型软件中漏洞的方案，先前工作一大类是CPU层的保护（进程沙盒 process sandbox、虚拟化、可信执行环境）。这些保护方法本身也可能存在漏洞，并且需要避免硬件层面的漏洞。
- 大家常见关注的漏洞：Memory Corruption、Transient Execution Attacks，这些攻击大多虽然有Type-Safe Language和Formal Verification方法，但是无法适用于legacy的程序。
- 本文采用类似微内核的方法，将机器物理隔离并且使用低开销的通信手段实现通信。

### Key Points: 可以使用Machine Level隔离，结合RDMA实现类似RPC的syscall转发，提供非常低开销的隔离。
- Host Machine正常运行，Isolated Machine运行TypeSafe的ThinOS。
- Isolated Machine上有一个Buffer保存Syscall的参数，Host Machine主动拿Buffer里的数据进行RPC。

### 威胁模型
- App有漏洞，OS可信
- App共享该机器上Host中的CPU缓存和物理内存等硬件资源，允许 Transist Execution 或 DRAM 干扰等硬件漏洞进行攻击。
- 不考虑：完全在内核的漏洞（不涉及用户态）、无效参数攻击

### 核心
- **如何达到最小TCB？** 采用Machine Level的隔离，通过不共享计算资源的物理隔离机器来实现强隔离。运行隔离环境的系统是Type-Safe的Thin OS。
- **物理隔离的机器如何保证基本功能？** 采用基于FPGA的专用硬件实现通信（允许从远程机器到目标机器上的任何物理地址的单向 RDMA）
- **性能？** 开销低，主要原因是RDMA。

### 思考
- 该方法主要可以缓解目前CPU和Memory的侧信道问题，RDMA是否会引入新的侧信道问题？
- 隔离角度上，CPU和Memory Corruption没有提供任何保证。
