---
hide: true
---



# 引言

GPU是现代计算机中必不可少的一部分，最初的GPU是用于进行图形计算，现在的GPU已经逐渐演变一个更加通用的计算单元，常常被用来加速计算神经网络等计算密集的工作。本文将回顾GPU的发展历史，了解GPU的演变过程，以及未来可能的演变方向。

# 早期

在20世纪八十年代左右，那时的GPU仅仅负责渲染的部分，一些计算依然是在的CPU上进行的。最初，只有绘制线框图的功能，后面逐渐增加了光照、绘制填充多边形、深度缓冲、颜色混合等功能。

NEC的μPD7220是与1981年12月发布，是第一个将PC图形显示处理器作为单个大规模集成电路芯片实现的产品，使低成本、高性能的视频图形卡的设计成为可能，是20世纪80年代最著名的图形芯片之一，它能够将直线、圆、弧和字符图形绘制到位映射显示器上，支持高达1024x1024的分辨率，并为新兴的PC图形市场奠定了基础。µPD7220建立了一个易于使用的低级指令集，应用程序开发人员可以很容易地将其嵌入到他们的程序中，从而加快绘图时间。[1]Intel的第一个图形芯片82720使用了这个芯片的设计授权[2]。

1987年，IBM随个人计算机退出的IBM8514在电子硬件中显示固定功能的2D基元，该产品创造了一个固定功能的PC图形加速器市场。

1991年5月，ATI发布的Mash 8，它基本上是IBM8541的克隆，并加上了一些扩展，如水晶字体，支持8位颜色模式，512 KB或1MB内存，Mach8没有一个集成的VGA核心，需要一个单独的VGA卡。后序产品Mash 32具有了集成的VGA核心[3]。

# 3D时代

1995年5月，NVIDIA发布了第一个能够实现3D渲染、视频加速和集成GUI加速的商用图形处理器。它试图取代2D图形卡、兼容Sound Blaster的音频系统和15针操纵杆端口，但是并不是非常成功。

同年9月，ATI发布了自己的第一个3D加速芯片3D Rage（也被称为Mach64 GT），这个芯片是将Mach64的2D核心与3D功能相结合，该芯片配置了2MB的内存。

在1996年11月，3dfx的Voodoo横空出世，虽然该芯片是一个纯3D加速芯片，需要配合2D的显卡使用，但是，它的性能远远领先现有的其他显卡。该芯片的出现几乎在一夜之间彻底改变了个人电脑图形领域，并使许多其他设计过时，其中还包括了大量的纯2D图形生产商。据估计，在Voodoo统治的全盛时期，3Dfx占据了80-85%的3D加速器市场。

次年，3dfx为了解决Voodoo需要搭配另一块2D显卡的问题推出了Voodoo Rush，它是一块集合了2D显示输出和3D加速功能的显卡，2D芯片采用的是Alliance生产的AT3D，然而，由于性能和兼容性都欠佳，因此，销量非常不好。[4]

同年，NVIDIA和ATI分别推出了自己的新产生RIVI 128和3D Rage Pro。

RIVI 128整合了传统的2D加速和3D加速，同时增加了Direct3D的兼容性，RIVA 128是第一批可以与Voodoo Graphics媲美的2D/3D组合卡之一。RIVA 128的2D功能在当时被认为是令人印象深刻的，甚至在质量和性能上都能与高端的纯2D显卡竞争。

而ATI的Rage Pro改进了Rage II的透视校正，纹理能力和三线过滤性能，增加了浮点单元，以减少对CPU的依赖。但是，它的性能在Nvidia的RIVA 128和3dfx的Voodoo加速器的范围内，没能超过竞争对手，另外，早期缺乏对于OpenGL的支持，使得销量不佳。

