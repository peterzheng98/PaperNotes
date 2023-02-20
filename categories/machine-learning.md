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