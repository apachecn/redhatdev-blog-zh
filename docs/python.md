# 你好世界 Hello 上的 Python

> 原文：<https://developers.redhat.com/rhel8/hw/python>

## RHEL 8 setup

这个 *Hello，World* 展示了如何在 Red Hat Enterprise Linux 8 上安装和运行一个包。如果你还没有，[下载](https://developers.redhat.com/products/rhel/download)并安装 RHEL 8，并向红帽订阅管理注册。如果您还没有订阅，当您通过 developers.redhat.com[下载时，将会为您创建一个免费的开发者订阅。](https://developers.redhat.com/)

确保安装了核心的 Red Hat Enterprise Linux 8 开发工具(Make、git、gcc)。如果您在安装期间没有选择开发工具，请现在安装它们:

```
$ sudo yum groupinstall ‘Development Tools’

```

注意:如果您的用户 ID 没有启用`sudo`，请参见[如何在 Red Hat Enterprise Linux 上启用 sudo](https://developers.redhat.com/blog/2018/08/15/how-to-enable-sudo-on-rhel/)。在系统安装过程中，勾选*框使该用户成为管理员*会为您的用户 ID 启用`sudo`。

## 安装 Python

首先，查看哪些 Python 模块可作为应用程序流存储库的一部分:

```
$ sudo yum module list | grep -i python
python27       2.7 [d] default [d]                    Python programming language, version 2.7
python36       3.6 [d][e] build, default [d]          Python programming language, version 3.6

```

对于“python36”模块，有两个配置文件:“默认”和“构建”。对于开发，使用“构建”配置文件。这将导致安装“python36-devel”包，构建任何使用动态加载代码(如 C/C++)的 python 模块都需要该包。

安装 Python 3.6:

```
$ sudo yum module install python36/build
```

Python 现已安装。有关上述命令的说明，请参见下面的使用应用程序流。

检查 Python 版本和路径:

```
$ python3 -V
Python 3.6.7

```

```
$ which python3
/usr/bin/python3
```

您还可以使用“python3.6”作为命令来专门运行 Python 3.6。

安装了用于安装模块和使用 Python 虚拟环境的 Python 模块“pip”和“venv”。“/usr/bin”中有运行这些的包装器脚本，但是强烈建议在不使用包装器脚本的情况下将它们作为模块运行。如果您安装了多个版本的 Python，这有助于避免冲突和其他意外。

```
$ python3 -m pip …

```

```
$ python3 -m venv ...
```

使用以下命令可以获得有关您的系统上安装了什么的更多信息:

```
$ ls /usr/bin/py* /usr/bin/pip3*  # see what’s in /usr/bin
$ rpm -qil python36    # get the info and list of files in the python36 meta package
$ python3 -m pip list    # installed modules
$ yum list installed python\*    # list installed packages matching python*

```

注意:默认情况下，没有没有版本号的`/usr/bin/python`。强烈建议使用特定的版本，如“python3”或更好的“python3.6”。这可以避免意外，并且是最佳实践。

虽然您可以安装一个包，让`/usr/bin/python `让您选择 Python 3 或 Python 2，但不推荐这样做。在某种程度上，在没有指定版本的情况下，对 Python 版本的期望是错误的。

更多信息，请参见[RHEL 8 上的 Python(链接待定)。

# 你好，世界

让我们创建一个可以从命令行运行的简单 Python 程序。使用 vi、nano 或 gedit 等文本编辑器，创建一个名为“hello.py”的文件，其内容如下:

```
hello.py
```

#!/usr/bin/Python 3.6 import sys version = " Python % d . % d " %(sys . version _ info . major，sys.version_info.minor) print("你好，红帽开发者世界 from "，version)

保存它并退出编辑器。然后，使程序可执行，并运行它:

```
$ chmod +x hello.py
$ ./hello.py
Hello, Red Hat Developer World from Python 3.6

```

## 使用 appstream

第一步是查看应用程序流(appstream)报告中有哪些模块可用:

```
$ sudo yum module list  # list all available modules in appstream

```

或者，只查找名为“nodejs”的模块

```
$ sudo yum module list nodejs

```

从输出中可以看到 Node.js 10 是要安装的默认模块，注意`[d]`。您可以简单地输入以下命令来安装默认的 nodejs 模块。

```
$ sudo yum module install nodejs

```

甚至用“@”来缩短:

```
$ sudo yum install @nodejs

```

上面的命令已经用默认的概要文件安装了 nodejs。概要文件是模块中的一组包，通常是它们的子集。对于该模块，默认配置文件被命名为“default”。在上面的步骤中，选择了“development”概要文件来安装开发概要文件中的包。

要了解有关模块的更多信息，请使用以下命令之一:

```
$ yum module info nodejs  # get info about the default nodejs module

```

```
$ yum module info nodejs:10   # get info about a specific module stream

```

*Last updated: November 19, 2020*