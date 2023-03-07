# 🥇 Machine Learning

## \[NSDI 2022\] MLaaS in the Wild: Workload Analysis and Scheduling in Large-Scale Heterogeneous GPU Clusters
作者：

## \[ASPLOS 2023\] ElasticFlow: An Elastic Serverless Training Platform for Distributed Deep Learning
作者：Diandian Gu, Yihao Zhao, Yinmin Zhong, Yifan Xiong, Zhenhua Han, Peng Cheng, Fan Yang, Gang Huang, Xin Jin, Xuanzhe Liu（北大和MSR）

文章认为现在云上基于服务器的分配并不是最适合DL的设计，需要引入一个更高的抽象，用户无需知晓下层系统的更多设计。作者认为DL任务对DDL的感知非常重要。

Key takeaways：
- 成果：提供了一个Serverless Training的平台，用户可以无需指定GPU数量，只需要指定任务的DDL和超参数（包括模型本身，模型Batch Size，终止条件，训练的DDL，数据集，optimizer等等）。
- 观察：任务并不随着GPU数量的上升而吞吐上升，不同任务可扩展性（GPU变多的Throughput）不同。

方法：
- 定义使用的资源是GPU的数量和运行时间的乘积，如果能够满足任务的ddl才分发。
- 拓扑感知的任务放置方法：把所有的GPU以树的模式结合，在树上找最合适的节点。
- 减少Fragment：使用Buddy Allocation，Worker数量为2的倍数，确保GPU不出现碎片。

核心挑战：Training并不Scale，受到Work放置的位置影响

效果：任务的DDL达成率很高，没有提训练的performance

不同：不涉及Inter-GPU Sharing、Batch仍然需要全局指定

## \[OSDI 2022\] Synergy: Resource Sensitive DNN Scheduling in Multi-Tenant Clusters
作者：Jayashree Mohan, Amar Phanishayee, Janardhan Kulkarni, Vijay Chidambaram
单位：MSR，UT at Austin，VMWare Research

Key Takeaways：
- 问题：机器学习训练任务对CPU和内存是敏感的，但是目前云上按照一个GPU配比一定的CPU和Memory实现。
- 设计了一套Optimistic Profiling机制推断任务对CPU和内存的敏感程度。
- 设计了一套workload-aware的分配策略，得到了性能的提升。

方法：
- Optimistic Profiling：利用Throughput和memory allocation分配之间的可预测性，利用内存分配的行为预测CPU的使用数量。
- Workload-aware分配：将执行过程的workload需求转换为一个demand vector，求解线性最优。

效果：提高了程序的执行时间（1.5x-2x）。

## \[Arxiv 2302.11358\]Faabric: Fine-Grained Distribution of Scientific Workloads in the Cloud
作者：Simon Shillaker, Carlos Segarra, Eleftheria Mappoura, Mayeul Fournial, Lluis Vilanova, Peter Pietzuch
单位：Imperial College London

Key takeaways:
- 科学计算在专用的VM池子里面的分配，存在严重的资源碎片问题。无法利用未使用的资源。
- 现有的Serverless编程范式并不支持函数间通信，提出了Faabric，使用线程和进程作为抽象的单位。

问题：Serverless场景下使用线程进程是否有些奇怪？

## \[IEEE Micro\] Failure Tolerant Training with Persistent Memory Disaggregation over CXL
作者：Miryeong Kwon, Junhyeok Jang, Hanjin Choi, Sangwon Lee, Myoungsoo Jung
单位：KAIST

