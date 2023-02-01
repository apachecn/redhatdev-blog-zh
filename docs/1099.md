# 如何在 RHEL 上安装 Clang/LLVM 5 和 GCC 7

> 原文：<https://developers.redhat.com/blog/2018/07/07/yum-install-gcc7-clang>

***本文有更新版本:[如何在红帽企业版 Linux 7 上安装 GCC 8 和 Clang/LLVM 6](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/)。***

如果你用 C/C++开发，Clang 工具和 GCC 的新版本对检查你的代码很有帮助，并给你更好的警告和错误信息来帮助避免错误。较新的编译器有更好的优化和代码生成。

你可以在 [Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview/) 上使用`yum`轻松安装最新支持的用于 C、C++、Objective-C 和 FORTRAN 的 [Clang](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 和 [GCC 编译器](https://developers.redhat.com/products/developertoolset/overview)。这些编译器以软件集合的形式提供，通常每年更新两次。2018 年 5 月更新的[包括 Clang/LLVM 5 和 GCC 7.3，以及 Go 和 Rust。](https://developers.redhat.com/blog/2018/05/03/announcing-ga-for-latest-software-collections-developer-toolset-compilers/)

如果您希望默认的`gcc`总是 GCC 7，或者您希望`clang`总是在您的路径中，本文展示了如何通过将软件集合添加到您的用户帐户的概要文件(点文件)中来永久启用它。还回答了一些关于软件集合的常见问题。

## TL；速度三角形定位法(dead reckoning)

### 如何安装 Clang/LLVM 5 和 GCC 7

1.  成为 root。
2.  启用`rhscl`、`devtools`和`optional`软件回购。
3.  将 Red Hat 开发人员工具密钥添加到您的系统中
4.  用`yum`安装`devtoolset7` (GCC 7)和`llvm-toolset-7`(铿锵 5)。
5.  可选:安装 Clang 静态分析工具`scan-build`和`clang-tidy`
6.  在您的普通用户 ID 下，运行`scl enable`将`devtoolset-7`和`llvm-toolset-7`添加到您的路径中
7.  可选:通过将`scl_source`添加到您的`.bashrc`来永久启用 GCC 7 和 Clang 5

```
$ su -
# subscription-manager repos --enable rhel-7-server-optional-rpms \
    --enable rhel-server-rhscl-7-rpms \
    --enable rhel-7-server-devtools-rpms

# cd /etc/pki/rpm-gpg
# wget -O RPM-GPG-KEY-redhat-devel https://www.redhat.com/security/data/a5787476.txt
# rpm --import RPM-GPG-KEY-redhat-devel

# yum install devtoolset-7 llvm-toolset-7
# yum install llvm-toolset-7-clang-analyzer llvm-toolset-7-clang-tools-extra # optional 
# exit

$ scl enable devtoolset-7 llvm-toolset-7 bash
$ gcc --version
gcc (GCC) 7.3.1 20180303 (Red Hat 7.3.1-5)

$ clang --version
clang version 5.0.1 (tags/RELEASE_501/final)

$ # Optionally permanently enable GCC 7 / Clang 5
$ echo "source scl_source enable devtoolset-7 llvm-toolset-7" &gt;&gt; ~/.bashrc
```

## 关于软件集合

使用软件集合需要额外的步骤，因为您必须启用想要使用的集合。启用只是向您的环境添加必要的路径(`PATH`、`MANPATH`、`LD_LIBRARY_PATH`)。一旦你掌握了窍门，它们很容易使用。理解环境变量变化在 Linux/UNIX 中的工作方式真的很有帮助。只能对当前流程进行更改。当创建子进程时，它继承父进程的环境。创建子级后，在父级中进行的任何环境更改都不会影响子级。

软件集合的好处是您可以同时安装 GCC 5、6、7 以及 Red Hat Enterprise Linux 附带的 GCC 4。用`scl enable`可以轻松切换版本。注意:最新的解释语言如 Python 3、PHP 7 和 Ruby 2.5 也可以通过 Red Hat 软件集合获得。的。包含 C#编译器的. NET 核心包也作为软件集合分发。因此，您应该花时间熟悉软件集合。

## 使用附加开发工具启用回购

虽然默认/基本 Red Hat Enterprise Linux 软件仓库有许多开发工具，但这些都是操作系统附带的较旧版本，在操作系统的整个 10 年生命周期内都受支持。更新更频繁且具有不同支持生命周期的产品包分布在默认情况下未启用的其他仓库中。

要启用其他 repos，请以 root 用户身份运行以下命令:

```
$ su -
# subscription-manager repos --enable rhel-7-server-optional-rpms \
  --enable rhel-server-rhscl-7-rpms \
  --enable rhel-7-server-devtools-rpms
# cd /etc/pki/rpm-gpg
# wget -O RPM-GPG-KEY-redhat-devel https://www.redhat.com/security/data/a5787476.txt
# rpm --import RPM-GPG-KEY-redhat-devel
```

注意事项:

*   您可以在一行**中输入以上所有内容，无需反斜杠**。如果您想使用多行来提高可读性，则需要使用反斜杠。
*   如果你使用的是红帽企业 Linux 的*工作站*变种，将`-server-`改为`-workstation-`。
*   该命令只需运行一次。回购将保持启用状态。安装或更新软件时，`yum`将搜索所有已启用的 repos。
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

## 安装 GCC 7 和 Clang/LLVM 5

您可以用一个命令安装 GCC 7 和 Clang/LLVM 5:

```
# yum install devtoolset-7 llvm-toolset-7
```

注意事项:

*   如果你只想安装 GCC 7，只需安装`devtoolset-7`。
*   如果你只想安装 Clang/LLVM 5，只需安装`llvmtoolset-7`。
*   这些软件包将在`/opt/rh`中安装。
*   它们不会被添加到您的路径中，直到您执行了`scl enable`。见下文。

### 查看其他软件包，查看其他可用版本

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
# yum install llvm-toolset-7-clang-analyzer llvm-toolset-7-clang-tools-extra
```

## 如何用 C/C++开发工具安装 Eclipse IDE

Red Hat Developer Tools 包括带有 C/C++开发工具(CDT)的 Eclipse IDE。截至 2018 年 5 月更新的版本为 4.7.3 氧气。要安装 IDE，请运行以下命令:

```
# yum install rh-eclipse47
```

如上所述，如果你想搜索额外的包，或者查看其他可用的版本，你可以使用`yum search`。有很多 Eclipse 包，所以您可能希望将输出重定向到一个文件，然后使用`grep`、`less`或您喜欢的文本编辑器。

```
# yum search rh-eclipse
```

## 如何使用 GCC 7 或 Clang (scl 使能)

现在已经安装了 GCC 7 和 Clang/LLVM 5。您不再需要在 root 用户 ID 下运行。其余的命令应该使用您的普通用户帐户来执行。

如前所述，软件集合安装在`/opt/rh`下，不会自动添加到您的`PATH`、`MANPATH`和`LD_LIBRARY_PATH`中。命令`scl enable`将进行必要的修改并运行一个命令。由于环境变量在 Linux(和 UNIX)中的工作方式，这些更改将仅对由`scl enable`运行的命令生效。您可以使用`bash`作为命令来启动交互式会话。这是使用软件集合最常见的方式之一(但不是唯一的方式)。

```
$ scl enable devtoolset-7 bash
$ gcc --version
gcc (GCC) 7.3.1 20180303 (Red Hat 7.3.1-5)
```

您可以使用`which`命令来查看`gcc`在您的`PATH`中的位置。注意:一旦您退出由`scl enable`启动的 bash shell，您将恢复到原来的`PATH`，其中不再有软件集合:

```
$ which gcc
/opt/rh/devtoolset-7/root/usr/bin/gcc
$ exit
$ which gcc
/usr/bin/gcc
$ gcc --version
gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-28)
```

对于 Clang/LLVM，使用`llvm-toolset-7`作为集合来启用:

```
$ scl enable llvm-toolset-7 bash
$ clang --version
clang version 5.0.1 (tags/RELEASE_501/final)
$ which clang
/opt/rh/llvm-toolset-7/root/usr/bin/clang
```

您也可以同时启用这两个集合。该顺序将决定路径中目录的顺序。如果两个集合中都有任何命令，则最后添加的集合将优先。

```
$ scl enable devtoolset-7 llvm-toolset-7 bash
$ echo $PATH
/opt/rh/llvm-toolset-7/root/usr/bin:/opt/rh/llvm-toolset-7/root/usr/sbin:/opt/rh/devtoolset-7/root/usr/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin
```

## 如何永久启用软件集合

要永久启用 GCC 7 和/或 Clang/LLVM 5，您可以为您的特定用户 ID 在“点文件”中添加一个`scl_source`命令。这是推荐的开发方法，因为只有在您的用户 ID 下运行的进程才会受到影响。这种方法的好处是，如果您使用的是图形桌面，从菜单中启动的任何东西都已经启用了集合。

使用您喜欢的文本编辑器，将下面一行添加到您的`~/.bashrc`:

```
# Add Red Hat Developer Toolset (GCC 7 and Clang 5) to my login environment
source scl_source enable devtoolset-7 llvm-toolset-7
```

注意:您也可以在构建脚本的开头添加这一行来选择构建所需的编译器。如果您的构建脚本不是作为 shell/bash 脚本编写的，您可以将它包装在一个带有`source scl_source *collection-name*`命令的 shell 脚本中，然后运行您的构建脚本。

永久启用软件集合的一个警告是**没有禁用命令**。一切都在环境变量中，所以您可以绕过它，但这将是一个手动过程。

## 但是我需要 GCC 6，怎么安装？

如果您需要 GCC 6 而不是 GCC 7，您可以遵循这些相同的说明。安装`devtoolset-6`会安装 GCC 6.3。

注:没有`devtoolset-5`。GCC 5 包含在`devtoolset-4`中，如果需要你也可以安装。为了与 GCC 主要版本同步，版本号被改为 6，跳过了 5。

使用软件集合，您可以同时安装 GCC 4、5、6 和 7，并在它们之间轻松切换。

## 如何查看特定版本的手册页

启用软件集合会将该软件集合的手册页添加到`MANPATH`前面，这是手册页的搜索路径。就像命令一样，后续目录中任何同名的现有手册页都将被隐藏。这是运行单个命令有用的时候之一，比如用`scl enable`运行`man`，而不是启动一个 bash shell。

```
$ scl enable devtoolset-7 man gcc  # see the GCC-7 man page
$ scl enable devtoolset-6 man gcc  # see the GCC-6 man page
```

因为没有 disable 命令，所以您需要一个变通办法来查看 GCC-4 的手册页:

```
$ (MANPATH=/usr/share/man &amp;&amp; man gcc)
```

上面的命令只是改变了 subshell 中的`MANPATH`,所以它不会影响其他任何东西。

注意:同样的技术也适用于其他命令，比如 GNU `info`和`INFOPATH`。

## 如何查看安装了哪些软件集合

 *您可以使用命令`scl -l`来查看安装了哪些软件集合。这将显示所有已安装的软件集合，无论它们是否已启用。

```
$ scl -l
devtoolset-7
llvm-toolset-7
rh-python36
```

## 如何辨别哪些软件集合启用了

 *环境变量`X_SCLS`包含当前启用的软件集合列表。这便于在 shell 脚本中使用。

```
$ echo $X_SCLS
$ for scl in $X_SCLS; do echo $scl; done
llvm-toolset-7
devtoolset-7
```

## 面向 C/C++开发人员的文章

Red Hat 开发人员博客上有许多对 C/C++开发人员有帮助的文章:

*   [Clang/LLVM 入门](https://developers.redhat.com/blog/2017/11/01/getting-started-llvm-toolset/)
*   [GCC 推荐的编译器和链接器标志](https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/)—用正确的标志改进警告和代码生成。
*   [GCC 7 的隐式失败检测](https://developers.redhat.com/blog/2017/03/10/wimplicit-fallthrough-in-gcc-7/)—检测 switch 块内缺失的 break 语句。通过`-Wimplicit-fallthrough`启用警告。如果您使用`-Wextra`，这也是将启用的警告之一
*   [使用 GCC 7 进行内存错误检测](https://developers.redhat.com/blog/2017/02/22/memory-error-detection-using-gcc/)

## 产品信息和文档

有关这些 Red Hat 产品的信息和文档:

*   GCC: [红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/)
*   Clang/LLVM、Go 和 Rust 编译器:[红帽开发者工具](https://developers.redhat.com/products/clang-llvm-go-rust/overview)
*   动态语言和许多其他更新包:[红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview/)

## 上游社区文档

*   [GCC 7 发布系列的新功能](https://gcc.gnu.org/gcc-7/changes.html)
*   Clang.llvm.org LLVM 项目网站上的铿锵页面
*   [Clang 5.0 发行说明](http://releases.llvm.org/5.0.2/tools/clang/docs/ReleaseNotes.html)
*   [LLVM 5.0 发行说明](http://releases.llvm.org/5.0.1/docs/ReleaseNotes.html)

## 更新文章:[如何在红帽企业版 Linux 上安装 GCC 8 和 Clang/LLVM 6](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/)

*Last updated: March 7, 2019***