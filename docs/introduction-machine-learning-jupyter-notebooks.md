# Jupyter 笔记本机器学习简介

> 原文：<https://developers.redhat.com/articles/2021/05/21/introduction-machine-learning-jupyter-notebooks>

最近，我在做一个边缘计算演示，它使用[机器学习](/blog/category/machine-learning/) (ML)来检测制造现场的异常。该演示是去年宣布的 [AI/ML 工业边缘解决方案蓝图](https://www.openshift.com/blog/now-available-ai/ml-industrial-edge-solution-blueprint-v1.0)的一部分。正如 GitHub 上的[文档中所述，该蓝图支持声明性规范，这些规范可以分层组织，并定义边缘参考架构中使用的所有组件，如硬件、软件、管理工具和工具。](https://github.com/redhat-edge-computing/industrial-edge-docs)

在项目开始时，我对机器学习只有一个大致的了解，缺乏从业者的知识来用它做一些有用的事情。同样，我听说过 Jupyter 笔记本，但不知道它们是什么，也不知道如何使用。

这篇文章是面向那些想要理解机器学习以及如何用 Jupyter 笔记本来实现它的开发人员的。您将通过构建一个机器学习模型来检测工厂使用的泵的振动数据中的异常，从而了解 Jupyter 笔记本电脑。一个[示例笔记本](https://github.com/sa-mw-dach/manuela-dev/blob/master/ml-models/anomaly-detection/Anomaly-Detection-simple-ML-Training.ipynb)将用于解释笔记本概念和工作流程。如果你想学习如何建立 ML 模型，有很多很好的资源可以利用。

## 什么是 Jupyter 笔记本？

计算笔记本已经被用作记录程序、数据、计算和发现的电子实验室笔记本。Jupyter 笔记本为开发数据科学应用程序提供了一个交互式计算环境。

Jupyter 笔记本将软件代码、计算输出、说明性文本和丰富的内容结合在一个文档中。笔记本允许在浏览器中编辑和执行代码，并显示计算结果。笔记本以扩展名`.ipynb`保存。Jupyter Notebook 项目支持几十种编程语言，它的名字反映了对 Julia (Ju)、Python (Py)和 r。

你可以通过使用公共的沙箱或者像 T2 JupyterHub T3 一样启用你自己的服务器来试用笔记本。JupyterHub 为多个用户提供笔记本。它产生、管理和代理单用户 Jupyter 笔记本服务器的多个实例。在本文中，JupyterHub 将在 [Kubernetes](/topics/kubernetes) 上运行。

## Jupyter 笔记本仪表盘

当笔记本服务器首次启动时，它会打开一个新的浏览器选项卡，显示笔记本仪表板。仪表板是您笔记本的主页。它的主要目的是显示用户可访问的文件系统部分，并提供正在运行的内核、终端和并行集群的概述。图 1 显示了一个笔记本仪表板。

[![A screenshot of the Jupyter notebook dashboard.](img/86b55f327fc19c9246460091a41299c3.png)](/sites/default/files/blog/2021/03/Jupyter_dashboard.png)Figure 1: A notebook dashboard.

Figure 1: A notebook dashboard.

以下部分描述了笔记本仪表板的组件。

### 文件选项卡

**文件**选项卡提供了用户可访问的文件系统视图。该视图通常以笔记本服务器启动的目录为根。

### 添加笔记本

点击**新建**按钮可以创建新的笔记本，点击**上传**按钮可以上传新的笔记本。

### 运行标签

**运行**选项卡显示服务器已知的当前正在运行的笔记本。

## 使用 Jupyter 笔记本

打开笔记本时，会创建一个新的浏览器选项卡来显示笔记本的用户界面。以下部分描述了该界面的组件。

### 页眉

在笔记本文档的顶部是一个包含笔记本标题的标题、一个菜单栏和一个工具栏，如图 2 所示。

[![A screenshot of the Jupyter header.](img/ad31b40feaa9d52d2709a6ff2d1b2606.png)](/sites/default/files/blog/2021/03/Jupyter-header.png)Figure 2: A notebook header.

Figure 2: Notebook header.

### 身体

笔记本的主体是由细胞组成的。单元格可以按任何顺序包含，并可以随意编辑。单元格内容分为以下几种类型:

*   降价单元格:这些单元格包含降价格式的文本，解释代码或包含其他富媒体内容。
*   代码单元:这些包含可执行代码。
*   原始单元格:当文本需要以原始形式包含时使用，无需执行或转换。

用户可以读取降价和文本单元格，并运行代码单元格。图 3 显示了细胞的例子。

[![Examples of cells.](img/58fbcdfec43f69a99aeeaaa7d70274c7.png)](/sites/default/files/blog/2021/03/Jupyter_cell_example.png)Figure 3: Examples of cells.

Figure 3: Examples of cells.

### 编辑和执行单元格

笔记本用户界面是*模态*。这意味着根据笔记本电脑的模式，键盘的行为会有所不同。笔记本有两种模式:编辑和命令。

当一个单元格处于编辑模式时，它有一个绿色的单元格边框，并在编辑器区域显示一个提示，如图 4 所示。在这种模式下，您可以像普通的文本编辑器一样在单元格中键入内容。

[![Code cell in edit mode with prompt to allow editing](img/955413d032b4652cc54197b9f7d0ab17.png "Jupyter_cell_edit")](/sites/default/files/blog/2021/03/Jupyter_cell_edit.png)

Figure 4: A cell in edit mode.

当一个单元格处于命令模式时，它有一个蓝色的单元格边框，如图 5 所示。在此模式下，您可以使用键盘快捷键来执行笔记本和单元格操作。比如在命令模式下按 **Shift+Enter** 执行当前单元格。

[![Cell in command mode](img/5a385a2e0358325a002cb139ad12df29.png "Jupyter_cell_select")](/sites/default/files/blog/2021/03/Jupyter_cell_select.png)

Figure 5: A cell in command mode.

### 运行代码单元格

要运行代码单元:

1.  单击代码单元格左上角[ ]区域内的任意位置。这将使单元进入命令模式。
2.  按 **Shift+Enter** 或者选择**单元格— >运行**。

代码单元按顺序运行；也就是说，每个代码单元只有在它前面的所有代码单元都运行之后才运行。

## Jupyter 笔记本入门

Jupyter 笔记本项目支持许多编程语言。在这个例子中，我们将使用 IPython。它使用与 Python 相同的语法，但提供了更具交互性的体验。您将需要以下 Python 库来进行机器学习所需的数学计算:

*   NumPy:用于创建和操作向量和矩阵。
*   熊猫:用于分析数据和数据争论。Pandas 获取 CSV 文件或数据库等数据，并从中创建一个名为`DataFrame`的 Python 对象。A `DataFrame`是 Pandas API 中的中心数据结构，类似于电子表格，如下所示:
    *   A `DataFrame`在单元格中存储数据。
    *   一个`DataFrame`有命名的列(通常)和编号的行。
*   Matplotlib:用于可视化数据。
*   Sklern:用于监督和非监督学习。该库为模型拟合、数据预处理、模型选择和模型评估提供了各种工具。它有内置的机器学习算法和模型，称为*估计器*。每个估计器都可以使用其`fit`方法来拟合一些数据。

## 使用 jupiter 笔记本进行机器学习

我们将使用 [MANUela ML 模型](https://github.com/sa-mw-dach/manuela-dev/blob/master/ml-models/anomaly-detection/Anomaly-Detection-simple-ML-Training.ipynb)作为笔记本示例来探索机器学习所需的各种组件。用于训练模型的数据位于 [raw-data.csv](https://github.com/sa-mw-dach/manuela-dev/blob/master/ml-models/anomaly-detection/raw-data.csv) 文件中。

笔记本遵循图 6 所示的工作流程。下面是对这些步骤的解释。

[![The notebook workflow for machine learning is explained in the sections of text that follow.](img/628e60549786fccc455eba3c6ccdd052.png "Dzone_ML_flow1")](/sites/default/files/blog/2021/03/Dzone_ML_flow1.png)

Figure 6: Notebook workflow for machine learning.

### 步骤 1:探索原始数据

使用代码单元导入所需的 Python 库。然后，将原始数据文件(`raw-data.csv`)转换为带有时间序列、泵 ID、振动值和指示异常的标签的`DataFrame`。所需的 Python 代码显示在图 7 的代码单元中。

[![Using a code cell to hold the IPython code that imports libraries and converts the raw data.](img/4f6dde18d8fe2e9871eb8f2c8a40f64e.png "Jupyter_wrangle")](/sites/default/files/blog/2021/03/Jupyter_wrangle.png)

Figure 7: Importing libraries and converting raw data.

运行单元产生一个带有原始数据的`DataFrame`，如图 8 所示。

[![Data frame with raw data](img/340717c46a844db4e712ea2220586e48.png "Jupyter_raw_df")](/sites/default/files/blog/2021/03/Jupyter_raw_df.png)

Figure 8: Data frame with raw data.

现在想象一下`DataFrame`。图 9 中的上图显示了振动数据的子集。下图显示了手动标记的异常数据(1 =异常，0 =正常)。这些都是机器学习模型应该检测到的异常。

[![A visualization shows raw data and anomalies as two charts.](img/742584831b564f10a11a25ca523ec169.png "Jupyter_plot_raw")](/sites/default/files/blog/2021/03/Jupyter_plot_raw.png)

Figure 9: Visualizing raw data and anomalies.

在对原始数据进行分析之前，需要对其进行转换、清理，并将其组织成更适合分析的其他格式。这个过程被称为*数据角力*或*数据角力*。

我们将把原始时间序列数据转换成小片段，用于监督学习。代码如图 10 所示。

[![Code for creating a new data frame](img/3688216baaf29d4f0114d497347b1380.png "Jupyter_data_episodes")](/sites/default/files/blog/2021/03/Jupyter_data_episodes.png)

Figure 10: Creating a new data frame.

我们希望将数据转换成长度为 5 集的新的`DataFrame`。图 11 显示了一个样本时间序列数据集。

[![Example of time series data](img/8b2a301f362c4fa3a3f12cfebe53b064.png "Jupyter_raw_timeseries")](/sites/default/files/blog/2021/03/Jupyter_raw_timeseries.png)

Figure 11: Example of time series data.

如果我们将样本数据转换成长度= 5 的剧集，我们会得到类似于图 12 的结果。

[![New data frame with data converted into episodes](img/17d00cf665d67ef2086b30ea11853a5c.png "Jupyter_episode")](/sites/default/files/blog/2021/03/Jupyter_episode.png)

Figure 12: New data frame with episodes.

现在让我们使用图 13 中的代码将时间序列数据转换成剧集。

[![Code for converting data into episodes](img/13ae1979456879e7c7356c31181965fa.png "Jupyter_data_episodes_code")](/sites/default/files/blog/2021/03/Jupyter_data_episodes_code.png)

图 13:将数据转换成剧集。

图 14 研究了长度为 5 的剧集的数据以及最后一列中的标签。

[![Episodes of length 5 and the label in the last column](img/707011a65da4ee4ea32d6c54581e7b80.png "Jupyter_data_epis_label")](/sites/default/files/blog/2021/03/Jupyter_data_epis_label.png)

图 14:长度为 5 的剧集和最后一列中的标签。

**注**:在图 14 中，F5 列是最新数据值，F1 列是给定剧集的最早数据。标签 L 表示是否存在异常。

数据现在可以进行监督学习了。

### 步骤 2:特征和目标列

像许多机器学习库一样，Sklern 需要分离的特征(X)和目标(Y)列。因此，图 15 将我们的数据分为特性和目标列。

[![Code for splitting data into feature and target columns](img/6e490258786ceeaadefb61ba68dab835.png "Jupyter_data_separate")](/sites/default/files/blog/2021/03/Jupyter_data_separate.png)

图 15:将数据分为特性和目标列。

### 步骤 3:训练和测试数据集

将数据集分成两个子集是一种很好的做法:一个子集用于训练模型，另一个子集用于测试训练好的模型。

我们的目标是创建一个能很好地概括新数据的模型。我们的测试集将作为新数据的代理。我们将数据集分为 67%的训练集和 33%的测试集，如图 16 所示。

[![Code for splitting data into training and test data sets](img/21c3b3f9c5ef1e54eee2b7bf1bf8fd5b.png "Jupyter_data_split")](/sites/default/files/blog/2021/03/Jupyter_data_split.png)

图 16:将数据分为训练和测试数据集。

我们可以看到，训练集和测试集的异常率是相似的；也就是说，数据集被相当公平地分割了。

### 第四步:模特训练

我们将使用`DecisionTreeClassifier`进行模型训练。*决策树*是一种用于分类和回归的监督学习方法。目标是创建一个模型，通过学习从数据特征推断的简单决策规则来预测目标变量的值。

`DecisionTreeClassifier`是一个在数据集上执行多类分类的类，尽管在这个例子中我们将使用它来分类成一个类。`DecisionTreeClassifier`将两个数组作为输入:数组 X 作为特征，数组 Y 作为标签。拟合后，模型可用于预测测试数据集的标签。图 17 显示了我们的代码。

[![Code for model training with DecisionTreeClassifier](img/74d0187f7e7da8c46620006d5a761fed.png "Jupyter_model")](/sites/default/files/blog/2021/03/Jupyter_model.png)

图 17:用决策树分类器进行模型训练。

我们可以看到，该模型取得了很高的准确率。

### 步骤 5:保存模型

保存模型并再次加载它以验证它的工作，如图 18 所示。

[![Code for saving the model](img/c33924dc7d046afcc845323fb53cbc41.png "Jupyter_model_save")](/sites/default/files/blog/2021/03/Jupyter_model_save.png)

图 18:保存模型。

### 步骤 6:用模型进行推理

现在我们已经创建了机器学习模型，我们可以用它来对实时数据进行推理。

在这个例子中，我们将使用谢顿来服务模型。为了让我们的模型在 Seldon 下运行，我们需要创建一个具有 predict 方法的类。predict 方法可以接收 NumPy 数组 X，并将预测结果返回为:

*   数字阵列
*   值的列表
*   一个字节串

我们的代码如图 19 所示。

[![Code for using Seldon to serve the ML Model](img/2a9cd7e577518621c711ca84006c8544.png "Jupyter_seldon_class")](/sites/default/files/blog/2021/03/Jupyter_seldon_class.png)

图 19:使用很少服务于机器学习

最后，让我们测试该模型是否能够预测一系列值的异常，如图 20 所示。

[![Code for Inference using the model](img/e7c738c4982e45f4fa319e743cc5a84a.png "Jupyter_seldon_result")](/sites/default/files/blog/2021/03/Jupyter_seldon_result.png)

图 20:使用模型的推理。

我们可以看到，该模型在推理方面也获得了高分。

## 本文的参考资料

有关本文中讨论的主题的更多信息，请参阅以下资料:

*   [欢迎来到合作实验室](https://colab.research.google.com/)
*   [关于 GESIS 笔记本](https://notebooks.gesis.org/)
*   [项目 Jupyter 主页](https://jupyter.org/)

*Last updated: August 15, 2022*