---
hide: true
---



# [CSF][2]

针对多种Spectre变体的微代码级的防御。

## 应对目标

主要是边界检查绕过攻击，在可接受性能水平上缓解光散场。

也包括针对其他变种的缓解措施。

主要关注适用数据缓存作为泄露信息的旁路攻击变体，可以扩展到其他旁路信道。

## 基本思想

利用动态更改解码的指令流的能力，在动态条件表明需要是来无缝地注入新的栅栏等微操作，使得可以在对功能影响较小的情况下低于攻击，并且引入了新的三种栅栏实现。

## 详细设计

1. 微代码定制机制，允许处理器在动态指令流中插入栅栏以减轻投机执行的不良副作用。
2. 解码器级信息流跟踪（DLIFT）框架，识别潜在的不安全执行模式，以触发微代码定制
3. 确保分支预测器核返回地址栈的错误



序列话指令会完全覆盖投机执行，解码时，停止提取任何后续指令，直到此指令前的所有指令退出。会造成长时间延迟和大量吞吐量损失。

三个新增的栅栏

LSQ-LFENCE和LSQ-MFNENCE在加载/存储对列中强制执行宽松栅栏。当LSQ-FlFENCE有效时，不允许从加载/存储对列发出任何后续加载指令，防止缓存状态被错误投机路径上的加载指令更改。

LSQ-MFENCE不允许从加载存储对列中发出任何后续内存指令，直到栅栏提交。

CFENCE是一个宽松的栅栏，它允许前面的指令继续进行，存储指令不受CGENCE的影响，将任何后续加载标记为非修改加载并允许通过栅栏，但被限制修改缓存状态。缓存命中的load指令可以读取缓存内容，但无法修改LRU和其他元数据位，缓存未明绿色的load被标记为不可缓存，允许在不改变缓存状态的情况下完成内存读取请求。

因为合理程序的未命中率较低，CFENCE的性能开销较低。



减少栅栏数量的优化

1. 对每个基本块中易受攻击的指令类型的第一个实例进行检测，对其他指令不进行检测也是安全的。可以通过在解码分支时，在硬件中设置一个标志来实现，在替代的解码器解码load，并且插入一个栅栏，同时清除标志，当标志被清除的时候，就是用原来的解码器。
2. 利用新的解码器级信息流跟踪（DLIFT）策略，只为不信任的数据上易受攻击的load操作添加栅栏。

### DLIFT

在流水线的解码阶段而不是在提交阶段提供污点信息。

挑战在于流水线前段比其他部分优先执行，在解码器阶段从寄存器堆中读取的污点时不准确的。

DLIFT将污点信息分为四个结构

1. 跟踪和维护架构寄存器污点信息的解码器级污点图
2. 带有污点信息的物理寄存器堆，执行时维护动态计算的污点信息。
3. 带有污点位图的TLB和页表，以跟踪缓存块级污点信息
4. 维护验证的架构寄存器污点信息的提交级污点图。

第一个结构时DLIFT新增的。解码时的污点信息不准确，load的有效地址在解码阶段获取不到，保守的方式时将所有未知的有效地址都标记为污点，但会严重降低性能。DLIF使用乐观的方式，即假设未知的有效地址都没有被污染，并依靠在执行阶段的错误检测来验证预测的污点和动态评估污点，当检测到一个未标记的污点时，更新推测的污点图，并冲刷流水线，重新从未标记的污点开始执行。

DLIFT引擎能够跟踪指令序列和执行模式，可以从错误的推测中快速恢复，并针对特定的脆弱目标插入栅栏。

将最先进的基于栅栏的Spectre缓解措施的栅栏开销减少6倍。

# [DOM][6]



# [Efficient Invisible Speculative Execution through Selective Delay and Value Prediction][4]

几种延迟Load指令的方法

1. naive延迟，直到指令到达ROB时才提交，使所有load指令非投机地执行，会造成严重的性能损失
2. eager延迟，当load不在其他指令的shadow下时发射指令，也会造成性能损失，但比naive方法有所改善。
3. 未命中时延迟，当L1命中，正常执行，但会延迟任何可能导致可见副作用的操作，如更新替换状态，当投机执行被验证的后，CPU向缓存发出信号执行延迟的操作；当L1未命中，放弃执行指令，等到离开阴影后再次正常执行。

值预测，在L1未命中时，通过预测L1缺失的值来继续执行，而不是延迟执行。值预测时局部的，其他核时不可见的，预测器的可见状态只有在预测被验证后才会更新，因此可以在投机过程中使用，而不会产生副作用；值预测只需对L1缓存进口少量修改，而不需要对其他内存层次结构和一致性协议进行修改。

# [InvarSpec][5]

基于观察，提出了投机不变性，投机不变性指，一个指令i到达某个点时，i是否会执行和是否时投机状态无关以及i的操作数和投机状态无关，则指令i此时是投机不变的。当一条投机指令是投机不变的，且指令的操作数也准备好了，则说该指令到达了安全点（ESP）。ESP是投机指令i可以执行的最早时间点，并且保证最终使用完全相同的操作数进行提交。

