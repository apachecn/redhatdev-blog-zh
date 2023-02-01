# Python 轮子、AI/ML 和 ABI 兼容性

> 原文：<https://developers.redhat.com/blog/2019/09/27/python-wheels-ai-ml-and-abi-compatibility>

Python 已经成为 AI/ML 世界中流行的编程语言。像 [TensorFlow](https://www.tensorflow.org/) 和 [PyTorch](https://pytorch.org/) 这样的项目将 Python 绑定作为数据科学家编写机器学习代码的主要接口。然而，分发 AI/ML 相关的 Python 包并确保各种 Python 包和系统库之间的应用程序二进制接口(ABI)兼容性提出了一系列独特的挑战。

用于 [Python wheels](https://opensource.com/article/19/2/manylinux-python-wheels) 的 manylinux 标准(例如 [manylinux2014](https://www.python.org/dev/peps/pep-0599/) )为这些挑战提供了一个实用的解决方案，但它也引入了 Python 社区和开发者需要考虑的新挑战。在我们深入研究这些额外的挑战之前，我们将简要地看一下用于打包和分发的 Python 生态系统。

## 车轮、AI/ML 和 ABIs

Python 包是使用`pip`命令安装的，该命令从 pypi.org 下载包。

```
pip install <package-name>
```

这些包有两种类型:

1.  纯 Python 轮子，可能针对也可能不针对特定的 Python 版本
2.  扩展轮，使用用 C/C++编写的本机代码

所有 AI/ML Python 包都是使用原生操作系统库的扩展轮。在一个发行版上编译的 Python 扩展模块可能无法在其他发行版上运行，甚至无法在安装了不同系统库的运行相同发行版的不同机器上运行。这是因为编译后的二进制文件记录了它们所依赖的 ABI，比如重定位、符号和版本、全局数据符号的大小等等。在运行时，如果 ABI 不匹配，加载程序可能会引发错误。带有版本的缺失符号的示例如下所示:

```
/lib64/libfoo.so.1: version `FOO_1.2' not found (required by ./app)
```

AI/ML 项目维护人员需要为 Windows、macOS X 和 Linux 发行版构建不同的 Python 包。预编译的二进制文件打包成一个 [*轮*](https://www.python.org/dev/peps/pep-0427/) 格式，文件扩展名为`.whl`。轮子是一个可以被解释为 Python 库的 zip 文件。

文件名包含特定的标签，`pip`命令使用这些标签来确定与安装 AI/ML 库的系统相匹配的 Python 版本和操作系统。轮子还包含 Python 项目的布局，因为它应该安装在系统上。为了避免用户需要编译这些包，项目维护人员在 pypi.org 上为 Windows、macOS 和 Linux 构建并上传特定平台的轮子。

以下是一些适用于 Linux 和非 Linux 发行版的轮子示例:

```
tensorflow-2.0.0-cp27-cp27m-macosx_10_11_x86_64.whl
tensorflow-2.0.0-cp35-cp35m-win_amd64.whl
tensorflow-2.0.0-cp36-cp36m-manylinux1_x86_64.whl
tensorflow-2.0.0-cp37-cp37m-manylinux2010_x86_64.whl
```

## Manylinux2014

AI/ML 项目的维护者想要为 Linux 发行版发布带有本机代码的 Python 库，他们面临着确保 ABI 兼容性的艰巨任务。编译后的代码需要在各种各样的 Linux 发行版上运行。

幸运的是，有一种方法可以使二进制文件与大多数(尽管不是全部)Linux 发行版兼容。为此，您需要构建一个二进制文件，并使用比您想要支持的任何发行版都旧的 ABI 基线。预期新的发行版将保持 ABI 保证；这样，只要新发行版提供了 ABI 基线，您就可以在新发行版上运行您的二进制文件。最终，ABI 基线将以不兼容的方式改变，这可能是向前移动基线的技术要求。将 ABI 基线向前移动还有其他非技术需求，它们围绕着发行版生命周期。

manylinux 平台标签是一种让你的 Python 库与大多数 linux 发行版兼容的方法。Python 的 manylinux 定义了一个 ABI 基线，并通过构建旧版本的发行版来瞄准基线。为了实现最大的兼容性，它使用了支持时间最长的免费发布版本的 Linux: [CentOS](https://www.centos.org/) 。

第一个名为 [manylinux1](https://www.python.org/dev/peps/pep-0513) 的 manylinux 平台标签使用 CentOS 5。第二次迭代叫做 [manylinux2010](https://www.python.org/dev/peps/pep-0571/) 使用 CentOS 6。最新的规范 [manylinux2014](https://www.python.org/dev/peps/pep-0599/) 是 Red Hat、其他供应商和 Python 社区将 manylinux 规范向前推进以使用 [CentOS 7/UBI 7](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image) 并支持更多架构的结果。

为了使 AI/ML Python 项目维护人员的生活更加轻松，Python 社区提供了一个预构建的 manylinux 构建容器，可用于构建项目轮，如下所示:

```
centos5 Image - quay.io/pypa/manylinux1_x86_64
centos6 Image - quay.io/pypa/manylinux2010_x86_64
ubi7 Image - quay.io/pypa/manylinux2014_x86_64(coming soon)
```

对于 AI/ML Python 项目用户来说,`pip`命令非常重要。`pip`命令将根据轮子标签和与系统匹配的轮子的 manylinux 平台标签安装合适的轮子文件。例如，manylinux2014 wheel 无法安装在 Red Hat Enterprise Linux (RHEL) 6 上，因为它没有 manylinux2014 规范中指定的系统库版本。Pip 将在 RHEL 6 上安装许多 linux2010 轮子，在 RHEL 7 上安装许多 linux2014 轮子。

AI/ML Python 项目用户必须确保在更新到下一个 AI/ML Python 项目版本之前，定期更新`pip`命令。如果用户正在使用容器，那么容器中应该有最新的`pip`命令。

## 其他挑战

尽管 manylinux 标准有助于提供可靠和稳定的扩展轮，但它也带来了两个额外的挑战:

1.  **生命周期**
    在某个时候，ABI 基线的参考平台将会寿终正寝。Python 社区必须积极跟踪项目使用的不同系统库的生命周期结束支持和 CVE，并潜在地将项目维护人员转移到下一个可用的 manylinux 平台标签。**注意:**CentOS 6 的停产日期为 2020 年 11 月 30 日。CentOS 7 的停产日期为 2024 年 6 月 30 日。
    最后，项目维护人员应该确保他们为所有的 linux 平台标签构建轮子，或者至少是最新规范的轮子。这将为用户提供最多的安装选项。
2.  **硬件厂商支持**
    几乎所有的 AI/ML Python 项目都有某种形式的硬件加速器支持，比如 CUDA(英伟达)、ROCm (AMD)、英特尔 MKL。硬件供应商可能不支持工具链的所有版本，项目维护人员应该选择一个基线工具链(gcc、binutils、glibc ),并将他们的轮子设置为匹配某个 linux 平台标签。一些项目可能需要支持多种架构，包括 Intel/AMD (i686、x86_64)、Arm (aarch64、armhfp)、IBM POWER (ppc64、ppc64le)或 IBM Z 系列(s390x)。不同架构上的回归测试对于发现兼容性问题至关重要。参见红帽企业版 Linux ABI 兼容性指南中的 [RHEL 7](https://access.redhat.com/articles/rhel-abi-compatibility) 和 [RHEL 8](https://access.redhat.com/articles/rhel8-abi-compatibility) 。

### 解决方法

Python 社区必须遵循用于定位 ABI 基线的参考软件的生命周期，并做出相应的计划。Python 开发人员必须仔细地将系统工具或开发人员工具与硬件供应商的软件需求相匹配。解决这两个问题是困难的，但最终是有益的挑战。

*Last updated: July 1, 2020*