1998年，3dfx推出了Voodoo2，它也是一个纯3D加速的性能，性能大幅度地由于Voodoo，并且还支持SLI技术，可以将两块Voodoo2的显卡连接起来，获得更高的性能，另外，Voodoo2是第一个使用多纹理的显卡，有两个纹理集成电路，是的纹理采样效率提高了一倍。

之后，3dfx推出了2D+3D的芯片Voodoo Banshee，虽然兼容性比Voodoo Rush要好，但是性能不及Voodoo2。

对于Voodoo2的回应，NVIDIA在1998年6月发布了RIVA TNT，它增加了第二条并行像素流水线，使像素填充率和渲染速度提高了一倍，还增加了16MB的SDR内存。在设计上，RIVA TNT的性能理论上会由于Voodoo2，但是由于发热问题，无法以设计的频率运行，实际的运行频率为90MHz，Voodoo2在性能上依然占据了领先地位。另外，RIVA TNT并没有支持3dfx的Glide API，当时，Glide API被认为是最好的3D游戏API，因此，NVIDIA的市场份额并没有太大的增长。

但是，它的AGP 2x接口允许在1600 x 1200的条件下进行游戏，并以24位Z型缓冲器（图像深度表示）进行32位色彩渲染，这比Voodoo 2的16位色彩支持和16位Z-buffer有巨大的改进，因此，RIVA TNT赢得了许多关注。

1999年3月，3dfx发布了它们第一个真正全新设计的2D+3D芯片Voodoo3，虽然改了AGP 2x接口，但不支持AGP 2x的绝大多数特性，性能与同期产品相近，并没有像之前的产品一样令人眼前一亮，从此，3dfx由盛转衰。

与此相反，NVIDIA和ATI越来越强盛。1999年3月，RIVA TNT2发布，它支持2048x2048纹理，32位Z-缓冲器，AGP 4X，具有高达32MB的VRAM， 时钟速度得到了提高（从90MHz提高到150+MHz），性能大幅提高。

ATI的Rage系列也在渐进式发展，Rage 128 Pro性能与RIVA TNT2大致相当。

这一阶段的显卡开始增加了纹理相关的功能，并且显卡的渲染速度也越来越快。

# 固定渲染管线

1999年9月，NVIDIA发布了GeForce 256，是世界上第一个被称为GPU的图形芯片，使GPU这个词流行起来，当时NVIDIA将这个词定义为，“一个集成了变换、照明、三角形设置/剪辑和渲染引擎的单芯片处理器，能够每秒处理至少1000万个多边形。”

GeForce 256可以进行大量浮点密集型的计算，并将变换和照明（T&L）计算从CPU移动到了GPU，将CPU从繁重的计算中解放出了，在这之前，这些操作都是由CPU执行的。但是，在那个时代，当CPU性能足够高时，GeForce 256的这个特点反而会成为计算瓶颈，成为渲染性能的障碍，这个在现在看起来意义重大的设计在当时可能成为一个缺点。另外，GeForce 256还是第一个完全符合Direct3D 7标准的3D加速器。

2000年4月，ATI也发不了新系列Radeon系列的第一款GPU，Radeon DDR，它是R100的第一个版本，引入了HyperZ，是一个对z-buffer优化的技术，整体渲染效率提高了20%，同时支持Direct3D 7和OpenGL 1.3，也支持硬件T&L。

不久之后，NVIDIA发布了GeForce 2作为回应。GeForce 2架构与GeForce 256架构类似，但GeForce 2采用了180纳米工艺，有更高的时钟速度，并且还正式引入了NSR（NVIDIA Shading Rasterizer），NSR是一种原始类型的可编程像素管道，与像素着色器有些类似。

3dfx虽然也发布了自己的新产品Voodoo4/5，但是，性能和价格都与NVIDIA和ATI的产品有差距，没能成功帮助3dfx起死回生，最终，3dfx在年底被NVIDIA收购。

此时，GPU的市场已经是ATI和NVIDIA的天下了。

