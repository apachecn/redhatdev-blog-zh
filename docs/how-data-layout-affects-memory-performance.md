# 数据布局如何影响内存性能

> 原文：<https://developers.redhat.com/blog/2019/04/02/how-data-layout-affects-memory-performance>

大多数人对计算机内存(又名随机存取存储器或 RAM)如何运行的心理模型是不准确的。对内存中任何字节的任何访问都具有相同的低成本这一假设在现代处理器上并不成立。在本文中，我将解释开发人员需要了解现代内存以及数据布局如何影响性能。

当前的内存越来越像一个速度极快的块存储设备。处理器不是读取或写入单个字节，而是读取或写入填充高速缓存行的字节组(通常大小为 32 到 128 字节)。对存储器的访问需要超过一百个时钟周期，比在处理器上执行指令慢两个数量级。因此，如果程序员对获得更好的性能感兴趣，他们可能会重新考虑程序中使用的数据结构。

### 延迟没有改善

关于内存，首先要注意的是访问主内存的延迟没有改善。在处理器上看到的大部分带宽改进是由于在单个事务中传输更大的字节组。在 20 世纪 80 年代，处理器通常一次传输几个字节(4 个或更少的字节)。当前处理器的内存操作正在将更大的 32 到 128 字节的组作为一个组移动，这是一条高速缓存线可以容纳的数据量。每次访问存储器时，由于设置时间选择被访问的存储器位置，会有一些延迟。在更大的一组字节之间共享延迟可以降低每个字节的成本。这就好比公共汽车并不比汽车快，但是公共汽车的载客量更大，在给定的时间内，它能让更多的人在两点之间移动。

然而，这些更宽的内存操作假设所有读取或写入的数据实际上都被处理器使用。如果处理器获取一个 64 字节的内存块，并且只修改一个字节，然后将修改后的字节存储回内存，则超过 98%的内存带宽都被浪费了。如“[如何避免一次浪费几个字节的内存兆字节](https://developers.redhat.com/blog/2016/06/01/how-to-avoid-wasting-megabytes-of-memory-a-few-bytes-at-a-time/)”中所述，可以为数据对齐填充数据结构每次从存储器加载数据结构或将数据结构存储到存储器时，用于对齐数据结构中的字段的那些未使用的字节都会浪费带宽。组织数据结构以避免用于数据对齐的填充可以导致更高的有效带宽。

处理器还可能试图通过推测性地获取数据来隐藏存储器访问延迟。硬件分析存储器访问序列，并检测其间具有恒定字节数的访问。一旦检测到内存中的这些步长，处理器就会在代码实际请求内存之前开始预取内存，从而减少代码中观察到的延迟。为了使这种方法有效，代码中使用的访问模式需要非常简单，比如数组中的每第 *n* 个元素。预取机制不会减少由于指针追踪链表而导致的随机存储器访问的存储器等待时间。

### 向量式指令

新处理器包括矢量风格的指令，如高级矢量扩展(AVX)，可以并行执行四个或八个操作。但是，要使用这些指令，操作数需要对数组中的相邻元素进行分组。使用结构数组(AoS)可能会阻止对来自多个结构的字段使用 vector-style 指令。开发人员可能希望使用数组结构(SoA)来获得允许使用 vector 指令的数据布局。在数组中使用相似的元素还可以减少数据中的填充，从而获得更有效的内存带宽。

考虑到处理器处理内存的方式，开发人员可以通过将数据结构设计得更像块设备上的文件来提高内存密集型应用的性能:

1.  安排布局以最小化读/写无用字节(对齐填充)
2.  尽量减少随机访问
3.  以可预测的步距访问元素，最好是顺序访问(步距 1)

有关优化内存性能的更多详细信息，请参考 Ulrich Drepper 的“[每个程序员都应该知道的内存知识](https://akkadia.org/drepper/cpumemory.pdf)”它提供了大量关于记忆实际运作的有用信息。[英特尔 64 和 IA-32 架构优化参考手册](https://software.intel.com/sites/default/files/managed/9e/bc/64-ia-32-architectures-optimization-manual.pdf)也详细介绍了如何构建代码以获得更好的内存性能。

*Last updated: March 26, 2019*