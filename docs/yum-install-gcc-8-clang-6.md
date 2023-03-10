# 如何在红帽企业版 Linux 7 上安装 GCC 8 和 Clang/LLVM 6

> 原文：<https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6>

近年来，人们做了大量的工作来改进 C/C++编译器。从事编译器自身工作的 Red Hat 工程师已经发布了许多文章，涵盖了可用性改进、T2 检测代码中可能的错误和安全问题。

[Red Hat Enterprise Linux 8 Beta](https://developers.redhat.com/rhel8)附带 GCC 8 作为默认编译器。本文向您展示如何在[红帽企业版 Linux 7](https://developers.redhat.com/products/rhel/overview/) 上安装 [GCC 8](https://developers.redhat.com/products/developertoolset/overview) 以及 [Clang/LLVM](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 6。你可以在 RHEL 7 和 8 上使用 Red Hat 更新的(和支持的)编译器。

如果您希望默认的`gcc`总是 GCC 8，或者您希望`clang`总是在您的路径中，本文展示了如何通过将软件集合添加到您的用户帐户的配置文件(点文件)中来永久启用它。还回答了一些关于软件集合的常见问题。

注意:软件包命名约定已经更改，与编译器主版本号更加一致。对于 LLVM/Clang 6，安装`llvm-toolset-6.0`。请确保您包括“点零零”！不要*不要*安装`llvm-toolset-7`，除非你要的是之前的主版本，LLVM/Clang 5。

## TL；速度三角形定位法(dead reckoning)

### 如何安装 GCC 8 和 Clang/LLVM 6.0

1.  成为 root。
2.  启用`rhscl`、`devtools`和`optional`软件回购。
3.  将 Red Hat 开发人员工具密钥添加到您的系统中。
4.  用`yum`安装`devtoolset-8` (GCC 8)和`llvm-toolset-6.0`(铿锵 6)。
5.  可选:安装 Clang 静态分析工具`scan-build`和`clang-tidy`。
6.  在您的普通用户 ID 下，运行`scl enable`将`devtoolset-8`和`llvm-toolset-6.0`添加到您的路径中。
7.  可选:通过将`scl_source`添加到您的`.bashrc`来永久启用 GCC 8 和 Clang 6。

```
$ su -
# subscription-manager repos --enable rhel-7-server-optional-rpms \
    --enable rhel-server-rhscl-7-rpms \
    --enable rhel-7-server-devtools-rpms

# yum install devtoolset-8 llvm-toolset-6.0
# yum install llvm-toolset-6.0-clang-analyzer llvm-toolset-6.0-clang-tools-extra # optional 
# exit

$ scl enable devtoolset-8 llvm-toolset-6.0 bash
$ gcc --version
gcc (GCC) 8.2.1 20180905 (Red Hat 8.2.1-3)
$ clang --version
clang version 6.0.1 (tags/RELEASE_601/final)
$ # Optionally permanently enable GCC 8 / Clang/LLVM 6.0
$ echo "source scl_source enable devtoolset-8 llvm-toolset-6.0" &amp;amp;amp;amp;amp;amp;amp;gt;&amp;amp;amp;amp;amp;amp;amp;gt; ~/.bashrc
```

## 关于软件集合

使用软件集合需要额外的步骤，因为您必须启用想要使用的集合。启用只是向您的环境添加必要的路径(`PATH`、`MANPATH`、`LD_LIBRARY_PATH`)。一旦你掌握了窍门，它们很容易使用。理解环境变量变化在 Linux/UNIX 中的工作方式真的很有帮助。只能对当前流程进行更改。当创建子进程时，它继承父进程的环境。创建子级后，在父级中进行的任何环境更改都不会影响子级。

软件集合的好处是，您可以同时安装 GCC 6、7 和 8 以及 Red Hat Enterprise Linux 附带的基本 GCC 4。用`scl enable`可以轻松切换版本。注意:最新的解释语言，如 Python 3、PHP 7 和 Ruby 2.5，也可以通过 Red Hat 软件集合获得。的。包含 C#编译器的. NET 核心包也作为软件集合分发。因此，您应该花时间熟悉软件集合。

## 使用附加开发工具启用回购

虽然默认/基本 Red Hat Enterprise Linux 软件仓库有许多开发工具，但这些都是操作系统附带的较旧版本，在操作系统的整个 10 年生命周期内都受支持。更新更频繁且具有不同支持生命周期的产品包分布在默认情况下未启用的其他仓库中。

要启用其他 repos，请以 root 用户身份运行以下命令:

```
$ su -
# subscription-manager repos --enable rhel-7-server-optional-rpms \
  --enable rhel-server-rhscl-7-rpms \
  --enable rhel-7-server-devtools-rpms
```

注意事项:

*   你可以在一行**中输入`subscription-manager`命令，不需要反斜杠**。如果您想使用多行来提高可读性，则需要使用反斜杠。
*   如果你使用的是红帽企业 Linux 的*工作站*变种，将`-server-`改为`-workstation-`。
*   上面的命令只需要运行一次。回购将保持启用状态。安装或更新软件时，`yum`将搜索所有已启用的 repos。
*   面向开发者的免费 Red Hat Enterprise Linux 订阅服务包括访问所有这些回购协议和 Red Hat Enterprise Linux 的 T2 服务器版本。*服务器*变体是一个超集。
*   有关更多信息，请参见免费订阅的[常见问题解答。](https://developers.redhat.com/articles/frequently-asked-questions-no-cost-red-hat-enterprise-linux-developer-subscription/)

要查看哪些回购可用于您当前的订阅，请运行以下命令:

```
# subscription-manager repos --list
```

要查看启用了哪些回购，请使用`--list-enabled`:

```
# subscription-manager repos --list-enabled
```

## 安装 GCC 8 和 Clang/LLVM 6.0

您可以用一个命令安装 GCC 8 和 Clang/LLVM 6.0:

```
# yum install devtoolset-8 llvm-toolset-6.0
```

注意事项:

*   如果你只想安装 GCC 8，只需安装`devtoolset-8`。
*   如果你只想安装 Clang/LLVM 6，只需安装`llvmtoolset-6.0`。
*   这些软件包将在`/opt/rh`中安装。
*   它们不会被添加到您的路径中，直到您执行了`scl enable`。见下文。

### 查看其他软件包和其他可用版本

您可以使用`yum search`来搜索其他包，并查看其他可用的版本:

GCC:

```
# yum search devtoolset
```

Clang/LLVM:

```
# yum search llvm-toolset
```

如果你想安装 Clang 的静态分析工具`scan-build`和`clang-tidy`，运行以下命令:

```
# yum install llvm-toolset-6.0-clang-analyzer llvm-toolset-6.0-clang-tools-extra
```

## 如何用 C/C++开发工具安装 Eclipse IDE

Red Hat Developer Tools 包括带有 C/C++开发工具(CDT)的 Eclipse IDE。截至 2018 年 11 月更新的版本为 4.8 光子。要安装 IDE，请运行以下命令:

```
# yum install rh-eclipse48
```

如上所述，如果你想搜索额外的包，或者查看其他可用的版本，你可以使用`yum search`。有很多 Eclipse 包，所以您可能希望将输出重定向到一个文件，然后使用`grep`、`less`或您喜欢的文本编辑器。

```
# yum search rh-eclipse
```

## 如何使用 GCC 8 或 LLVM/Clang 6 (scl 使能)

现在已经安装了 GCC 8 和 Clang/LLVM 6。您不再需要在 root 用户 ID 下运行。其余的命令应该使用您的普通用户帐户来执行。

如前所述，软件集合安装在`/opt/rh`下，不会自动添加到您的`PATH`、`MANPATH`和`LD_LIBRARY_PATH`中。命令`scl enable`将进行必要的修改并运行一个命令。由于环境变量在 Linux(和 UNIX)中的工作方式，这些更改只对由`scl enable`运行的命令有效。您可以使用`bash`作为命令来启动交互式会话。这是使用软件集合最常见的方式之一(但不是唯一的方式)。

```
$ scl enable devtoolset-8 bash
$ gcc --version
gcc (GCC) 8.2.1 20180905 (Red Hat 8.2.1-3)
```

您可以使用`which`命令来查看`gcc`在您的`PATH`中的位置。注意:一旦您退出由`scl enable`启动的 bash shell，您将恢复到原来的`PATH`，其中不再有软件集合:

```
$ which gcc
/opt/rh/devtoolset-8/root/usr/bin/gcc
$ exit
$ which gcc
/usr/bin/gcc
$ gcc --version
gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-28)
```

对于 Clang/LLVM，使用`llvm-toolset-6.0`作为集合来启用:

```
$ scl enable llvm-toolset-6.0 bash
$ clang --version
clang version 6.0.1 (tags/RELEASE_601/final)
$ which clang
/opt/rh/llvm-toolset-6.0/root/usr/bin/clang
```

您也可以同时启用这两个集合。您使用的顺序将决定路径中目录的顺序。如果两个集合中都有任何命令，则最后添加的集合将优先。

```
$ scl enable devtoolset-8 llvm-toolset-6.0 bash
$ echo $PATH
/opt/rh/llvm-toolset-6.0/root/usr/bin:/opt/rh/llvm-toolset-6.0/root/usr/sbin:/opt/rh/devtoolset-8/root/usr/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin
```

## 如何永久启用软件集合

要永久启用 GCC 8 和/或 Clang/LLVM 6.0，您可以为您的特定用户 ID 在“点文件”中添加一个`scl_source`命令。这是推荐的开发方法，因为只有在您的用户 ID 下运行的进程才会受到影响。这种方法的好处是，如果您使用的是图形桌面，从菜单中启动的任何东西都已经启用了集合。

注意:永久启用软件集合的一个警告是**没有禁用命令**。一切都在环境变量中，所以您可以绕过它，但这将是一个手动过程。

使用您喜欢的文本编辑器，将下面一行添加到您的`~/.bashrc`:

```
# Add Red Hat Developer Toolset (GCC 8 and Clang 6) to my login environment
source scl_source enable devtoolset-8 llvm-toolset-6.0
```

注意:您也可以在构建脚本的开头添加这一行来选择构建所需的编译器。如果您的构建脚本不是作为 shell/bash 脚本编写的，您可以将它包装在一个带有`source scl_source *collection-name*`命令的 shell 脚本中，然后运行您的构建脚本。

## 但是我需要 GCC 7 或者 LLVM 5；我如何安装它们？

参见上篇: [*如何在红帽企业版 Linux*](https://developers.redhat.com/blog/2018/07/07/yum-install-gcc7-clang/) 上安装 Clang/LLVM 5 和 GCC 7。

一般来说，您可以按照本文中的说明来安装作为 Red Hat 软件集合分发的编译器。用`devtoolset-6`或`devtoolset-7`代替`devtoolset-8`，或用`llvm-toolset-7`代替`llvm-toolset-6.0`。

注:没有`devtoolset-5`。GCC 5 包含在`devtoolset-4`中，如果需要你也可以安装。为了与 GCC 主要版本同步，版本号被改为 6，跳过了 5。

使用软件集合，您可以同时安装 GCC 和/或 LLVM 的多个版本，并在它们之间轻松切换。

## 如何查看特定版本的手册页

启用软件集合会将该软件集合的手册页添加到`MANPATH`前面，这是手册页的搜索路径。就像命令一样，后续目录中任何同名的现有手册页都将被隐藏。这是运行单个命令有用的时候之一，比如用`scl enable`运行`man`，而不是启动一个 bash shell。

```
$ scl enable devtoolset-8 man gcc  # see the GCC-8 man page
$ scl enable devtoolset-7 man gcc  # see the GCC-7 man page
```

因为没有 disable 命令，所以您需要一个变通方法来查看 GCC-4 的手册页:

```
$ (MANPATH=/usr/share/man &amp;amp;amp;amp;amp;amp;amp;amp;amp;&amp;amp;amp;amp;amp;amp;amp;amp;amp; man gcc)
```

上面的命令只是改变了 subshell 中的`MANPATH`,所以它不会影响其他任何东西。

注意:同样的技术也适用于其他命令，比如 GNU `info`和`INFOPATH`。

## 如何查看安装了哪些软件集合

 *您可以使用命令`scl -l`来查看安装了哪些软件集合。这将显示所有已安装的软件集合，无论它们是否已启用。

```
$ scl -l
devtoolset-8
llvm-toolset-6.0
rh-python36
```

## 如何辨别哪些软件集合启用了

 *环境变量`X_SCLS`包含当前启用的软件集合列表。这便于在 shell 脚本中使用。

```
$ echo $X_SCLS
$ for scl in $X_SCLS; do echo $scl; done
llvm-toolset-6.0
devtoolset-8
```

## 面向 C/C++开发人员的文章

Red Hat 开发人员博客上有许多对 C/C++开发人员有帮助的文章:

*   [GCC 推荐的编译器和链接器标志](https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/)—用正确的标志改进警告和代码生成。
*   [GCC 8 中的可用性改进](https://developers.redhat.com/blog/2018/03/15/gcc-8-usability-improvements/)
*   [Clang/LLVM 入门](https://developers.redhat.com/blog/2017/11/01/getting-started-llvm-toolset/)
*   [用 GCC 8 检测字符串截断](https://developers.redhat.com/blog/2018/05/24/detecting-string-truncation-with-gcc-8/)
*   [GCC 7 的隐式失败检测](https://developers.redhat.com/blog/2017/03/10/wimplicit-fallthrough-in-gcc-7/)—检测 switch 块内缺失的 break 语句。通过`-Wimplicit-fallthrough`启用警告。如果您使用`-Wextra`，这也是将启用的警告之一。
*   [使用 GCC 7 进行内存错误检测](https://developers.redhat.com/blog/2017/02/22/memory-error-detection-using-gcc/)
*   [用 GCC 插件诊断函数指针安全缺陷](https://developers.redhat.com/blog/2017/03/17/diagnosing-function-pointer-security-flaws-with-a-gcc-plugin/)
*   [更好地使用 C11 原子——第一部分](https://developers.redhat.com/blog/2016/01/14/toward-a-better-use-of-c11-atomics-part-1/)

## 产品信息和文档

有关这些 Red Hat 产品的信息和文档:

*   GCC: [红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/)
*   Clang/LLVM、Go 和 Rust 编译器:[红帽开发者工具](https://developers.redhat.com/products/clang-llvm-go-rust/overview)
*   [支持 Clang/LLVM、Go 和 Rust 的生命周期](https://developers.redhat.com/blog/2018/11/20/support-lifecycle-for-clang-llvm-go-and-rust/)
*   动态语言和许多其他更新包:[红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview/)

## 上游社区文档

*   [GCC 8 发布系列:变更、新特性和修复](https://www.gnu.org/software/gcc/gcc-8/changes.html)
*   [Clang.llvm.org](https://clang.llvm.org/index.html)—LLVM 项目现场的铿锵页面
*   [Clang 6.0 发行说明](https://releases.llvm.org/6.0.0/tools/clang/docs/ReleaseNotes.html)
*   [LLVM 6.0 发行说明](https://releases.llvm.org/6.0.1/docs/ReleaseNotes.html)

*Last updated: March 26, 2019***