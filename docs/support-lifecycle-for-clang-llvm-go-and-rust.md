# 支持 Clang/LLVM、Go 和 Rust 的生命周期

> 原文：<https://developers.redhat.com/blog/2018/11/20/support-lifecycle-for-clang-llvm-go-and-rust>

在我们最近发布了[Clang/LLVM 6.0、Go 1.10 和 Rust 1.29](https://developers.redhat.com/blog/2018/11/13/clang-llvm-6-0-go-1-10-and-rust-1-29-now-ga-for-rhel/) 之后，我想分享一下我们将如何支持它们向前发展。以前，这些包处于“ [技术预览版](https://access.redhat.com/support/offerings/techpreview) ”状态，这意味着它们是为“您在开发过程中测试功能和提供反馈”而提供的，并且“在 Red Hat 订阅级别协议下不完全受支持，功能可能不完整，并且不用于生产使用”。

现在我们已经将它们提升到完全支持状态，这意味着什么？最简单地说，正式上市(GA)意味着这些软件包已经正式进入其生命周期的“[](https://access.redhat.com/support/policy/updates/errata/#Full_Support_Phase)”全面支持阶段:

> *在完全支持阶段，合格的关键和重要安全勘误表建议(RHSAs)以及紧急和选定的高优先级错误修复勘误表建议(RHBAs)可能会在可用时发布。其他勘误表建议可能会在适当的时候发布。*
> 
> *如果可用，Red Hat 可能会自行决定提供新的或改进的硬件支持和精选的增强软件功能，通常在次要版本中提供。Red Hat 可自行决定独立于次要版本提供不需要实质性软件变更的硬件支持。*
> 
> *次要版本还将包括可用且合格的勘误表公告(RHSAs、RHBAs 和 RHEAs)。次要版本是累积性的，包括以前发布的更新的内容。在这个阶段，次要版本的焦点在于解决中等或更高优先级的缺陷。*
> 
> 在完全支持阶段，将为次要版本提供更新的安装映像。

由于这些软件包发展很快，我们将在一个精简的生命周期中支持它们，更新节奏对特定的软件包有意义。对于 Rust，这意味着每个季度(大约每 3 个月)都会有更新，对于 LLVM 和 Go，这意味着每 6 个月更新一次。

由于这些软件包移动如此之快，对它们的支持将与我们“通常”所做的略有不同——我们“通常”做的是缓慢移动并长期支持软件包的特定版本。对于 LLVM、Rust 和 Go 工具集，我们将只维护最近发布的版本。如果在旧版本中发现了错误或漏洞，补救的方法将是更新到该工具集的最新版本。如果在当前版本中发现了一个 bug，我们将在下一个计划的版本中解决它。默认情况下，这将是下一个预定的次要版本(Red Hat Enterprise Linux 8 中的应用程序流)。

关于这些工具集的更多信息可以在 developers.redhat.com 找到。

## 请参见

*   [如何在红帽企业版 Linux 上安装 GCC 8 和 Clang/LLVM 6](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/)
*   [如何在红帽企业版 Linux 上安装 Go](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world/#fndtn-go)
*   [如何在红帽企业版 Linux 上安装 Rust](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world/#fndtn-rust)

*Last updated: March 5, 2019*