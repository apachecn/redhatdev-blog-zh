# 扩展架构选择以更好地武装 Red Hat Enterprise Linux 开发人员

> 原文：<https://developers.redhat.com/blog/2018/04/19/rhel-expanding-architectural-choices-arm-developers>

Red Hat Enterprise Linux 继续为企业系统管理员和开发人员提供最佳体验，并为将工作负载迁移到公共云和私有云提供坚实的基础。实现这种普遍性的方法之一是 [Red Hat 的多架构计划](https://www.redhat.com/en/blog/open-across-all-architectures-introducing-red-hat’s-multi-architecture-initiative)，该计划专注于将 Red Hat 的软件组合引入不同的硬件架构。

上周[红帽企业版 Linux 7.5 上线](https://www.redhat.com/en/about/press-releases/red-hat-strengthens-hybrid-clouds-backbone-latest-version-red-hat-enterprise-linux)。它提出了几项与开发人员和系统管理员相关的改进，例如通过驾驶舱控制台进行高级 GUI 系统管理，这应该有助于新的 Linux 管理员、开发人员和 Windows 用户执行专家任务，而不必进入命令行。

这个版本也标志着 Red Hat Enterprise Linux 的一个新的里程碑:所有支持的架构现在都同时启用。支持的架构列表包括 x86_64、PowerPC Big Endian 和 Little Endian、s390x，以及最近推出的 [64 位 Arm](https://www.redhat.com/en/blog/red-hat-introduces-arm-server-support-red-hat-enterprise-linux) 和 [IBM POWER9](https://www.redhat.com/en/blog/red-hat-launches-support-latest-ibm-hardware-red-hat-enterprise-linux) 架构。

此外，我们正在通过在免费的开发人员订阅中包含 64 位 Arm 服务器的操作系统，使开发人员更容易访问新的架构。你现在可以从 developers.redhat.com 直接下载 Red Hat Enterprise Linux for ARM。如果你没有账号，加入红帽开发者计划；[注册简单且免费](http://developers.redhat.com/register)。

为了避免您在安装和运行这个操作系统版本时遇到一些麻烦，您应该知道，并不是所有的 Arm 系统都被设计为真正的企业级服务器。我们的操作系统专注于启用支持 [SBSA](https://en.wikipedia.org/wiki/Server_Base_System_Architecture) 和 [SBBR](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.den0044b/index.html) 标准的 Arm 服务器硬件，并且只使用 [UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) 启动。这意味着，按照设计，你将无法在 [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi) 或类似的轻量级设备上运行这个操作系统。

我们期待看到更多的企业开发人员为他们的软件移植和测试工作获取这些位，特别是因为包含的用户平台工具和实用程序是所有现有 Red Hat Enterprise Linux 7 版本的标准。Red Hat Enterprise Linux for ARM 的重大新闻是包含了 4.14 Linux 内核。此外，如果您需要更新的软件组件或编译器，请务必获取最新版本的[开发工具集](https://developers.redhat.com/products/developertoolset/overview/)和[软件集合](https://developers.redhat.com/products/softwarecollections/overview/)，它们可以在所有支持的架构上免费获得，包括 64 位 Arm。

最重要的是，无论你决定做什么，都要乐在其中！

*Last updated: November 15, 2018*