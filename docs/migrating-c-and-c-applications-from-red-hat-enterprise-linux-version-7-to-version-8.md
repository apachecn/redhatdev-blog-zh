# 将 C 和 C++应用程序从 Red Hat Enterprise Linux 版本 7 迁移到版本 8

> 原文：<https://developers.redhat.com/blog/2020/10/08/migrating-c-and-c-applications-from-red-hat-enterprise-linux-version-7-to-version-8>

当把你在[Red Hat Enterprise Linux](https://developers.redhat.com/topics/linux)(RHEL)7 上编译的应用程序迁移到 RHEL 8 上时，你可能会遇到由于应用程序二进制接口(ABI)的变化而引起的问题。ABI 描述了应用程序及其操作环境之间的低级二进制接口。该接口需要编译器和连接器等工具，以及生成的运行时库和操作系统本身，以在以下方面达成一致:

*   应用程序的数据类型、大小和对齐方式。
*   调用约定，定义如何传递函数参数以及如何检索返回值。
*   功能和符号名称及其版本。

随着工具链和操作环境随着 Red Hat Enterprise Linux 后续版本的发展，ABI 的细节将会改变。Red Hat 对不同版本之间的应用程序兼容性做出了某些保证。这些保证在 [RHEL 7 应用兼容性指南](https://access.redhat.com/articles/rhel-abi-compatibility) (ACG)和 [RHEL 8 应用兼容性指南](https://access.redhat.com/articles/rhel8-abi-compatibility)中有所概述。这篇文章是 ACG 指南的附录。

## 迁移兼容的 C 和 C++应用程序

将您的 [C 和 C++](https://developers.redhat.com/topics/c) 应用程序代码从 RHEL 7 迁移到 RHEL 8 时，避免兼容性问题的最直接方法是在 RHEL 8 上重建代码。在迁移到新版本之前，用户可以利用集装箱化或虚拟化环境在 RHEL 7 系统上的 RHEL 8 环境中进行构建和测试。

在某些情况下，无需重新构建，就可以将基于 RHEL 7 的 C 和 C++应用程序部署到 RHEL 8。如果你遵循了 RHEL 7 ACG 中的指导，并且只依赖于在[兼容性级别 1 (CL1)](https://access.redhat.com/articles/rhel-abi-compatibility#Appendix) 中的 C 和 C++库，那么 Red Hat 在 RHEL 8 中提供了这些库的兼容版本。事实上，我们为三个主要的 RHEL 版本维护了这些库的稳定版本。但是，请注意，不能保证 RHEL 8 将提供 CL1 之外的 RHEL 7 兼容库。

## 迁移共享库

除非 C 应用程序通过`dlopen()`加载共享库，否则`ldd`命令会显示给定应用程序所依赖的库(在`dlopen()`的情况下，您可以使用`LD_DEBUG=all my_application`或`strace`命令来观察哪些库正在被加载)。然而，C++应用程序的共享库更复杂，也更受限制。

### RHEL 7 和 RHEL 8 之间的 C++ ABI 不兼容性

RHEL 7 的 C++库很可能与 RHEL 8 的相应库不兼容。原因是 RHEL 8 的系统编译器对 C++代码使用了新的 ABI，与 RHEL 7 系统编译器使用的 ABI 不同。RHEL 7 版本的 C++库使用旧的 ABI，而 RHEL 8 版本使用新的。链接到 RHEL 7 版 C++库的应用程序不能使用该库的 RHEL 8 版。ACG CL1 中仅有的 C++库是 C++标准库(`libstdc++`)和线程构建模块库(`libtbb`和`libtbbmalloc`)。

RHEL 7 和 RHEL 8 之间的 C++ ABI 的变化是由于在 C++标准库中引入了新版本的`std::string`和`std::list`。关于`std::string` ABI 变化的详细信息，参见 Jason Merril 的文章 [GCC5 和 C++11 ABI](https://developers.redhat.com/blog/2015/02/05/gcc5-and-the-c11-abi/) 。对 C++ ABI 的修改适用于所有语言模式，所以不管你是为`-std=c++11`还是其他`-std`选项编译都没关系。

### 不起作用的变通办法

如果你的应用程序代码依赖于不在 ACG CL1 中的库，那么简单地复制那些二进制依赖项可能会很有诱惑力；然而，这既不被支持也不太可能起作用。静态链接 RHEL 7 上的所有依赖项并部署到 RHEL 8 上也很有吸引力。静态链接`glibc`阻止了一些功能的运行，所以这个选项也不受支持，不被 Red Hat 推荐。静态链接到`libstdc++`是可能的，但是没有必要这样做。RHEL 8 附带的`libstdc++.so.6`向后兼容。如果 RHEL 8 上的那些库的版本与 RHEL 7 版本不兼容，静态链接到其他 C++库可能是一个解决方案。

### 构建指南

最后，ACG 提供了在 RHEL 7 上构建将在 RHEL 8 上运行的应用程序的推荐方法。这实质上意味着根据 ACG CL1 中特别列出的库依赖项之外的您自己的版本来构建您的应用程序，而不是那些由基础系统提供的库依赖项。请参见《ACG 》,了解如何为使用超出所需兼容级别的库构建的应用程序提供兼容库。

## 常见问题

在结束之前，我将回答自 RHEL 8 发布以来我们收到的最相关的问题。

### 问:如果我想在 RHEL 8 上部署应用程序，Red Hat 有什么建议？我应该在哪里构建我的应用程序？

避免兼容性问题的最直接的方法是在 RHEL 8 上构建你的应用。

### 问:我可以在 RHEL 7 上构建我的应用程序，并将其部署在 RHEL 8 上吗？

**答**:是的，如果你遵循[应用兼容性指南](https://access.redhat.com/articles/rhel-abi-compatibility)中的指导。

构建在 RHEL 7 上的给定应用程序是否可以部署在 RHEL 8 上取决于该应用程序所链接的系统库。对于 RHEL 7 ACG CL1 中的库，RHEL 8 中提供了该库的兼容版本。对于所有其他库，不能保证 RHEL 8 提供兼容版本。

对于 C++库，这个建议不是假设的，因为 RHEL 7 的 C++库很可能与 RHEL 8 的相应库不兼容。为了在 RHEL 7 上构建可以在 RHEL 8 上运行的应用程序，ACG 提出了以下建议:

> *为应用程序提供兼容性库，这些应用程序所构建的库不符合所需的兼容性级别。*

换句话说，用你自己版本的库来构建你的应用，而不是 RHEL 7 版本。这样，您的应用程序依赖于您控制的库，而不是属于 RHEL 的库，并且在主要版本之间不兼容。

### 问:为什么在 RHEL 7 上用系统编译器构建的 C++库与 RHEL 8 不兼容？

**A**:RHEL 8 上的系统编译器对 C++代码使用了新的 ABI，与 RHEL 7 系统编译器使用的 ABI 不同。RHEL 7 版本的 C++库使用旧的 ABI，RHEL 8 版本使用新的。链接到 C++库的 RHEL 7 版本的应用程序将不能使用该库的 RHEL 8 版本。

### 问:C++编译器和库做了什么改变导致了 RHEL 7 和 8 之间的 ABI 不兼容？

**A** :从 RHEL 8 开始，C++标准库中定义了`std::string`和`std::list`类型的新版本。因此，构建在 RHEL 7 上的 C++二进制文件(应用程序和库)可能无法与构建在 RHEL 8 上的 C++二进制文件链接——或者它们可能成功链接，但在运行时失败。

### 问:如果我想在 RHEL 8 上构建一个 C++应用，需要用`-std=c++11`编译吗？

**答**:不会。对 C++ ABI 的修改适用于所有语言模式，不管你是用`-std=c++11`还是`-std=c++98`或者任何其他`-std`选项编译。

### 问:我可以通过将共享库从 RHEL 7 复制到 RHEL 8 来解决库 ABI 兼容性问题吗？

**答**:将 RHEL 7 的库复制到 RHEL 8 可能可行，但是红帽不支持。将 RHEL 7 的库复制到 RHEL 8 也有不好的一面，比如当红帽为软件包发布勘误表修复 bug 或安全漏洞时，库不会被`yum` / `dnf`自动更新。

### 问:我在 RHEL 6 上构建了我的应用程序，它在 RHEL 7 上运行良好。我的应用程序也能在 RHEL 8 上运行吗？

**答**:看情况。如果应用程序只使用 RHEL 6 ACG 中 CL1 的库，那么它应该可以在 RHEL 8 上工作，因为这些库对于三个主要版本(RHEL 版本 6、7 和 8)都是稳定的。如果应用程序使用 CL1 之外的 RHEL 6 库，但仍然可以在 RHEL 7 上运行，它也可以在 RHEL 8 上运行，但也可能不能。从来没有人保证这样的应用程序能在 RHEL 7 上运行。

### 问:在主要的 RHEL 版本中，哪些包保证是 ABI 兼容的？

**答**:参见 RHEL 各版本的应用兼容性指南。兼容级别为 1 的库保证是兼容的。在 RHEL 7 上，CL1 中仅有的 C++库是`libstdc++`和线程构建模块(TBB)库。

### 问:有同时支持新旧 c++ ABI 的库吗？

**答**:是的，`libstdc++`有。TBB 图书馆不受 ABI 变化的影响，因此它们也与两种 ABI 兼容。

### 问:如何才能知道我的应用程序依赖于哪些库？

**A**:`ldd`命令显示一个二进制文件依赖的所有共享库(包括来自其他库的间接依赖)。

### 问:我可以在 RHEL 7 上为 RHEL 8(使用 RHEL 8 系统编译器)构建一个应用程序吗？

虽然理论上是可能的，但是这样做将会很笨拙并且容易出错。如果在 RHEL 7 上开发是部署在 RHEL 8 上的应用程序的一个要求，红帽建议在容器或虚拟机中运行 RHEL 8。

### 问:如果我在 RHEL 7 上静态链接到 glibc/libstdc++，产生的二进制文件能在 RHEL 8 上正常工作吗？

**答**:红帽不支持静态链接应用。静态链接`glibc`会阻止某些功能的运行，不推荐使用。静态链接`libstdc++`是可能的，但这不是必须的，因为 RHEL 8 上的`libstdc++.so.6`库是向后兼容的。如果 RHEL 8 上的那些库的版本与 RHEL 7 版本不兼容，静态链接到其他 C++库可能是一个解决方案。

*Last updated: October 6, 2020*