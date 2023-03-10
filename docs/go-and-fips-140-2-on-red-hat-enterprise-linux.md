# 去和 FIPS 140-2 对红帽企业 Linux

> 原文：<https://developers.redhat.com/blog/2019/06/24/go-and-fips-140-2-on-red-hat-enterprise-linux>

Red Hat 通过`go-toolset`包向 [Red Hat Enterprise Linux](https://developers.redhat.com/rhel8/) 客户提供 Go 编程语言。如果这个包对您来说是新的，并且您想了解更多，请查看为了解背景而撰写的一些[以前的](https://access.redhat.com/documentation/en-us/red_hat_developer_tools/2018.2/html-single/using_go_toolset/index) [文章](https://developers.redhat.com/blog/2017/10/31/getting-started-go-toolset/)。

`go-toolset`包目前正在发售 Go 版本 1.11.x，红帽计划在 2019 年秋季发售 1.12.x。目前，`go-toolset`包只提供了 Go 工具链(例如编译器和相关工具如`gofmt`)；然而，我们正在考虑添加其他工具，以提供一个更加完整和功能全面的 Go 开发环境。

在这篇文章中，我将讨论一些我们一直致力于的 go-toolset 的改进、变化和令人兴奋的新特性。这些变化带来了许多上游改进和 CVE 修复，以及我们内部与上游一起开发的新功能。

## FIPS 140-2 验证加密

我们很高兴地宣布，我们计划为`go-toolset`提供一项新功能，允许 Go 绕过标准库加密例程，转而调用 FIPS 140-2 验证的加密库。当您的 RHEL 系统以 FIPS 模式启动时，Go 将通过一个新的包调用 OpenSSL，这个包在 Go 和 OpenSSL 之间架起了一座桥梁。您也可以通过在您的环境中设置`GOLANG_FIPS=1`来手动启用它。

这项新功能建立在先前存在的上游工作(改为调用 BoringSSL)之上，并增加了一些新功能，例如:

*   能够调用 OpenSSL (v1.0.2 和 v1.1.1)
*   遵循现代、最新的 FIPS 标准
*   系统 FIPS 设置的运行时检测
*   能够手动启用 FIPS 模式
*   更严格的 TLS 自动只使用 FIPS 批准的密码
*   还会有更多！

只有当您的系统以 FIPS 模式启动或者您通过`GOLANG_FIPS=1`明确启用它时，使用 OpenSSL 进行加密的新特性才会启用；否则，没有变化，将使用标准库加密。如果你想完全退出这个特性，你可以用`-tags no_openssl`来构建你的程序，完全禁用它，并使用纯上游标准库加密来构建。我们正在继续开发这一功能，并打算在 Go 2 中提出新的功能，以允许以标准化的方式使用 FIPS 验证的加密模块。

## 使用 go 工具集

### RHEL7 入门

首先，启用正确的回购:

```
yum-config-manager --enable rhel-7-server-devtools-rpms && \
yum-config-manager --enable rhel-7-server-rpms && \
yum-config-manager --enable rhel-7-server-optional-rpms
```

接下来，安装软件包:

```
yum install -y go-toolset-1.11
```

RHEL7 中的包 go-toolset 提供了 go 作为一个[软件集合](https://developers.redhat.com/products/softwarecollections/overview/)。要使用您已安装的 Go 版本，您必须启用它:

```
scl enable go-toolset-1.11 go version
```

### RHEL8 入门

对于 RHEL8，go-toolset 通过 [AppStream](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/) 提供。要开始，只需键入:

```
yum install go-toolset
```

这将安装 Go 1.11.x 工具链。

请注意，您不必像在 RHEL7 中那样指定 major.minor 版本。因为 go 工具集是作为 AppStream 中的一个模块提供的，所以您会自动订阅 RHEL8 流，这意味着该流中提供的 Go 工具链的任何更新都会直接发送给您。

此外，RHEL8 模块不再是软件集合。这意味着为了使用新安装的 Go 工具链，您只需像平常一样使用 Go 命令:

```
go version
```

### 退出权

要完全绕过并编译掉所有这些更改，您可以用`-tags no_openssl`构建您的程序。

例如:

```
go build -tags no_openssl
```

## 包扎

在 Red Hat，我们通过提供和创新新功能以及回馈社区来努力改进客户使用的工具。请继续关注这项工作的更新以及更多与 Go 相关的激动人心的消息！

*Last updated: June 21, 2019*