Key takeaways:
- 提出了利用CXL实现使用Disaggreated Memory同时达到低开销Fault Tolerance，应用于超大模型的训练。
具体设计：
(1) 将持久内存（PMEM）和GPU整合到缓存一致的域（Type-2 CXL）
(2) 分析了推荐系统（Recommendation System）的特点并且将checkpoint从关键路径上分离。具体而言是，Batch-aware Checkpoint，在CXL-GPU、CXL-MEM上同时进行MLP的操作
(3) 给出了一种checkpoint的技术，可以relax在batch之间更新参数和嵌入向量
奇怪的观察与挑战：
- 观察1：工业使用的RM（推荐系统）会占用T级别甚至P级别的内存（引用了Facebook的文章，但是原文里没有提到P级别的内存）
- 观察/挑战2：推荐系统模型需要容错（精度不下降），因为其训练时间长，错误开销大。同时snapshot的时候不能成为性能瓶颈。
效果：与其他PMEM工作相比，训练上提高5.2x速度，节约76%能耗。（Workload的属性其实没有利用好）

## \[ISCA 2022\] Software-Hardware Co-design for Fast and Scalable Training of Deep Learning Recommendation Models
Tag：Neo、NeoEX
作者：Dheevatsa Mudigere, Yuchen Hao, Jianyu Huang, Zhihao Jia, Andrew Tulloch, Srinivas Sridharan, Xing Liu, Mustafa Ozdal, Jade Nie, Jongsoo Park, Liang Luo, Jie Amy Yang, Leon Gao, Dmytro Ivchenko, Aarti Basant, Yuxi Hu, Jiyan Yang, Ehsan K. Ardestani, Xiaodong Wang, Rakesh Komuravelli, Ching-Hsiang Chu, Serhat Yilmaz, Huayu Li, Jiyuan Qian, Zhuobo Feng, Yinbin Ma, Junjie Yang, Ellie Wen, Hong Li, Lin Yang, Chonglin Sun, Whitney Zhao, Dimitry Melts, Krishna Dhulipala, KR Kishore, Tyler Graf, Assaf Eisenman, Kiran Kumar Matam, Adi Gangidi, Guoqiang Jerry Chen, Manoj Krishnan, Avinash Nayak, Krishnakumar Nair, Bharath Muthiah, Mahmoud khorashadi, Pallab Bhattacharya, Petr Lapukhov, Maxim Naumov, Ajit Mathews, Lin Qiao, Mikhail Smelyanskiy, Bill Jia, Vijay Rao
单位：Meta、CMU

问题：推荐系统存在大量Data-intensive的embedding operators，这些operators对算力的占用相对比较少。目前的软件和硬件不能很好的适配这一趋势。（就是模型非常吃数据）
- 软件问题：现在深度学习的并行可以分为数据并行、模型并行和pipeline。但是这些并行主要优化compute-intensive，而不是data-intensive。
- 硬件问题：硬件主要加速节点间的中心化数据同步（例如参数服务）和AllReduce（NCCL），但是DLRM实际上存在更复杂的节点间通信范式。

方法：
- 提出4个维度的并行：Table-wise、Row-wise、Column-wise和data的parallelism
- 高性能的Embedding计算：(1)混合的CUDA内核融合技术（融合Embedding算子和Embedding计算以及他们的更新）(2)软件管理的缓存机制和内存压缩机制。
- 硬件平台设计：专用RDMA节点，主要是考虑网络通信

结果：在128张A100的GPU + 16个网络节点的组合提高了40倍训练速度。

## \[ACM Measure 23\] DaeMon: Architectural Support for Efficient Data Movement in Fully Disaggregated Systems
Tag：DaeMon
作者： Christina Giannoula, Kailong Huang, Jonathan Tang, Nectarios Koziris, Georgios Goumas, Zeshan Chishti, Nandita Vijaykumar
单位：University of Toronto

Key Takeaways:
- 在Disaggreated System上，远程内存的搬运开销非常大。
- 设计了Daemon利用专用引擎实现了对上层软件透明的计算，实现了自适应的粒度配置（基于带宽切分、链路压缩和数据搬运解耦）
- 效果：与页粒度的搬运相比获得了2.39x和3.06x的数据移动加速。

关键技术：
- 把数据迁移offload给专用硬件引擎处理。
- 优先考虑缓存行级别的数据移动，利用硬件链接压缩（把多个缓存行的访问压缩成一个page）来减少网络带宽消耗并减轻排队延迟。



