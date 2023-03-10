# 红帽脚本语言测试版:增加了 Ruby 2.5，Perl 5.26 更新 PHP 7.1.8

> 原文：<https://developers.redhat.com/blog/2018/04/06/red-hat-scripting-languages-beta-adds-ruby-2-5-perl-5-26-updates-php-7-1-8>

Red Hat 每年两次发布新版本的编译器工具集、脚本语言、开源数据库和/或 web 工具等。以便应用程序开发人员能够获得最新的稳定版本。这些 Red Hat 支持的产品被打包成 [Red Hat 软件集合](https://developers.redhat.com/products/softwarecollections/overview/)(脚本语言、开源数据库、web 工具等。)、[红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) (GCC)，以及最近新增的编译器工具集 [Clang/LLVM、Go、Rust](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 。所有这些都是 yum 可安装的，并且包含在大多数 Red Hat Enterprise Linux 订阅和所有 Red Hat Enterprise Linux 开发人员订阅中。大多数 Red Hat 软件集合和 Red Hat Developer Toolset 组件也可以作为 Linux 容器映像用于跨 Red Hat Enterprise Linux、Red Hat OpenShift 容器平台等的混合云开发。

**Red Hat Software Collections 3.1 beta 带来了以下新的/更新的脚本语言:**

## 新增: **Ruby 2.5**

Ruby 2.5.0 是 Ruby 2.5 系列的第一个稳定版本。它引入了许多新功能和性能改进。值得注意的变化如下:

*   救援/否则/确保现在可以直接与 do/end 块一起使用。
*   添加 yield_self 以生成其上下文中的给定块。与 tap 不同，它返回块的结果。
*   支持分支覆盖和方法覆盖度量。分支覆盖率表明哪些分支被执行，哪些不被执行。方法覆盖率表明哪些方法被调用，哪些不被调用。通过运行具有这些新特性的测试套件，您将知道执行了哪些分支和方法，并更严格地评估测试套件的总覆盖率。
*   哈希#slice 和哈希#transform_keys。
*   Struct.new 可以创建接受关键字参数的类。
*   可枚举#any？，所有？，没有？，还有一个？接受模式参数。
*   顶级常量查找不再可用。
*   我们最喜欢的库之一 pp.rb 现在已经自动加载了。你不再需要写“pp”了。
*   以相反的顺序打印回溯和错误消息(最早的调用在前，最近的调用在后)。当一个长的回溯出现在你的终端(TTY)上时，你可以很容易地在回溯的底部找到原因行。请注意，只有当回溯直接打印到终端时，顺序才会颠倒。[实验性]

有哪些版本，在哪里？

*   RHSCL 包含 Ruby 1.9.3、Ruby 2.0、Ruby 2.2、Ruby 2.3 和 Ruby 2.4
*   RHEL 6 有 Ruby 1.8.7
*   RHEL 7 包含了 Ruby 2.0

Ruby 2.5 只适用于 RHEL 7。

包装名称:rh-ruby25

Linux 容器映像:rhscl-beta/ruby-25-rhel7

## **新增内容:Perl 5.26**

Perl 5.26 版本比以前的版本有了显著的性能提升。性能变化包括哈希、读取线、优化数组和哈希赋值、将单个数字字符串转换为数字、拆分和引用赋值方面的改进。这个版本还包含了对 Perl 出色的 Unicode 支持的更新，增加了 Unicode 9.0。这是一个相当小的更新，增加了一些新的字符集和 72 个新的表情符号。

有哪些版本，在哪里？

*   RHEL 6 包括 Perl 5.10
*   RHEL7 包括 Perl 5.16
*   RHSCL 包含 Perl 5.16、5.20、5.24 和现在的 5.26

Per 5.26 仅适用于 RHEL 7。

包装名称: rh-perl526

Linux 容器映像:rhscl-beta/perl-526-rhel7

## **更新版本:PHP 7.0.27**

PHP 7.0 是 PHP 的一个重要的新版本。PHP 7.1 现在带来了许多有用的特性，从短数组析构到负字符串偏移量，再到改进的返回类型。

有哪些版本，在哪里？

RHEL 6 有 PHP 5.3
RHEL7 有 PHP 5.4
RHSCL 有 PHP 5.6、7.0、7.1

PHP 7.0.27 适用于 RHEL 6 & 7；x86_64。

包装名称: rh-php70

Linux 容器映像:rhscl-beta/php-70-rhel7

## 参考资料:

*   参见 [Hello World](https://developers.redhat.com/products/softwarecollections/hello-world/) 快速安装软件集合。
*   [RHSCL 3.1 beta 发行说明](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3-beta/html/3.1_release_notes/)
*   [使用容器映像的 RHSCL 3.1 beta】](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3-beta/html/using_red_hat_software_collections_container_images/)
*   [红帽集装箱目录](https://access.redhat.com/containers/)

*Last updated: November 15, 2018*