如果一个指令，它的执行依赖操作数的微架构资源使用，并且这些使用的资源会暴露操作数，那么称这个指令为发射器（transmitter），这里指load指令。消除（Squashing）指令是指可能会产生导致安全问题的清空流水线的指令。这里可能是分支、load或可能造成异常的指令。

若一个消除指令已经执行且产生了最终结果，则该消除指令已经到达了他的安全结果点（OSP）。

基于TSO的x86架构，执行的消除指令到达OSP（Outcome Safe Point）的条件为，若指令i不是load指令，则当所有旧的分支都确定了且ROB中最多有一个旧的load指令，且这个load指令处于不能被消除的位置。若指令i是一个load指令，则当指令i在ROB中处于一个不能在被消除的点。综上，当load指令在ROB头部的时候，才能不会被消除。

当一个load指令i的操作数准备好了，并且它的每个较早的消除指令都到了OSP，则称该load指令i已经到了ESP点。

一个指令i的安全指令时指旧的消除指令，即使这些指令没有执行或产生最终结果，也不会影响指令i成为投机不变的指令。

对于一个给定的load指令i，安全的分支是那些结果不会影响指令i是否执行以及指令i将使用的操作数。

利用投机不变性，可以将一些指令的栅栏删除而不会影响安全性。



InvarSpec框架有两个部分，程序分析通道和相关硬件。

程序分析通道对每个受保护的指令i，识别出对i安全的旧指令集，运行时，InvarSpec微架构加载这个信息，将这些指令的程序计数器（PC）置于发射器的安全集中（SS），并利用它来确定投机性指令合适可以在不需要保护的情况下提前执行。当一个发射指令执行的时候，InvarSpec硬件会计算ESP条件，并且确定该指令是否到达了ESP。而SS 中较旧的分支和load指令不需要达到它们的OSP，硬件就可以得指令i已达到其 ESP 的结论。 结果，指令i可以更快地到达其 ESP 并且可以更快地执行

同时也会为每条消除指令j产生安全集，此时，不在发射指令安全集中的消除指令j也能够尽快地判断是否到达OSP。

InvarSpec结合了编译器和硬件机制，包括一个二进制文件的分析通道，以及在运行时使用这些信息的流水线微架构。

评估显示，InvarSpec有效地减少了硬件防御方案的执行开销。例如，在SPEC17上，它将栅栏保护的平均执行开销从195.3%降低到108.2%，将Delay-On-Miss的执行开销从39.5%降低到24.4%，而将InvisiSpec的执行开销从15.4%降低到10.9%。

# [MuonTrap][1]

## 应对目标

主要是应对不同域之间的旁路通道，而不是任意的旁路通道

## 基本想法

在核和L1缓存中间增加一个小型的快速的L0过滤缓存，防止投机执行的结果在内存层级中传播。L0缓存会随着上下文的切换而清空。

近考虑有内存系统造成的旁路攻击，近关注适用范围更广、更难预防的攻击。

## 详细设计

代码会从传统的缓存系统中访问数据，但是不会从非推测性（L1-L3）的缓存中驱逐数据，也不改变数据，但会改变L0中的数据。当指令提交时，数据才有可能从L0拷贝会L1中。

L0缓存会在上下文切换、沙盒中进程切换时刷新，也可以在错误投机时刷新。因此，投机执行的数据不会被跨进程访问。

L0中的每个缓存行有一个提交位，未提交的数据不会写回L1，有一个有效位，来标志当前缓存是否有效。缓存行同时使用虚拟地址和物理地址标记。

L0缓存也可以增加一致性机制。

指令预取也可能会泄漏敏感信息，因此，MuonTrap中仅会根据已经提交的指令来出发预取，而不会根据投机执行的指令进行预取，另外，还会在L0中，为每个缓存行增加一个标签，表明是从哪个非投机的层次结果中引入的，当指令提交的时候，就会否送相应的预取通知，避免不必要的预取。

对于指令缓存可以增加L0缓存来避免可能存在的旁路通道。

为了防止通过TLB的攻击，将TLB也存储在L0中，指令提交后，相关的TLB被移动到非推测的TLB中。

在多核处理器中，每个核会运行一个线程，需要防止线程之间无法无法推测其他线程的数据，因此，每个线程必须有独立的L0缓存，或者使用进程ID进行区分。

在沙箱中运行时，只需在沙箱的边界插入投机屏障即可，而不需要想目前那样在整个程序中插入屏障。

在PEC CPU2006上实现4%的减速，最坏情况为47%

# [Hardware-Software Contracts for Secure Speculation][3]



# Reference

[1]: https://dl.acm.org/doi/pdf/10.1109/ISCA45697.2020.00022
[2]: http://www.cs.virginia.edu/venkat/papers/asplos2019.pdf
[3]: https://spectector.github.io/papers/hwsw-contracts.pdf
[4]: https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=8980347
[5]: https://people.csail.mit.edu/mengjia/data/invarspec.pdf
[6]: https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=8675250