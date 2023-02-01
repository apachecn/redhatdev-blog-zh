# 介绍 CodeReady Linux Builder

> 原文：<https://developers.redhat.com/blog/2018/11/15/introducing-codeready-linux-builder>

RHEL 8 引入了一个新的库，CodeReady Linux Builder(或简称为“Builder”)，开发者在为 RHEL 开发应用时可能会用到它。众所周知,“开发人员”并不是一个通用的术语。因此，我借此机会尝试解释您在开发活动中可能需要 Builder 的时候。

首先，如果你是一个典型的 web 开发人员，处理 PHP、Ruby 或 Perl，你不太可能需要通过 Builder 交付的内容。在大多数情况下，AppStream 存储库中提供的 PHP 包、Ruby gems 和 Perl 模块将提供足够的功能来开发和运行您自己创建的应用程序，以及运行像 Drupal、Wordpress、Rails 或 Twiki 这样的框架。请参见适当的 [如何让这些东西开始运行](http://developers.redhat.com/rhel8) 。

Ruby 和 Perl 都在构建器库中提供了额外的库。但是，它们不太常用，或者只在构建时使用。

接下来是 Java 开发人员。同样，AppStream 中已经提供了您通常会用到的许多功能和 jar。例如 ant、maven 和 apache-commons-logging 可以直接在 AppStream 中找到。但是，如果您需要一些只构建的组件，您可以在构建器存储库中找到它们。

如果你是. Net 开发者，你可以直接在 AppStream 中找到核心运行时&工具，作为“dotnet”包。当您构建应用程序时，您将从 Microsoft 或这些依赖项的上游获取大部分依赖项。作为. Net 开发人员，您不需要构建器存储库。

转到传统的编译语言，构建器库确实是针对你的。对于像 C 和 C++这样的语言，许多头文件、devel 包等等。可以在构建器存储库中找到。作为这类开发人员，您肯定希望在您的构建机器上启用构建器存储库。但是，通常情况下，您不需要在运行时部署中启用存储库。

很像。Net，LLVM/Clang，Go & Rust 语言编译器直接在 AppStream 中提供，带有一些支持开发的工具。如果您使用这些语言中的一种，您将不需要构建器存储库。

最后但同样重要的是，当您想要打包和部署您的应用程序时，您也可以在 Builder 存储库中找到许多支持这一过程的工具。比如介子，德加纽，多西根都可以用。

希望你会发现对新的 Code Ready Linux Builder 的描述是有帮助的，我们真的希望 RHEL8 对内容库的改变会使事情变得更简单、更容易找到。

https://wp.me/p8e0as-2fmr

*Last updated: September 3, 2019*