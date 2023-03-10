# 在 Red Hat Enterprise Linux 8 中支持 Clang/LLVM、Go 和 Rust 的生命周期

> 原文：<https://developers.redhat.com/blog/2019/11/07/support-lifecycle-for-clang-llvm-go-and-rust-in-red-hat-enterprise-linux-8>

[Red Hat Enterprise Linux(RHEL)8 . 1 . 0](https://www.redhat.com/en/about/press-releases/red-hat-ups-iq-intelligent-operating-system-latest-release-red-hat-enterprise-linux-8?sc_cid=701f2000000tyBjAAI)包括对我们的 llvm 工具集、 [go 工具集](https://developers.redhat.com/blog/2019/06/24/go-and-fips-140-2-on-red-hat-enterprise-linux/)和 rust 工具集应用流的更新，为开发人员提供这些[编译器工具链](https://developers.redhat.com/products/gcc-clang-llvm-go-rust/overview)的最新版本。这些流的上游项目进展非常快，每六个月 LLVM 和 Go 发布一次新特性，每六周(！)对于铁锈。围绕这些工具链的社区鼓励用户对用户始终保持最新版本，这就是为什么我们试图尽快获得新版本的 Red Hat Enterprise Linux。

从支持的角度来看，我们将在 [RHEL 8](https://developers.redhat.com/rhel8/) 的整个生命周期内继续支持这些应用流。我们将通过定期更新到更新的上游版本，在流中提供新的特性和错误修复。对于 llvm-toolset 和 go-toolset，您可以预期每六个月更新一次，对于 rust-toolset，您可以预期每三个月更新一次。

Go 和 Rust 语言不断发展，每次编译器更新都会添加新的特性，这就是为什么这么多用户对获得最新版本的编译器感兴趣。同时，这些编译器被设计成与旧代码保持兼容。因此，即使我们在 RHEL 8 应用流中升级到新版本的 Go 和 Rust，您也不需要更新代码库来保持其可编译性。一旦你使用 Go 或 Rust 应用程序流编译了你的有效代码，你可以假设它将在 RHEL 8 的整个生命周期中继续使用该流编译。

我们很高兴能继续为您带来最新最棒的新编译器技术。请继续关注 Red Hat 开发者博客，了解更多关于 LLVM、Go 和 Rust 的内容。

*Last updated: July 1, 2020*