这个阶段的之前的GPU是固定功能流水线的架构，在这个阶段，人们通过配置来决定gpu如何进行渲染。

# 可编程渲染管线

固定渲染管线是非常不灵活的，随着GPU的功能越来越多，所需要的配置也越来越多。而且，固定渲染管线能够实现的功能取决于硬件的支持，无法灵活的实现想要的效果。[5]

因此，在2001年2月，NVIDIA发布了GeForce 3，这一系列的GPU首次引入了可编程的像素着色器和顶点着色器，允许开发人员灵活的进行渲染。此外，该系列显卡增加了光速内存架构LMA（NVIDIA的HyperZ），可以更好的管理内存；GeForce 3还对抗锯齿功能进行了改善，增加了MSAA和Quincunx AA；纹理单元也得到了升级，支持8次各向异性过滤。

2001年10月，ATI发布了Radeon 200作为回应，这一系列是ATI首款具有可编程像素和顶点处理器的GPU。此外，还包含了HyperZ II（HyperZ的升级版本），可以节省内存带宽，减少广度绘制。ATI还首次加入了曲面细分硬件加速模块，Truform，不过由于很少使用，ATI在未来的架构中取消了该模块。但是，ATI发布的驱动程序再次出了问题，对R8500的测评结果产生了极大的影响。

2002年2月，NVIDIA发布了GeForce 4，使用了150nm制造工艺，有更高的时钟速率，升级到LMA II，是当时显卡性能的巅峰。

2002年8月，ATI推出了R300 GPU，在性能和架构上都有了很大的改进。它是第一个支持着色器模型2.0、顶点着色器2.0和像素着色器2.0的架构，是一个倒装芯片GPU封装，可以更好的冷却芯片；使用了HyperZ的最新改进版本HyperZ III；第一个真正利用256位内存总线的芯片。

NVIDIA在2003年推出了对应的产品GeForce系列的第一个产品FX 5800，在抗锯齿方法虽然得到了加强，但是性能还是逊色于ATI的Radeon 9700，并且还有非常大的噪声。在三个半月后，NVIDIA发布了FX 9500，Detonator FX 驱动器大大提高了 AA 和 AF，击败了Radeon 9800 Pro，称为速度最快的显卡。

在NVIDIA的GeForce 6系列中，增加了源于3dfx的SLI技术，使得两个显卡可以共同工作。

ATI也在次年公布了自己的多卡互连技术CrossFire，除了提供和SLI一样的AFR和SFR外，还提供了一种称为SuperTiling的渲染技术。CrossFire也和SLI一样，存在驱动相关的问题。

ATI和NVIDIA的竞争一直持续到了2006年，在ATI被AMD收购后，AMD称为了NVIDIA新的竞争对手。

这个阶段的GPU包含了两个可编程的处理阶段，顶点处理器执行顶点着色程序和像素处理器执行像素着色程序。由于可编程过程的发展，两个处理器在功能上逐渐变得相似起来。另外，由于顶点和像素的对应关系不确定，在设计中，顶点处理器和像素处理器的数量关系不好确定，因此，推动了统一处理架构的诞生[6]。

# 通用计算时代

在2005年，XBox 360中使用了统一着色架构的GPU Xenos，该芯片由ATI设计[7] [8]。随后，NVIDIA发布了GeeForce 8系列也采用了使用统一着色架构的TESLA架构。

Tesla架构[6]如下图所示，是基于一个可扩展的处理器阵列。

![image-20211201142248431](/Users/shellingford/Library/Application Support/typora-user-images/image-20211201142248431.png)

GeForce 8800 GPU有8个纹理处理器集群（TPCs），每个TPC是一个处理单元，每个TPC有两个流式多处理器（SMs），每个SMs有8个流式处理器（SP）核。每个SM都可以执行顶点处理程序和像素处理程序。

Tesla的SM使用了新的处理器架构，称为单指令多线程（SIMT），以32个并行线程为一组创建、管理、调度和执行线程，称为wrap。一个wrap中的线程使用不同的数据执行相同的程序。

