---
title: Nvidia GPU Architecture
hide: true
---

# CUDA
CUDA是一个硬件和软件架构，可以让NVIDIA的GPU来执行由C、C++、Fortran、OpenCL、DirectCompute和其他语言写的程序。

CUDA程序调用并行内核，内核在一组并行的线程中并行地执行。编译器组织以线程块和线程块网格的方式来组织线程，GPU在并行线程块的一个网格中初始化内核程序，线程块的每个线程执行一个内核程序的实例，并且有自己的线程ID。

线程块是一组并发执行的线程，这些线程可以通过屏障同步和共享内存相互协作，每个线程块有自己的ID。

一个网格是线程块数组，它们执行相同的内核，从全局内存中读取输入并向全局内存中写入输出，在独立的内核调用之间进行同步。

在CUDA编程模型中，每个线程有自己的私有内存空间。每个线程块有块共享内存，用于线程间通信、数据共享和结果共享。线程块网格在全局内存空间中共享结果。

CUDA与GPU的映射关系是
* 一个GPU执行一个或多个内核网格。
* 一个流式多处理器（SM）执行一个或多个线程块
* SM中的CUDA核和其他执行单元执行线程。

SM以32个线程为一组执行，称为一个warp。可以通过让wrap中的线程执行相同的代码路径并访问附近地址的内存来极大地提高性能。

# Fermi

Fermi主要关注以下方面的改进
* 改进了说精度浮点的性能
* 支持ECC
* 缓存层级
* 更多的共享内存
* 更快的上下文切换
* 更快的原子操作

Fermi架构的主要亮点如下
* 第三代流式多处理器（SM）
  * 每个流式多处理器有32个CUDA，是GT200的四倍
  * 双精度浮点数的性能峰值时GT200的八倍
  * Dual Wrap调度器同时调度，派发来自两个独立的warp中的指令
  * 配置共享内存和L1缓存的64MB RAM
* 第二代并行线程执行ISA
  * 完整C++支持的统一地址空间
  * 为OpenCL和DirectCompute进行了优化
  * 完整的IEEE 754-2008 32位和64为精度
  * 带64位扩展的完整32位整型路径
  * 支持过渡到64位寻址的内存访问指令
  * 通过预测改进了性能
* 改进的内存子系统
  * 带有可配置的L1缓存和统一L2缓存的NVIDIA并行数据缓存层级
  * 带有ECC内存支持（第一个）
  * 极大地改进了原子内存操作性能
* NVIDIA GigaThread Engine
  * 10倍的上下文切换速度
  * 并发内核执行
  * 乱序线程阻塞执行
  * 双重叠内存传输引擎

实现了所有DirectX 11的硬件特征，包括曲面细分和DirectCompute（DirectCompute是微软的一个API接口，允许使用GPU进行通用计算）。

设计目标

1. 卓越的游戏性能：翻新了几何管线，加倍了CUDA核，加倍了ROP，提高了填充率。
2. 一流的图像质量：实现了新的32CSAA
3. 电影级的几何真实感：全新的分布式几何处理架构，使用多个PolyMorph引擎实现。并行几何处理能力
4. 革命性的游戏计算架构：图形和物理之间更快的上下文切换，并发计算内核执行和增强的缓存架构，增加原子操作性能。

## 架构

![image-20211123094226769](/Users/shellingford/Library/Application Support/typora-user-images/image-20211123094226769.png)

Fermi架构包含了四个GPCs(Graphics Processing Clusters)，16个SMs(SMs)和六个内存控制器。

GPU通过Host Interface读取cpu命令，GigaThread Engine从系统内存中获取指定的数据并拷贝的framebuffer中，然后创建并分发thread blocks到各个SM是中，每个SMs轮流调度warp到CUDA和和其他执行单元。当出现工作增加的时候，如曲面积分和光栅化之后，GigaThread Engine也会将工作重新分配个SM。

Fermi共有512个CUDA核，每个SMs有32个核，共16个SMs，每个SMs是高度并行的，支持最大48个warp。L2缓存是统一的。

Fermi有48个ROP单元，被分为6组，每组8个，每组有一个内存控制器，内存控制器、ROP和L2缓存是紧密耦合的。

### GPC架构

![image-20211123101027805](/Users/shellingford/Library/Application Support/typora-user-images/image-20211123101027805.png)

GPC包含一个Raster Engine和最多四个SM。

# Kepler

28nm纳米工艺，最佳的性能/功耗比，提供最佳的细分性能，更高水平的几何复杂性、物理模拟、立体3D处理和高级的抗锯齿效果。

Kepler设计的重点是提升性能功耗比，一个主要的改进是SMX（新的Streaming Multiprocessor）以图形时钟运行而不是以两倍的图形时钟运行。

## 架构

Kepler由GPCs、Streaming Multiprocessors和内存控制器组成。

下图是GTX 680的架构，有四个GPCs，八个下一点SMX和四个内存控制器。

![image-20211123082140331](/Users/shellingford/Library/Application Support/typora-user-images/image-20211123082140331.png)

每个GPC有专用的光栅引擎和两个SMX单元，共有1546个CUDA核。

完全修改了内存子系统，有更高的内存时钟（6008MHz），每个内存控制器有一个128BK的L2缓存和8个ROP（Render output unit）单元，每个ROP单元可以处理一个颜色样本