# 在 ARM 上运行了多年的 Linux 之后，ARM 服务器上的红帽之年是什么时候？

> 原文：<https://developers.redhat.com/blog/2017/07/05/after-years-of-linux-on-arm-when-is-the-year-of-red-hat-on-arm-servers>

从无处不在的 Raspberry Pi 等业余 SoC 设备到移动设备市场的完全主导，ARM 处理器已经证明了该架构的价值。很容易理解为什么 ARM 处理器能够在这个市场上爆发，因为它们能够在一个相当小的物理空间中装载相当多的性能。以高通的骁龙 400 处理器为例，该处理器被用于包括华为手表在内的许多产品中。该处理器采用 14 纳米设计，提供双核、快速、高性能。这使得它足够小，可以戴在手腕上，而且它的效率足够高，可以将功耗限制在足以依靠电池运行的水平，电池也可以戴在手腕上。ARM 处理器在这些领域的实用性是众所周知的，而 RHEL 似乎对移动、嵌入式设备或业余爱好者市场兴趣不大的事实也是众所周知的。不过，Red Hat 开发人员应该感兴趣的是 ARM 处理器似乎正在展示的企业服务器市场的潜力。虽然 x86 设备可能会继续用于工作站和笔记本电脑，但 ARM 处理器的低功耗和小物理设计使其受到功耗和空间限制的服务器市场的关注。

早在 2013 年，服务器市场就开始了拥抱 ARM 架构的漫长旅程。虽然这在几年前看起来很有潜力，但这些机器正变得越来越主流。以 [Cavium 的 ThunderX 处理器](http://www.cavium.com/ThunderX_ARM_Processors.html)为例，它拥有 48 个内核，频率为 2.5 GHz，能够安装在 1U 服务器上。这当然在 x86 架构处理器(如众所周知的英特尔至强系列)的性能和价格范围之内。随着三星宣布使用 10 纳米光刻技术的 ARM 处理器等进步，ARM 处理器在服务器市场的增长潜力看起来甚至更高。正是考虑到这一点，Red Hat 一直在为 ARM 架构开发，这也是 Red Hat 开发人员感兴趣的原因。

**ARM 简史**

ARM 最初代表 Acorn RISC Machine，后来改为 Advanced RISC Machine。为了理解它的重要性，人们应该理解 RISC 或“精简指令集计算机”的含义精简指令集计算机的想法实际上不是限制发送到 CPU 的指令或潜在指令的数量，而是减少指令对内存的调用次数。x86 架构被称为 CISC，或“复杂指令集计算机”，发送到 CPU 的每条指令可能需要多达 12 个数据存储周期。RISC 机器将数据存储周期限制为每条指令一个周期。

RISC 设备甚至可以追溯到 20 世纪 60 年代。最著名的执行是斯坦福大学的 MIPS 和加州大学伯克利分校的 RISC(这显然为该概念提供了名称)。太阳微系统公司开发了 RISC 并将其商业化为 SPARC(或可扩展处理器体系结构)。ARM(或 Acorn RISC 机器)在 20 世纪 80 年代开始开发，并在 20 世纪 90 年代与苹果合作，最终成为高级 RISC 机器。和苹果的合作产生了 ARM v6，现在我们看到了 ARM v7 和 v8 的生产。随着这些发展，ARM 主导了移动领域，98%的手机都包含 ARM 处理器。据估计，仅 2013 年就生产了至少 100 亿个 ARM 处理器。

**手臂的限制**

ARM 操作系统实施的主要问题是缺乏标准。虽然这在移动市场提供了一个福音，允许手机制造商采用嵌入式软件方法，但这在企业服务器市场有很大的局限性。在 2014 年接受 Register 采访时，Red Hat 的首席 ARM 架构师乔恩·马斯特斯(Jon Masters)表示:“如果我在服务器上有 20 种不同的串行端口连接方式，那就有问题了。”他还将 ARM 市场称为“嵌入式动物园”。问题是，虽然设备和硬件定制适用于嵌入式设备和手机，但在工业环境中，这只是一个问题。

已经尝试并且正在尝试在军火市场中解决这个问题。ARM 制定了[服务器基础系统架构(SBSA)](https://static.docs.arm.com/den0029/a/Server_Base_System_Architecture_v3_1_ARM_DEN_0029A.pdf) 标准，规定了 ARM 处理器必须遵循的架构，以及[服务器基础引导要求(SBBR)](http://infocenter.arm.com/help/topic/com.arm.doc.den0044b/DEN0044B_Server_Base_Boot_Requirements.pdf)，指定了 ARM 处理器引导的标准方式。显然，架构标准对企业服务器的执行有影响。引导标准也很重要，因为大多数嵌入式和移动设备都使用像 [U-Boot](http://www.denx.de/wiki/U-Boot/) 这样的引导加载程序，这限制了为该处理器生成发行版的能力。随着 SBSA 和 SBBR 标准的引入，制造商需要遵循这些标准。

**ARM 服务器:现在和未来**

上面提到的 Cavium 的 ThunderX 处理器，以及 Jon Masters 称之为 ARM 服务器理想标准的 AMD 的“Seattle”Opteron 1100 处理器，显示了将基于 ARM 的服务器推向主流的潜力。复杂的情况将 AMD 发布 ARM 处理器的时间从 2015 年推迟了一年，到 2016 年，这使其进一步落后于新的英特尔至强设备。软件支持似乎正在等待这些处理器经过验证的可行性，以及它们与 x86 设备竞争或超越 x86 设备的能力。早在 2014 年，红帽就发布了一款不支持 ARM 的[“早期访问”版本的 RHEL，该版本带有制造商代码。从那以后，似乎没有太大的变化，至少对用户来说，RHEL 的“早期访问”版本仍然可以获得，没有制造商的代码支持。](https://www.redhat.com/en/about/press-releases/red-hat-launches-arm-partner-early-access-program-partner-ecosystem)

RHEL 在 ARM 系统上的可行性似乎是显而易见的，特别是考虑到上游的大量开发与 [Fedora ARM 项目](https://arm.fedoraproject.org/)以及 ARM 上[CentOS](https://lists.centos.org/pipermail/arm-dev/)的活动。事实上，带有 Fedora ARM 的 Raspberry Pi 3 现在可以成为一个稳定、高性能的家庭服务器。如果市场开始向 ARM 服务器过渡，RHEL 很可能仍将在该领域处于领先地位，因此对于开发人员来说，记住架构的这种过渡非常重要。用于服务器和工作站的 ARM 可能已经达到了极限，它将退回到业余爱好者和移动架构。然而，它的快速增长和最初的服务器性能确实显示了未来市场主导地位的潜力。考虑到这一点，几年后 RHEL 的发展可能意味着 RHEL ARM 的发展。

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: June 30, 2017*