从架构图中可以看作，GPU的核心数量远远高于CPU，并且GPU的架构针对并行计算进行了优化，计算速度远远高于CPU。GPU的核心也变得更加通用，配合上NVIDIA发布的CUDA，使得GPU不仅仅可以进行图形相关的计算，还可以执行一些更加通用的计算。[9]中展现了4个使用GPU计算的例子。

2010年，NVIDIA发布了Fermi架构[10]，如下图所示，Fermi架构是第一个计算GPU并对Tesla架构进行了一些改进，提高了双精度浮点计算的性能、增加了ECC支持、改进了缓存架构、增加了更多的共享内存、加速了上下问切换和原子操作。

![image-20211203102215280](/Users/shellingford/Library/Application Support/typora-user-images/image-20211203102215280.png)

Fermi架构中使用CUDA核替代了Tesla核中的SP，每个CUDA核可以执行整数或浮点数指令。

2012年，NVIDIA发布了新一代的GPU架构，Kepler[11]。Kepler改进了Fermi架构的功耗问题，设计时优先考虑了性能功耗比。Kepler采用了新的流式多处理器，称为SMX，与Fermi相比，每个SMX有更多的CUDA核，更高的内存带宽。相较于前一代，性能大幅度提升。

2014年，NVIDIA再次发布了新的架构，Maxwell[12]，是Kepler的升级版本，有更多的核，更加强大的性能。

2016年，NVIDIA发布Pascal架构[13]，与Kepler和Maxwell架构相比，每个SM只有64个CUDA核，但是SM的数量大大增加了，每个SM的寄存器堆的大小不变，意味着每个CUDA核可以使用的寄存器堆更多了，总的寄存器数量大大增加了。

对于许多高性能计算应用，如线性代数、数值模拟和量子化学等，双精度浮点的计算是非常重要的，因此，每个SM中除了32位浮点计算的CUDA核外，还增加了64位浮点计算单位DP Unit，增加了高精度浮点的计算，DP Unit与CUDA核的比例是1:2，

对于神经网络的计算而言，浮点计算不需要过高的精度，使用16位浮点数可以减少内存使用，从而训练核不是更大的网络，因此，32位的CUDA和可以处理16位的浮点数据，并且吞吐量是32位浮点数的两倍。

另一个重要的改进是增加了NVLink，该接口提供了GPU到GPU的数据传输，双向带宽为160Gb/s，是16倍PCIe Gen 3带宽的5倍，解决了多GPU计算系统中PCIe带宽瓶颈的问题。

Pascal架构的设计考虑到了科学计算、神经网络方面的计算需求，进行了针对性的设计。

Volta架构[15]的第一个产品于2017年发布，Volta的SM针对深度学习进行了优化，在Pascal中每个SM被划分为两个处理块，每个处理块有32个32浮点计算CUDA核，16个64位浮点计算DP unit。而在Volta架构中，每个SM被划分为四个处理块，每个处理块有16个32浮点CUDA核，16个32位整形CUDA核，8个64位浮点计算核，两个Tensor核，一个新增的L0指令缓存。

其中，每个Tensor核是在4 $\times$ 4矩阵上进行运算并执行$D = A \times B + C$操作。A和B都是16位浮点数，C和D是16位或32位浮点数。Tensor核是专门为深度学习而设计的，用于加快神经网络中的矩阵运算，对于单精度的矩阵乘法应用，Tensor核的性能是Pascal的1.8倍，对于16位浮点和32位浮点的混合精度的矩阵操作，Tensor核的性能比Pascal提高了9被以上。

此外，NVLINK也得到了升级，带宽更高，连接数量更多。

Pascal中的CUDA核被拆成了FP32 CUDA核和INT32 CUDA核，因此，可以同时执行整数运算和单精度浮点数运算。

