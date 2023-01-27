# Application Informed Kernel Synchronization Primitives

作者：Sujin Park, Georgia Tech; Diyu Zhou and Yuchen Qian, EPFL; Irina Calciu, Graft; Taesoo Kim, Georgia Tech; Sanidhya Kashyap, EPFL

{% hint style="info" %}
- 核心问题：1. 内核同步原语并不对程序开发者开放，程序开发者无法控制内核同步原语行为。2.内核锁没有很好的适应异构硬件。
- 方案：提出workload-specific和hardware-aware的内核锁解决病态使用内核锁（pathological usage），同时提供动态profiling技术。
- 设计：提出一种内核锁的修改方法，可以不用重新编译或者重启内核。
{% endhint %}

## 提出问题
1. 安全性问题：引入过多的锁之后是否会有安全性问题？
2. 内核锁不修改如何实现？

