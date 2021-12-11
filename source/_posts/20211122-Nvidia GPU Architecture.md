---
hide: true
---



# Fermi

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