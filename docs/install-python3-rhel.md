# 如何在红帽企业版 Linux 上安装 Python 3

> 原文：<https://developers.redhat.com/blog/2018/08/13/install-python3-rhel>

本文展示了如何在[红帽企业版 Linux](https://developers.redhat.com/products/rhel/overview/) 7 上安装 Python 3、`pip`、`venv`、`virtualenv`和`pipenv`。完成本文中的步骤后，您应该能够很好地使用 RHEL 学习许多 Python 指南和教程。注:RHEL 8 安装请见 RHEL 8 上的 *[Python。](https://developers.redhat.com/blog/2018/11/14/python-in-rhel-8/)*

使用 Python 虚拟环境是隔离特定于项目的依赖关系并创建可再现环境的最佳实践。还介绍了在 RHEL 7 上使用 Python 和软件集合的其他技巧和常见问题。

在 RHEL 上安装 Python 3 有很多不同的方法。本文使用 Red Hat 软件集合，因为这些集合为您提供了由 Red Hat 构建和支持的最新 Python 安装。在开发过程中，支持可能对你来说不那么重要。然而，对于那些必须部署和操作您编写的应用程序的人来说，支持是很重要的。要理解这一点的重要性，请考虑当您的应用程序处于生产环境中，并且在核心库中发现了一个严重的安全漏洞(例如 SSL/TLS)时会发生什么。这种类型的场景是许多企业使用 Red Hat 的原因。

本文使用的是 Python 3.6。在撰写本文时，这是最新的稳定版本。但是，您应该能够将这些说明用于 Red Hat 软件集合中的任何 Python 版本，包括 2.7、3.4、3.5 以及未来的集合，如 3.7。

在本文中，将讨论以下主题:

1.  [TL；DR](#tldr) (步骤总结)
2.  [为什么使用红帽软件集合](#why-rhscl)
3.  [完整的安装步骤及说明](#full-install)
4.  [如何通过红帽软件集合使用 Python 3](#how-to-use)
5.  [使用 Python 虚拟环境](#create-env)
    1.  [我应该用`venv`还是`virtualenv`还是别的什么？](#which-venv)
    2.  [使用`venv`](#use-venv)
    3.  [使用`virtualenv`](#use-virtualenv)
    4.  [使用`pipenv`](#use-pipenv) 管理应用依赖关系
6.  [使用 Python 的一般技巧](#python-tips)
7.  [使用软件集合的技巧](#scl-tips)
    1.  [在虚拟环境之前启用 Python 集合](#scl-first)
    2.  [如何永久启用软件集合](#scl-permanent)
    3.  [如何在#中使用 RHSCL 的 Python 3！脚本的一行](#scl-script)
    4.  [如何判断哪些软件集合被启用](#which-scl-enabled)
    5.  [如何查看安装了哪些软件集合](#which-scl-installed)
8.  [故障排除](#troubleshooting)
9.  [更多信息:在 Red Hat 平台上用 Python 开发](#more-info)

## TL；速度三角形定位法(dead reckoning)

以下是基本步骤，这样你就可以开始了。见下文的解释和更多细节。

### 如何在 RHEL 上安装 Python 3

1.  成为`root`。
2.  使用`subscription-manager`启用`rhscl`和`optional`软件回购。
3.  使用`yum`安装`@development`。这确保你已经得到了 GCC、`make`、`git`等。因此您可以构建任何包含编译代码的模块。
4.  使用`yum`安装`rh-python36`。
5.  可选:使用`yum`从 RHSCL RPMs 安装`python-tools`、`numpy`、`scipy`和`six`。

```
$ su -
# subscription-manager repos --enable rhel-7-server-optional-rpms \
  --enable rhel-server-rhscl-7-rpms
# yum -y install @development
# yum -y install rh-python36

# yum -y install rh-python36-numpy \
 rh-python36-scipy \ 
 rh-python36-python-tools \
 rh-python36-python-six

# exit
```

### 在 RHEL 上使用 Python 3

1.  在您的普通用户 ID 下，运行`scl enable`将`python 3`添加到您的路径中。
2.  创建一个 Python 虚拟环境并激活它。(注意:您的提示已更改为显示虚拟环境。)
3.  使用`pip`在隔离的环境中安装您需要的任何附加模块，而不是`root`。

```
$ scl enable rh-python36 bash
$ python3 -V
Python 3.6.3

$ python -V  # python now also points to Python3 
Python 3.6.3

$ mkdir ~/pydev
$ cd ~/pydev

$ python3 -m venv py36-venv
$ source py36-venv/bin/activate

(py36-venv) $ python3 -m pip install ...some modules...
```

如果您开始一个新会话，以下是使用虚拟环境的步骤:

```
$ scl enable rh-python36 bash

$ cd ~/pydev
$ source py36-env/bin/activate
```

## 为什么使用红帽软件收藏

使用 Red Hat 软件集合的好处是，您可以同时安装多个版本的 Python 以及 RHEL 7 附带的基础 Python 2.7。用`scl enable`可以轻松切换版本。

注意:最新的稳定软件包。Net Core、Go、Rust、PHP 7、Ruby 2.5、GCC、Clang/LLVM、Nginx、MongoDB、MariaDB、PostgreSQL 等等都是`yum` -可安装为软件集合。因此，您应该花时间熟悉软件集合。

使用软件集合需要一个额外的步骤，因为您必须启用想要使用的集合。启用只是向您的环境添加必要的路径(`PATH`、`MANPATH`、`LD_LIBRARY_PATH`)。一旦你掌握了窍门，软件集合就很容易使用了。理解环境变量变化在 Linux/UNIX 中的工作方式真的很有帮助。只能对当前流程进行更改。当创建子进程时，它继承父进程的环境。创建子级后，在父级中进行的任何环境更改都不会影响子级。因此，`scl enable`所做的更改将只影响当前的终端会话或从它启动的任何东西。本文还展示了如何为您的用户帐户永久启用软件集合。

* * *

## 安装先决条件

### 安装开发工具，包括 GCC、make 和 git

如果你安装依赖于编译代码的模块，你需要工具来编译它们。如果您尚未安装开发工具，请运行以下命令:

```
$ su -
# yum install @development
```

### 使用附加开发工具启用回购

虽然默认/基本 RHEL 软件回购协议有许多开发工具，但这些都是操作系统附带的旧版本，在操作系统的整个 10 年生命周期内都受支持。更新更频繁且具有不同支持生命周期的产品包分布在默认情况下未启用的其他仓库中。

红帽软件收藏在`rhscl`回购中。RHSCL 包依赖于`optional-rpms` repo 中的包，所以您需要同时启用这两个包。

要启用额外的回购，请以`root`的身份运行以下命令:

```
$ su -
# subscription-manager repos \
 --enable rhel-7-server-optional-rpms \
 --enable rhel-server-rhscl-7-rpms
```

注意事项:

*   您可以在一行**中输入以上所有内容，无需反斜杠**。如果您想使用多行来提高可读性，则需要使用反斜杠。
*   如果您使用的是 RHEL*工作站*的变体，请将`-server-`更改为`-workstation-`。
*   该命令只需运行一次。回购将保持启用状态。安装或更新软件时，`yum`将搜索所有已启用的 repos。
*   面向开发者的[免费 RHEL 订阅](https://developers.redhat.com/products/rhel/overview/)包括访问所有这些回购和 RHEL 的*服务器*变种。*服务器*变体是一个超集。
*   有关更多信息，请参见免费订阅的[常见问题解答。](https://developers.redhat.com/articles/frequently-asked-questions-no-cost-red-hat-enterprise-linux-developer-subscription/)

要查看哪些回购可用于您当前的订阅，请运行以下命令:

```
# subscription-manager repos --list
```

要查看启用了哪些回购，请使用`--list-enabled`:

```
# subscription-manager repos --list-enabled
```

* * *

## 安装 Python 3

现在可以用`yum`安装 Python 3.6(或 RHSCL 中的其他版本):

```
# yum install rh-python36
```

注意事项:

*   这些软件包将在`/opt/rh/`中安装。
*   在您运行`scl enable`之前，它们不会被添加到您的路径中。见下文。
*   对于其他版本的 Python，使用以下作为包/集合名:
    Python 3.5:`rh-python35`
    Python 3.4:`rh-python34`
    Python 2 . 7 . 13:`python27`
*   许多附加软件包将作为依赖项安装。这些包括`python-devel`、`pip`、`setuptools`和`virtualenv`。
*   如果您必须构建任何动态链接到 Python 的模块(如 C/C++代码)，那么`python-devel`包包含所需的文件。

### 安装附加软件包

或者，您可能希望安装软件集合中的以下 RPM 软件包:

*   Python 工具:`rh-python36-python-tools`是 Python 3、`2to3`和`idle3`中包含的工具集合。
*   Numpy: `rh-python36-numpy`是 Python 的一个快速多维数组工具。
*   Scipy: `rh-python36-scipy`为 Python 提供科学工具。
*   六:`rh-python36-python-six`提供 Python 2 和 3 兼容的实用程序。
*   Sqlalchemy: `rh-python36-python-sqlalchemy`是一个模块化的灵活的 Python ORM 库。
*   PyYAML: `rh-python36-PyYAML`是 Python 的 YAML 解析器和发射器。
*   Simplejson: `rh-python36-python-simplejson`是一个简单、快速、可扩展的用于 Python 的 json 编码器/解码器。

示例:

```
# yum install rh-python36-numpy \
 rh-python36-scipy \ 
 rh-python36-python-tools \
 rh-python36-python-six
```

注意:默认情况下，系统模块不会与 Python 虚拟环境一起使用。创建包含系统模块的虚拟环境时，使用选项`--system-site-packages`。

* * *

## 如何使用 Python 3 ( `scl enable`)

Python 3 现已安装。您不再需要在`root`用户 ID 下运行。其余的命令应该使用您的普通用户帐户来执行。

如前所述，软件集合安装在`/opt/rh`下，不会自动添加到您的`PATH`、`MANPATH`和`LD_LIBRARY_PATH`中。命令`scl enable`将进行必要的修改并运行一个命令。由于环境变量在 Linux(和 UNIX)中的工作方式，这些更改将仅对 scl `enable`运行的命令生效。您可以使用`bash`作为命令来启动交互式会话。这是使用软件集合最常见的方式之一(但不是唯一的方式)。

```
$ scl enable rh-python36 bash
$ python3 -V
Python 3.6.3

$ python -V # python now points to Python 3
Python 3.6.3

$ which python
/opt/rh/rh-python36/root/usr/bin/python
```

注意:启用 Python 集合会使路径中没有版本号的`python`指向 Python 3。`/usr/bin/python`还是会是 Python 2。您仍然可以通过键入`python2`、`python2.7`或`/usr/bin/python`来运行 Python 2。建议您使用版本号，以避免对`python`的含义产生任何歧义。这也适用于`.../bin`中的其他 Python 命令，如`pip`、`pydoc`、`python-config`、`pyvenv`和`virtualenv`。更多信息，参见 [PEP 394](https://www.python.org/dev/peps/pep-0394/) 。

注意:参见下面的*如何永久启用软件集合*将 Python 3 永久放在您的路径中。

* * *

## 创建 Python 虚拟环境(最佳实践)

使用 Python 虚拟环境是隔离特定于项目的依赖关系并创建可再现环境的最佳实践。换句话说，这是一种避免导致依赖性地狱的冲突依赖性的方法。使用虚拟环境将允许您使用`pip`在普通用户 ID 下的隔离目录中安装项目所需的任何模块。您可以轻松地拥有多个具有不同依赖关系的项目。要处理特定的项目，您需要激活虚拟环境，它会将正确的目录添加到您的路径中。

将虚拟环境与`pip list`、`pip freeze`和`requirements.txt`文件一起使用，为您提供了一个可重现的环境来运行您的代码。其他需要运行您的代码的人可以使用您生成的`requirements.txt`文件来创建一个匹配的环境。

默认情况下，虚拟环境不会使用任何系统安装的模块，或者安装在您的主目录下的模块。从隔离的角度来看，为了创建可重复的环境，这通常被认为是正确的行为。然而，您可以通过使用参数`--system-site-packages`来改变这一点。

### 我应该用`venv`还是`virtualenv`还是别的什么？

当您从红帽软件集合安装 Python 3 时，`venv`、`virtualenv`和`pip`将被安装，因此您可以准备安装您选择的任何模块。当前 Python 文档中的“安装 Python 模块”是这样说的:

*   `venv`是创建虚拟环境的标准工具，从 Python 3.3 开始就是 Python 的一部分。
*   `virtualenv`是`venv`的第三方替代品(和前身)。它允许在 Python 3.4 之前的版本上使用虚拟环境，这些版本要么根本不提供`venv`，要么不能自动将`pip`安装到创建的环境中。

所以对于 Python 3 的所有近期版本来说，`venv` **是首选**。

如果你使用 Python 2.7，你将需要使用`virtualenv`。

创建虚拟环境的命令仅在使用的模块名称上有所不同。创建后，激活虚拟环境的命令是相同的。

注意:对于`virtualenv`，建议使用`python3.6 -m virtualenv`而不是`virtualenv`命令。参见下面的*避免使用 Python 包装脚本*了解更多信息。

### 使用`venv`创建并激活虚拟环境

如果您还没有这样做，请启用`rh-python36`收藏:

```
$ scl enable rh-python36 bash
```

现在创建虚拟环境。为了避免任何意外，请使用运行 Python 的显式版本号:

```
$ python3.6 -m venv myproject1
```

每当您需要激活虚拟环境时，运行以下命令。

```
$ source myproject1/bin/activate
```

注意:一旦你激活了一个虚拟环境，你的提示将会改变，提醒你你正在一个虚拟环境中工作。示例:

```
(myproject1) $
```

注意:当您再次登录或开始新的会话时，您将需要再次使用`source`命令激活虚拟环境。注意:在激活虚拟环境之前，您应该已经运行了`scl enable`。

更多信息，请参见 docs.python.org[的](http://docs.python.org) [Python 3 教程](https://docs.python.org/3/tutorial/)中的[虚拟环境和包](https://docs.python.org/3/tutorial/venv.html)。

### 使用`virtualenv`创建并激活虚拟环境

如果您还没有这样做，请启用`rh-python36`收藏:

```
$ scl enable rh-python36 bash
```

现在创建虚拟环境。为了避免任何意外，请使用运行 Python 的显式版本号:

```
$ python3.6 -m virtualenv myproject1
```

每当您需要激活虚拟环境时，运行以下命令。注意:在激活虚拟环境之前，您应该已经运行了`scl enable`。

```
$ source myproject1/bin/activate
```

注意:一旦你激活了一个虚拟环境，你的提示将会改变，提醒你你正在一个虚拟环境中工作。示例:

```
(myproject1) $
```

注意:当您再次登录或开始新的会话时，您将需要再次使用`source`命令激活虚拟环境。注意:在激活虚拟环境之前，您应该已经运行了`scl enable`。

更多信息，请参见 [*Python 打包用户指南*](https://packaging.python.org/) 中的 *[使用 pip 和 virtualenv](https://packaging.python.org/guides/installing-using-pip-and-virtualenv/) 安装软件包。*

### 使用`pipenv`管理应用依赖关系

来自 [*Python 打包用户指南*](https://packaging.python.org/) 教程、 *[管理应用依赖](https://packaging.python.org/tutorials/managing-dependencies/)* :

["Pipenv](https://packaging.python.org/key_projects/#pipenv) 是 Python 项目的依赖管理器。如果你熟悉 Node.js 的 npm 或者 Ruby 的 [bundler](http://bundler.io/) ，它在精神上和那些工具是相似的。虽然单独的 [pip](https://packaging.python.org/key_projects/#pip) 对于个人使用来说已经足够了，但是 Pipenv 还是推荐用于协作项目，因为它是一个更高级的工具，可以简化常见用例的依赖管理。"

使用 pipenv，您不再需要单独使用`pip`和`virtualenv`。`pipenv`目前不是标准 Python 3 库或 Red Hat 软件集合的一部分。可以用`pip`安装。(注:参见下面关于不运行`pip install`为`root`的建议。)由于`pipenv`使用`virtualenv`来管理环境，您应该在没有激活任何虚拟环境的情况下安装`pipenv` **。但是，不要忘记首先启用 Python 3 软件集合。**

```
$ scl enable rh-python36 bash # if you haven't already done so
$ python3.6 -m pip install --user pipenv
```

用`pipenv`创建和使用隔离环境的工作方式与`venv`或`virtualenv`稍有不同。当您安装第一个软件包时，如果当前目录中没有`Pipfile`，将自动创建一个虚拟环境。然而，用您想要使用的特定 Python 版本显式地创建一个环境是一个很好的实践。

```
$ scl enable rh-python36 bash # if you haven't already done so 
$ mkdir -p ~/pydev/myproject2
$ cd ~/pydev/myproject2
$ pipenv --python 3.6
$ pipenv install requests
```

要激活 Pipenv 环境，请进入该目录并运行`pipenv shell`。

```
$ scl enable rh-python36 bash # if you haven't already done so 
$ cd ~/pydev/myproject2
$ pipenv shell
```

Pipenv 与`scl enable`的相似之处在于，它不试图用`source`修改当前环境，而是启动一个新的 shell。`exit`去激活，外壳。您还可以使用`pipenv run command`在 pipenv 环境中运行一个命令。

有关更多信息，请参见:

*   *[管理应用依赖](https://packaging.python.org/tutorials/managing-dependencies/) [*中的*](https://packaging.python.org/)*Python 打包用户指南
*   在 Pipenv.org[的](http://pipenv.org/)[文件](https://docs.pipenv.org/)
*   *[Pipenv 和虚拟环境](https://docs.python-guide.org/dev/virtualenvs/)* 在[Python](https://docs.python-guide.org/)网站的搭便车指南

* * *

## 使用 Python 的一般技巧

### `python`命令:通过使用版本号来避免意外

为了避免意外，不要输入`python`。在命令中使用明确的版本号，比如`python3.6`或`python2.7`。

至少，总是使用`python3`或`python2`。如果您正在阅读本文，那么您的系统上已经安装了不止一个版本的 Python。根据您的路径，您可能会得到不同的版本。激活和停用虚拟环境，以及启用软件集合，会改变您的路径，因此很容易混淆键入`python`将获得什么版本。

任何 Python 实用程序，如`pip`或`pydoc`，都会出现同样的问题。建议使用版本号，例如`pip3.6`。至少使用主版本号:`pip3`。请参阅下一节，了解更可靠的替代方案。

### 以`#!/usr/bin/env python`开头的脚本可能会中断

多年来，建议是以`#!/usr/bin/env python`开始脚本，以避免脚本中出现像`/usr/bin`或`/usr/local/bin`这样的硬编码路径。这个构造将搜索您的路径来找到 Python。启用软件集合和/或激活虚拟环境可以改变您的道路。因此，当您的路径改变时，以这个结构开始的 Python 2 脚本可能会突然中断。随着虚拟环境使用的增加，最好不再使用这种结构，因为您可能会得到带有不同模块的不同 Python 安装。

### 使用`which`来确定将运行哪个 Python 版本

使用`which`命令确定键入命令时将使用的完整路径。这将帮助你理解哪个版本的`python`首先出现在你的路径中，当你键入`python`时，它将开始运行。

示例:

```
$ which python # before scl enable
/usr/bin/python

$ scl enable rh-python36 bash

$ which python
/opt/rh/rh-python36/root/usr/bin/python

$ source ~/pydev/myproject1/bin/activate

(myproject1) $ which python
~/pydev/myproject1/bin/python
```

### 避免 Python 包装脚本，如`virtualenv`:使用模块名

一些 Python 实用程序作为包装脚本放在您的路径中的一个`.../bin`目录中。这很方便，因为你只需键入`pip`或`virtualenv.`大多数 Python 实用程序实际上只是带有包装脚本的 Python 模块，用于启动 Python 并运行模块中的代码。

包装器脚本的问题与键入`python`时发生的歧义相同。当您键入不带版本号的命令时，您将获得哪个版本的`pip`或`virtualenv`？为了正常工作，还有一个额外的复杂性，即该实用程序需要与您打算使用的 Python 版本相匹配。如果您无意中混合了不同的版本，可能会出现一些微妙的(难以诊断的)问题。

注意:包装脚本可以驻留在几个目录中。您获得的版本取决于您的路径，当您启用软件集合和/或激活虚拟环境时，路径会发生变化。用`pip --user`安装的模块将它们的包装脚本放在`~/.local/bin`中，激活软件集合或虚拟环境会使其变得模糊。

通过使用`-m` modulename 直接从特定版本的 Python 运行模块，可以避免路径问题带来的意外。虽然这需要更多的输入，但这是一种更安全的方法。

建议:

*   不用`pip`，用`python3.6 -m pip`。
*   不用`pyvenv`，用`python3.6 -m venv`。
*   不用`virtualenv`，用`python3.6 -m virtualenv`。

### 不要以 root 身份运行`pip install`(或与`sudo`一起运行)

直接或者通过使用`sudo`以根用户身份运行`pip install`是一个坏主意，并且**会在某个时候给你带来问题**。您可能会遇到的一些问题包括:

*   RPM 包和`pip`安装包之间的冲突。当你需要安装一个固定的或升级的软件包或模块时，这些冲突很可能会出现。安装可能会失败，或者更糟的是，您可能会以安装失败而告终。最好让`yum`成为系统目录中文件的独占管理器。
*   不容易复制的运行时环境。很难确定哪些模块是通过 RPM 包或`pip`安装的。当您想在另一个系统上运行您的 Python 代码时，需要安装什么？需要在系统范围内安装吗？您会得到与测试代码时相同版本的模块吗？
*   升级模块来解决一个依赖关系可能会破坏其他一些代码。不幸的是，在许多情况下，代码需要模块的特定版本，而较新的版本可能不兼容。将`pip install`作为`root`运行意味着所有模块都被安装在系统范围的目录中，这使得很难确定哪些模块是为特定应用程序安装的。

使用虚拟环境将允许您将为每个项目安装的模块与来自 Red Hat 的 Python 安装的模块隔离开来。使用虚拟环境被认为是创建隔离环境的最佳实践，这种隔离环境提供了特定用途所需的依赖关系。在虚拟环境中运行`pip`时，您不需要使用`--user`,因为它将默认安装在虚拟环境中，您应该对其具有写权限。

如果您没有使用虚拟环境，或者需要一个在虚拟环境之外可用的模块/工具，使用`pip --user`在您的主目录下安装模块。

如果你认为这过于可怕，看看这个 xkcd 漫画。不要忘记悬停，这样你就可以看到替代文本。

### 使用虚拟环境代替`pip --user`

有些指南推荐使用`pip --user`。虽然这比以`root`的身份运行`pip`更可取，但是使用虚拟环境是更好的实践，可以适当地隔离给定项目或一组项目所需的模块。`pip --user`安装使用`~/.local`，它可以通过启用软件集合和/或激活虚拟环境来隐藏。对于在`~/.local/bin`中安装包装脚本的模块，这会导致包装脚本和模块之间的不匹配。

此建议的例外是您需要在虚拟环境之外使用的模块和工具。首要的例子是`pipenv`。你应该用`pip install --user pipenv`来安装`pipenv`。这样，您就可以在没有任何虚拟环境的情况下拥有`pipenv`。

### 不要在自己的项目中使用系统 Python

安装在`/usr/bin/python`和`/usr/bin/python2`中的 Python 版本是操作系统的一部分。RHEL 在一个特定的 Python 版本(2.7.5)中进行了测试，该版本将在该操作系统支持的整个十年生命周期内保持不变。许多内置的管理工具实际上是用 Python 编写的。试图在`/usr/bin`中改变 Python 的版本可能会破坏一些操作系统的功能。

有时，您可能希望在不同版本的操作系统上运行您的代码。该操作系统可能会安装不同版本的 Python，如`/usr/bin/python`、`/usr/bin/python2`，甚至是`/usr/bin/python3`。您编写的代码可能依赖于某个特定的版本，该版本最好通过虚拟环境和/或软件集合来管理。

上述情况的一个例外是，如果您正在编写系统管理工具。在这种情况下，您应该在`/usr/bin`中使用 Python，因为它为操作系统中的 API 安装了正确的模块和库。注意:如果您正在用 Python 编写系统管理或管理工具，您可能想看看 Ansible。Ansible 是用 Python 写的，用 Jinja2 做模板，为很多系统任务提供了更高层的抽象。

提示:如果您需要使用 Python 2.7，请安装`python27`软件集合。遵循上述安装步骤，但使用`python27`代替`rh-python36`。您可以同时启用这两个集合，这样您就可以同时拥有更新的`python2.7`和`python3.6`。注意:您最后启用的集合将是您路径中的第一个集合，它决定了当您键入一个没有明确版本号的命令(如`python`或`pip`)时所获得的版本。

### 不要改变或覆盖`/usr/bin/python`、`/usr/bin/python2`或`/usr/bin/python2.7`

如上所述，系统 Python 是 Red Hat Enterprise Linux 7 的一部分，由关键系统实用程序如`yum`使用。(对，yum 是用 Python 写的。)所以覆盖系统 Python 很可能会破坏你的系统——非常严重。如果你试图从源代码编译 Python，不要在没有使用不同前缀的情况下使用`make install`(作为根用户)，否则它会覆盖`/usr/bin/python`。

* * *

## 软件收藏提示

### 在虚拟环境之前启用 Python 集合

在使用任何 Python 虚拟环境实用程序创建或激活环境之前，您应该始终启用 Python 软件集合**。为了正常工作，您需要在您的路径中有您想要的 Python 版本，因为 Python 虚拟环境将需要它。如果您尝试以错误的顺序启用/激活，会出现许多问题，其中一些问题很微妙。**

 ``venv`的例子:

```
$ scl enable rh-python36 bash
$ python3.6 -m venv myproject1
$ source myproject1/bin/activate
```

稍后在新外壳中重新激活时:

```
$ scl enable rh-python36 bash
$ source myproject1/bin/activate
```

`virtualenv`的例子:

```
$ scl enable rh-python36 bash
$ python3.6 -m virtualenv myproject1
$ source myproject1/bin/activate
```

稍后在新外壳中重新激活时:

```
$ scl enable rh-python36 bash
$ source myproject1/bin/activate
```

### 如何永久启用软件集合

要将 Python 3 永久添加到您的路径中，您可以为您的特定用户 ID 在“点文件”中添加一个`scl_source`命令。这种方法的好处是每次登录时已经启用了收集。如果您使用的是图形桌面，那么从菜单中启动的所有东西都已经启用了集合。

这种方法有一些注意事项:

*   当你输入不带版本号的`python`，**时，你将得到 Python 3 而不是 Python 2** 。你仍然可以通过输入`python2`或者`python2.7`来获得 Python 2。强烈建议使用明确的版本号。
*   以上适用于`.../bin`中的其他 Python 命令，如`pip`、`pydoc`、`python-config`、`pyvenv`和`virtualenv`。使用版本号以避免意外。
*   **没有** **`scl disable`** **命令**。一切都在环境变量中，所以您可以绕过它，但这将是一个手动过程。但是，您可以启用不同的软件集合，该集合将优先于您的配置文件中的集合。

使用您喜欢的文本编辑器，将下面一行添加到您的`~/.bashrc`:

```
# Add RHSCL Python 3 to my login environment
source scl_source enable rh-python36
```

注意:您还可以在构建脚本的开头添加`scl_source`行，以选择构建所需的 Python。如果您的构建脚本不是作为 shell/bash 脚本编写的，您可以将它包装在一个 shell 脚本中，该脚本包含 source `scl_source`命令，然后运行您的构建脚本。

### 如何在#中使用 RHSCL 的 Python 3！剧本的一行

您可以从软件集合中创建一个使用 Python 的脚本，而不需要首先手动运行`scl enable`。这可以通过使用`/usr/bin/scl enable`作为脚本的解释器来完成:

```
#!/usr/bin/scl enable rh-python36 -- python3
import sys

version = "Python %d.%d" % (sys.version_info.major, sys.version_info.minor)
print("You are running Python",version)
```

注意:您可能会尝试只使用到`.../root/usr/bin/python`的完整路径，而不使用`scl enable`。很多情况下，这是行不通的。该行为取决于特定的软件集合。对于大多数收藏来说，由于`LD_LIBRARY_PATH`没有被正确设置，这个操作会因为共享库错误而失败。`python27`集合没有给出错误，但是它找到了错误的共享库，所以您得到了错误的 Python 版本，这可能会令人惊讶。然而，`rh-python36`可以不设置`LD_LIBRARY_PATH`直接引用，但是它是目前唯一一个这样工作的 Python 集合。不能保证将来的收集会以同样的方式工作。

### 如何查看安装了哪些软件集合

您可以使用命令`scl -l`来查看安装了哪些软件集合。这将显示所有已安装的软件集合，无论它们是否已启用。

```
$ scl -l
python27
rh-python36
```

### 如何辨别哪些软件集合已启用

环境变量`X_SCLS`包含当前启用的软件集合列表。

```
$ echo $X_SCLS
$ for scl in $X_SCLS; do echo $scl; done
rh-python36
python27
```

在脚本中，您可以使用`scl_enabled *collection-name*`来测试特定的集合是否被启用。

### 如何找到红帽软件集合列表，支持多长时间？

参见红帽客户门户网站上的[红帽软件集合产品生命周期](https://access.redhat.com/support/policy/updates/rhscl)。它有一个 Red Hat 软件集合包和支持信息的列表。

您也可以查看[发行说明](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/)以获取最新发布的 Red Hat 软件集合。

### 找到额外的 RPM 包并查看其他可用的版本

您可以使用`yum search`来搜索其他包，并查看其他可用的版本:

要搜索属于`rh-python36`集合的其他包:

```
# yum search rh-python36
```

从 Python 3.4 集合开始，集合和包名都带有前缀`rh-`。所以您可以使用下面的命令来查看所有的`rh-python`包，从而查看哪些集合是可用的。

```
# yum search rh-python
```

注意:要查看 Python 2.7 集合中的可用包，请搜索`python27`。

```
# yum search python27
```

当然，您可以搜索`python`并获得名称或描述中包含`python`的所有可用 RPM 的列表。这将是一个很长的列表，所以最好将输出重定向到一个文件，并使用`grep`或文本编辑器来搜索该文件。以`python-`开头的包(没有版本号)是安装在`/usr/bin`的基础 RHEL Python 2.7.5 包的一部分。

* * *

## 故障排除

### Python:加载共享库时出错

当您尝试运行二进制文件，但找不到它所依赖的共享库时，会出现此错误。通常，当试图从软件集合中运行`python`而没有首先启用它时，会发生这种情况。除了设置`PATH`，`scl enable`还设置了`LD_LIBRARY_PATH`。这会将包含软件集合共享对象的目录添加到库搜索路径中。

要查看修改了哪些环境变量，看一下`/opt/rh/rh-python/enable`。

```
$ cat /opt/rh/rh-python36/enable 
export PATH=/opt/rh/rh-python36/root/usr/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/opt/rh/rh-python36/root/usr/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export MANPATH=/opt/rh/rh-python36/root/usr/share/man:$MANPATH
export PKG_CONFIG_PATH=/opt/rh/rh-python36/root/usr/lib64/pkgconfig${PKG_CONFIG_PATH:+:${PKG_CONFIG_PATH}}
export XDG_DATA_DIRS="/opt/rh/rh-python36/root/usr/share:${XDG_DATA_DIRS:-/usr/local/share:/usr/share}"
```

### 运行`python`时 Python 版本错误

首先，在没有版本号的情况下运行`python`很可能会在某个时候给你一个意想不到的 Python 版本。结果取决于您的`PATH`，这取决于您是否启用了软件集合和/或激活了虚拟环境。如果您使用一个版本号，比如`python3.6`，并且您还没有启用/激活正确的环境，您将得到一个清晰易懂的“命令未找到”错误。

第二，如果你忘记启用软件集合，你也可能得到错误的版本。启用软件集合会首先将集合的`/bin`目录放在您的路径中，因此它会隐藏同名的所有其他版本的命令。

**即使您给出了`python`二进制文件**的完整路径，也需要启用软件集合。对于大多数集合，如果没有正确设置库路径，将会出现共享库错误(见上文)。然而，如果您尝试使用`python27`集合，您将得到 Python 2.7.5(默认版本),而不是您期望的 Python 2.7.13。这是因为共享库依赖是从`/lib`中得到满足的，而不是从软件集合中，所以你选择了系统 Python。

### 运行`pip`时出错:导入错误无法导入名称“main”

如果像一些指南建议的那样运行`pip upgrade --user pip`，那么`pip`命令将不再工作。这个问题是路径问题加上版本之间的不兼容。`pip`的用户安装在`~/.local/bin`中放置了一个新的`pip`命令。然而，`~/.local/bin`在你的路径中*在*软件集合之后。因此，您会得到与新模块不兼容的旧包装脚本。

这可以通过几种方式解决:

*   使用虚拟环境。创建或激活虚拟环境后，您将在虚拟环境的`.../bin`目录中获得正确的`pip`包装脚本。
*   将`pip`作为模块:`python3.6 -m pip install ...`(参见上面的“避免 Python 包装脚本”)。)
*   不要在虚拟环境之外升级`pip`。
*   使用`pip`包装脚本的完整路径:`~/.local/bin/pip3.6`。
*   启用 Python 软件集合后，将`~/.local/bin`添加为您的`PATH`中的第一个目录。

注意:要卸载安装在`~/.local`中的升级版`pip`，请在您的普通用户 ID(不是`root`)下运行以下命令:

```
$ python3.6 -m pip uninstall pip
```

### 找不到`virtualenv3.6`

`rh-python36`软件集合包括`virtualenv`包装脚本，但是没有`virtualenv3.6`的链接。有两个解决方法，但首先我应该指出`venv`现在是 Python 3 虚拟环境的首选工具。

首选的解决方法是完全避免包装脚本，直接调用模块:

```
$ python3.6 -m virtualenv myproject1
```

或者，您可以在您的`~/bin`目录中创建自己的符号链接:

```
$ ln -s /opt/rh/rh-python36/root/usr/bin/virtualenv ~/bin/virtualenv3.6
```

## 

* * *

## 更多信息:在 Red Hat 平台上用 Python 开发

Nick Coghlan 和 Graham Dumpleton 在 DevNation 2016 上发表了一篇关于在红帽平台 上使用 Python 开发 *[的演讲。这个演讲充满了信息，仍然很有意义。其中包括使用容器构建 Python 应用程序、使用 s2i 和部署到 Red Hat OpenShift 的信息。我建议观看视频或者至少](https://developers.redhat.com/videos/youtube/tLTSQiVQ8qk/)[回顾一下幻灯片](https://www.slideshare.net/ncoghlan_dev/developing-in-python-on-red-hat-platforms-devnation-2016)。*

https://www.youtube.com/watch?v=tLTSQiVQ8qk

## 

* * *

## 总结

读完这篇文章后，你学到了:

*   如何在 Red Hat Enterprise Linux 上使用 Red Hat 软件集合安装 Python 3 和 Red Hat 支持的其他 Python 版本
*   Python 虚拟环境是安装 Python 模块的最佳实践，同时隔离依赖性以避免冲突。您可以使用`venv`和`virtualenv`创建和激活虚拟环境。这两个工具都将作为软件集合的一部分为您安装。
*   关于`pipenv`，一个类似于`npm`的工具，Python 打包指南推荐用于管理应用依赖，尤其是在共享项目上。Pipenv 提供了一个集成了`pip`和`virtualenv`的命令。
*   需要避免的事情，例如:
    *   将`pip install`作为`root`运行，以避免与`yum`安装的 RPM 包冲突
    *   键入不带版本号的`python`,以避免运行哪个版本的模糊性以及由此可能导致的意外
    *   修改/usr/bin/python，因为许多系统管理工具如`yum`依赖于它，可能会崩溃
*   使用 Red Hat 软件集合的提示
    *   使用虚拟环境时，始终在之前启用 Python 软件集合
    ***   如何永久地启用一个软件集合，这样您就可以一直拥有 python3 了*   如何在#中使用 RHSCL 中的 Python 3！剧本的一行**
***   如何解决常见问题，例如
    *   Python:加载共享库时出错
    *   `pip upgrade`用:ImportError 中断 pip 无法导入名称“main”
    *   键入`python`时 Python 版本错误**

***Last updated: November 15, 2018***`