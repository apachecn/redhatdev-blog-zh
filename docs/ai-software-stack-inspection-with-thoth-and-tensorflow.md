# 用 Thoth 和 TensorFlow 进行人工智能软件栈检查

> 原文：<https://developers.redhat.com/blog/2020/09/30/ai-software-stack-inspection-with-thoth-and-tensorflow>

Thoth 项目开发了增强开发人员和数据科学家日常生活的开源工具。透特使用机器生成的知识，通过使用[人工智能](https://developers.redhat.com/topics/ai-ml) (AI)到[强化学习(RL)](https://en.wikipedia.org/wiki/Reinforcement_learning#:~:text=Reinforcement%20learning%20(RL)%20is%20an,supervised%20learning%20and%20unsupervised%20learning.) ，来提升你的应用的性能、安全性和质量。这种机器学习方法在 [Thoth adviser](https://github.com/thoth-station/adviser) 中实现(如果您想了解更多信息，[单击此处](https://www.youtube.com/watch?v=WEJ65Rvj3lc&t=1s))，并由 [Thoth integrations](https://github.com/thoth-station/adviser/blob/master/docs/source/integration.rst) 用于根据[用户输入](https://github.com/thoth-station/thamos#using-custom-configuration-file-template)提供软件堆栈。

在本文中，我将介绍一个案例研究——最近在导入 TensorFlow 2.1.0 时对运行时问题的检查——来演示 Thoth 团队和 Thoth 组件之间的人机交互。通过从头到尾跟踪案例研究，您将了解透思如何收集和分析一些数据，以向其用户提供[建议](https://thoth-station.ninja/justifications)，包括机器人，如 [Kebechet](https://github.com/thoth-station/kebechet#kebechet) 、[人工智能支持的持续集成管道](https://github.com/AICoE/aicoe-ci)，以及使用 [GitHub 应用](https://github.com/thoth-station/qeb-hwt)的开发人员。

Thoth machinery 和团队都依赖于运行在 Red Hat OpenShift 上的机器人和自动化管道。透特采纳了各种意见来确定正确的建议:

*   [Solver](https://github.com/thoth-station/solver) ，Thoth 用它来发现某些东西是否可以安装在特定的运行时环境中，比如[Red Hat Enterprise Linux](https://developers.redhat.com/topics/linux)(RHEL)8 with Python 3.6。
*   [安全指标](https://github.com/thoth-station/datasets/tree/master/notebooks/thoth-security-dataset#thoth-security-datasets)揭示不同性质的漏洞，可应用于安全建议。
*   [项目元信息](https://github.com/thoth-station/mi)，如影响整个项目的项目维护状态或开发过程行为。
*   检查，Thoth 使用它来发现跨包的代码质量问题或性能。

本文主要关注检查。我将向您展示通过[项目 Thoth](https://thoth-station.ninja/) 的[依赖猴子](https://github.com/thoth-station/adviser/blob/master/docs/source/dependency_monkey.rst)和 [Amun](https://github.com/thoth-station/amun-api#amun-service) 组件运行的自动化软件堆栈检查的结果。Thoth 使用自动化检查为 Thoth 用户引入关于软件栈的新建议。整合建议的另一种方式是通过自动化管道，它可以:

*   提升性能
*   优化[机器学习](https://developers.redhat.com/blog/category/machine-learning/) (ML)模型推理
*   确保在模型运行期间(例如，在推理期间)没有失败
*   避免使用不能保证安全性的软件栈。

## 透特组件:阿蒙和依赖猴子

给定应该安装的软件包列表和运行应用程序所需的硬件， [Amun](https://github.com/thoth-station/amun-api#amun-service) 在所请求的环境中执行所请求的应用程序堆栈。阿蒙是透特的执行引擎。然后使用[透思性能指标(PI)](https://github.com/thoth-station/performance) 构建并测试应用程序。参见 Amun 的[自述文件](https://github.com/thoth-station/amun-api/blob/master/README.rst)了解更多关于这项服务的信息。

另一个 Thoth 组件， [Dependency Monkey](https://github.com/thoth-station/adviser/blob/master/docs/source/dependency_monkey.rst) ，可以用来调度 Amun。Dependency Monkey 旨在自动评估软件堆栈的某些方面，如代码质量或性能。因此，它的目标是自动验证软件栈并聚集相关的观察结果。

从这两个组件中，Thoth 团队创建了 [Thoth 性能数据集](https://github.com/thoth-station/datasets/blob/master/notebooks/thoth-performance-dataset)，其中包含了对软件栈性能的观察。例如，Thoth 性能数据集可以使用 [PIconv2d](https://github.com/thoth-station/performance/blob/master/tensorflow/conv2d.py) 来获得不同应用类型(如机器学习)和代码质量的性能数据。然后，它可以使用像 [PiImport](https://github.com/thoth-station/performance/blob/master/tensorflow/import.py) 这样的性能指标来发现应用程序运行过程中的错误。

## 透明和可再现的数据集

本着开源的精神，Thoth 团队希望保证我们收集和使用的数据集和知识是透明和可复制的。机器学习模型，比如由 [Thoth Adviser](https://github.com/thoth-station/adviser) 利用的强化学习模型，应该和它们正在处理的数据集一样透明。

为了透明，我们引入了 [Thoth Datasets](https://github.com/thoth-station/datasets#thoth-datasets) ，在这里我们共享我们用来分析数据集合和所有结果的笔记本。我们鼓励任何对这个话题感兴趣的人使用 Thoth 数据集来验证我们的发现或用于其他目的。

为了再现性，我们引入了[依赖猴子动物园](https://github.com/thoth-station/dependency-monkey-zoo)，在那里我们收集了用于运行分析的所有规范。将所有规格放在一个地方让我们能够重现研究结果。任何人都可以使用这些规格在不同的环境中进行类似的研究，以便进行比较。

## 案例研究:TensorFlow 2.1.0 的自动化软件堆栈检查

对于这个案例研究，我们将使用 Thoth 的 Amun 和 Dependency Monkey 组件来自动生成数据。然后，我们将介绍可重用的 Jupyter 笔记本模板，以从数据集中提取特定的信息。最后，我们将根据结果创建新的建议。

这种人机交互的人的方面侧重于评估结果的质量和制定建议。剩下的过程是机器自动化的。自动化使该过程易于重复，从而产生新的分析信息源。

在接下来的部分中，我将介绍最初的问题，然后描述所执行的分析以及由此产生的对 Thoth 用户的新建议。

## 初始请求

我们此次检查的目标是分析导入 TensorFlow 2.1.0 时的构建和运行时故障，并使用这些故障得出关于软件堆栈质量的观察结果。

对于这个分析，Dependency Monkey 采样了所有可能的`TensorFlow==2.1.0`栈的状态空间(来自上游构建)。出于检查的目的，我们使用 [PiMatmul](https://github.com/thoth-station/performance/blob/master/tensorflow/matmul.py) 性能指示器构建并运行了应用程序。

下面的部分详细描述了[依赖性猴子检查结果](https://github.com/thoth-station/dependency-monkey-zoo/tree/master/tensorflow/inspection-2020-09-04)和由此产生的[分析](https://github.com/thoth-station/notebooks/issues/70)。

## 第一个分析

从检查结果的[软件堆栈分析](https://github.com/thoth-station/datasets/blob/master/notebooks/thoth-performance-dataset/PerformanceTensorFlow2.1.0SoftwareStackCombinations.ipynb)中，我们发现 TensorFlow 2.1.0 在一次运行中大约 50%的检查中出现错误。该错误显示在 Jupyter 笔记本的以下输出中:

```
'2020-09-05 07:14:36.333589: W tensorflow/stream_executor/platform/default/dso_loader.cc:55] Could not load dynamic library \'libnvinfer.so.6\'; dlerror: libnvinfer.so.6: cannot open shared object file: No such file or directory
2020-09-05 07:14:36.333811: W tensorflow/stream_executor/platform/default/dso_loader.cc:55] Could not load dynamic library \'libnvinfer_plugin.so.6\'; dlerror: libnvinfer_plugin.so.6: cannot open shared object file: No such file or directory
2020-09-05 07:14:36.333844: W tensorflow/compiler/tf2tensorrt/utils/py_utils.cc:30] Cannot dlopen some TensorRT libraries. If you would like to use Nvidia GPU with TensorRT, please make sure the missing libraries mentioned above are installed properly.
/opt/app-root/lib/python3.6/site-packages/h5py/__init__.py:36: FutureWarning: Conversion of the second argument of issubdtype from `float` to `np.floating` is deprecated. In future, it will be treated as `np.float64 == np.dtype(float).type`.
from ._conv import register_converters as _register_converters
/opt/app-root/lib/python3.6/site-packages/requests/__init__.py:91: RequestsDependencyWarning: urllib3 (1.5) or chardet (2.3.0) doesn\'t match a supported version!
RequestsDependencyWarning)
Traceback (most recent call last):
 File "/home/amun/script", line 14, in <module>
  import tensorflow as tf
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow/__init__.py", line 101, in <module>
from tensorflow_core import *
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow_core/__init__.py", line 40, in <module>
from tensorflow.python.tools import module_util as _module_util
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow/__init__.py", line 50, in __getattr__
module = self._load()
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow/__init__.py", line 44, in _load\n    module = _importlib.import_module(self.__name__)
File "/opt/app-root/lib64/python3.6/importlib/__init__.py", line 126, in import_module
return _bootstrap._gcd_import(name[level:], package, level)
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow_core/python/__init__.py", line 95, in <module>
from tensorflow.python import keras
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow_core/python/keras/__init__.py", line 27, in <module>
from tensorflow.python.keras import models
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow_core/python/keras/__init__.py", line 27, in <module>
from tensorflow.python.keras import models
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow_core/python/keras/models.py", line 25, in <module>
from tensorflow.python.keras.engine import network
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow_core/python/keras/engine/network.py", line 46, in <module>
from tensorflow.python.keras.saving import hdf5_format
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow_core/python/keras/saving/hdf5_format.py", line 32, in <module>
from tensorflow.python.keras.utils import conv_utils
  File "/opt/app-root/lib/python3.6/site-packages/tensorflow_core/python/keras/utils/conv_utils.py", line 22, in <module>
from six.moves import range  # pylint: disable=redefined-builtin
ImportError: cannot import name \'range\''
```

具体来说，我们可以看到一些`six`和`urllib3`的组合产生了错误，如下面的[输出](https://github.com/thoth-station/notebooks/issues/70#issuecomment-688656575)所示:

```
=============================================
urllib3
=============================================

In successfull inspections:
['urllib3-1.10.4-pypi-org' 'urllib3-1.16-pypi-org' 'urllib3-0.3-pypi-org'
'urllib3-1.21.1-pypi-org' 'urllib3-1.25.1-pypi-org'
'urllib3-1.25-pypi-org' 'urllib3-1.18.1-pypi-org'
'urllib3-1.24.1-pypi-org' 'urllib3-1.10.1-pypi-org'
'urllib3-1.10.3-pypi-org' 'urllib3-1.25.7-pypi-org'
'urllib3-1.10-pypi-org' 'urllib3-1.7.1-pypi-org' 'urllib3-1.13-pypi-org'
'urllib3-1.19.1-pypi-org' 'urllib3-1.11-pypi-org'
'urllib3-1.10.2-pypi-org' 'urllib3-1.15.1-pypi-org'
'urllib3-1.25.3-pypi-org' 'urllib3-1.13.1-pypi-org'
'urllib3-1.21-pypi-org' 'urllib3-1.17-pypi-org' 'urllib3-1.23-pypi-org']

In failed inspections:
['urllib3-1.5-pypi-org']

In failed inspections but not in successfull:
{'urllib3-1.5-pypi-org'}

In failed inspections and in successfull:
set()

=============================================
six
=============================================

In successfull inspections:
['six-1.13.0-pypi-org' 'six-1.12.0-pypi-org']

In failed inspections:
['six-1.13.0-pypi-org' 'six-1.12.0-pypi-org']

In failed inspections but not in successfull:
set()

In failed inspections and in successfull:
{'six-1.13.0-pypi-org', 'six-1.12.0-pypi-org'}
```

因此，我们发现 [**urllib3**](https://pypi.org/project/urllib3/) 库版本在所有失败的检查中都是相同的，但在任何成功的检查中都不是，而 [**六个**](https://pypi.org/project/six/) 库版本在一次失败和成功之间没有显示任何差异。

## 第二个分析

对于我们的下一步，我们决定运行另一个分析来限制案例。对于这次运行，我们使用了一个新创建的名为 [PiImport](https://github.com/thoth-station/performance/blob/master/tensorflow/import.py) 的性能指标，如表 1 所示。

Table 1: The PiImport performance indicator.

| **描述** | Dependency Monkey 采样了所有可能的`TensorFlow==2.1.0`栈的状态空间(来自上游构建)。应用程序是使用 [PiImport](https://github.com/thoth-station/performance/blob/master/tensorflow/import.py) 性能指示器构建和运行的。 |
| **规格** | [依赖猴子规范](https://github.com/thoth-station/dependency-monkey-zoo/tree/master/tensorflow/inspection-2020-09-08.1) |
| **目标** | 确定未能产生最终[建议](https://github.com/thoth-station/adviser/pull/1172)的具体版本。 |
| **参考** | [发布](https://github.com/thoth-station/datasets/issues/16) |

## 第二次分析的结果

根据新的[分析](https://github.com/thoth-station/datasets/blob/master/notebooks/thoth-performance-dataset/PerformanceTensorFlow2.1.0SoftwareStackCombinationsErrors.ipynb)，我们能够识别出`urllib3`和`six`的所有特定版本，它们不能一起工作，并在运行时导致问题。图 1 中的输出显示了这两个包的不兼容版本。

d 图 1:识别不允许运行 Tensorflow 2.1.0 的 urllib3 和 six 的不兼容版本。

## 建议

所有这些回溯导致了一个叫做 **TensorFlow21Urllib3Step** 的[顾问步骤](https://github.com/thoth-station/adviser/pull/1172)，你可以在[顾问步骤](https://github.com/thoth-station/adviser)中找到它。通过这一步，我们可以惩罚包含特定版本`urllib3`的软件堆栈，这些软件堆栈在尝试导入 TensorFlow 2.1.0 时会导致运行时问题。下面这个由 Thoth 创造的预测，为用户带来了更高质量的软件栈。

Table 2: The TensorFlow21Urllib3Step adviser step.

| **Title** | 2.1 版本中的 TensorFlow 在导入时会导致运行时错误，这是由于`urllib3`和`six`包不兼容造成的。 |
| **问题描述** | 某些版本的包`urllib3`附带了一个捆绑版本的`six`，它有自己的导入和导入上下文处理机制。在 TensorFlow 代码库中导入`urllib3`会导致捆绑的`six`模块的初始化，这会与后续从非捆绑的`six`模块导入发生冲突。 |

您可以在这里找到完整的问题描述和建议的解决方案。

*Last updated: June 20, 2022*