2018年的Turing架构[16]除了对SM的性能参数进行加强外，最重要的突破是在GPU中增加了RT核，这是专门用于执行光线追踪操作的加速器单元。

此时的渲染管线也发生了变化，由于目前RT核的计算能力还不足以完全实现实时光线追踪，因此，现在的渲染管线是一种混合的渲染关系，具体如下图，由光栅化渲染关系完成整个场景，由RT核来在特定区域完成次级光线的计算，来生成高质量的反射、折射和阴影。

![image-20211203151709954](/Users/shellingford/Library/Application Support/typora-user-images/image-20211203151709954.png)

在这种混合渲染管线中，光追管线和光栅化关系是同时执行的。

NVIDIA最新的Ampere架构于2020年发布，新 "多实例GPU"（MIG）虚拟化和 GPU分区功能，这部分是为云服务提供的优化，MIG允许将GPU安全地划分为多达七个独立的GPU实例，同时为多个用户提供独立的GPU资源。Ampere还针对深度学习进行了进一步都优化，现在Apmere可以支持所有深度学习的数据类型，并增加了对稀疏矩阵的优化。

自从统一着色架构开始，GPU已经不仅仅是为图形做运算的硬件了，由于GPU中拥有许多计算单元，同时是为并行计算设计的，天然比较适合执行计算密集和并行度较高的程序。从NVIDIA近年来的架构演变可以知道，NVIDIA正在努力将GPU打造成一个通用的计算平台而不仅仅用于图形计算。

# 展望

尽管，光栅图形计算和深度学习计算具有相似之处，NVIDIA在针对深度学习计算进行优化的时候也使得GPU图形计算能力极大提高，但是，个人认为，这种情况无法持续很久。因为，图形学的未来必定要实现实时的光线追踪，目前，NVIDIA仅在Turing之后的架构中增加了支持光追的RT核，而RT核对与通用计算帮助不大。正如，最初图形计算是在CPU进行的，为了提升计算性能解放CPU而将图形相关计算逐渐的移到了GPU上。为了得到更高的性能，势必要针对使用场景对硬件进行优化。Google开发了专门用于加速机器学习的TPU。AMD也采用了分离的架构，在通用计算平台上使用CDNA架构，而在图形显卡上使用RDNA架构。

# 参考文献

[1] https://www.computer.org/publications/tech-news/chasing-pixels/famous-graphics-chips

[2] https://en.wikipedia.org/wiki/NEC_%C2%B5PD7220

[3] https://en.wikipedia.org/wiki/ATI_Mach_series

[4] https://zh.wikipedia.org/wiki/3dfx_Interactive

[5] https://cseweb.ucsd.edu/~ravir/6160-fall04/papers/p149-lindholm.pdf

[6] https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=4523358

[7] https://en.wikipedia.org/wiki/Xenos_(graphics_chip)

[8] https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=1624324

[9] https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=4490127

[10] https://www.nvidia.com/content/PDF/fermi_white_papers/NVIDIA_Fermi_Compute_Architecture_Whitepaper.pdf

[11] https://www.nvidia.com/content/PDF/product-specifications/GeForce_GTX_680_Whitepaper_FINAL.pdf

[12] https://www.microway.com/download/whitepaper/NVIDIA_Maxwell_GM204_Architecture_Whitepaper.pdf

[13] https://images.nvidia.cn/content/pdf/tesla/whitepaper/pascal-architecture-whitepaper.pdf

[14] http://www.mathcs.emory.edu/~cheung/Courses/355/Syllabus/94-CUDA/Docs/gpu-hist-paper.pdf

[15] https://images.nvidia.cn/content/volta-architecture/pdf/volta-architecture-whitepaper.pdf

[16] https://images.nvidia.cn/aem-dam/en-zz/Solutions/design-visualization/technologies/turing-architecture/NVIDIA-Turing-Architecture-Whitepaper.pdf