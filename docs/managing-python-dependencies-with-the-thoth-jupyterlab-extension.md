# 用 Thoth JupyterLab 扩展管理 Python 依赖关系

> 原文：<https://developers.redhat.com/blog/2021/03/19/managing-python-dependencies-with-the-thoth-jupyterlab-extension>

JupyterLab 是一款灵活而强大的工具，用于处理 Jupyter 笔记本电脑。它的交互式用户界面(UI)让你可以使用终端、文本编辑器、文件浏览器和 Jupyter 笔记本上的其他组件。JupyterLab 3.0 于 2021 年 1 月发布。

Thoth 项目开发了增强开发人员和数据科学家日常生活的[开源](/topics/open-source)工具。透特使用机器生成的知识，通过使用[人工智能](/topics/ai-ml)的强化学习来提高应用程序的性能、安全性和质量。([观看此视频](https://www.youtube.com/watch?v=WEJ65Rvj3lc&t=1s)了解更多关于使用强化学习解决依赖性的信息。)

这种机器学习方法在 [Thoth adviser](https://github.com/thoth-station/adviser) 中实现，这是一个针对 Python 应用的推荐引擎。[透思集成](https://github.com/thoth-station/adviser/blob/master/docs/source/integration.rst)根据[用户输入](https://github.com/thoth-station/thamos#using-custom-configuration-file-template)利用这些知识提供软件堆栈建议。

本文向您介绍了 [jupyterlab-requirements](https://github.com/thoth-station/jupyterlab-requirements) ，这是一个 jupyterlab 扩展，用于管理和优化 Jupyter 笔记本中的 Python 依赖关系。正如您将了解到的，使用`jupyterlab-requirements`扩展是确保您的代码和实验总是可重复的一种聪明而简单的方法。

## 使应用程序依赖性可重现

当创建代码或进行实验时，可再现性是一个重要的要求。确保其他人可以在创建者使用的相同环境中重新运行实验是至关重要的，尤其是在开发机器学习应用程序时。

让我们考虑开发应用程序的第一步:指定依赖关系。例如，你的项目可能依赖于[熊猫](https://pandas.pydata.org/docs/getting_started/index.html)进行数据探索和操作，或者 [TensorFlow](https://pypi.org/project/tensorflow/) 进行模型训练。

完成这项任务的一种方法是在笔记本单元中运行一个命令，直接在主机上安装依赖项，如图 1 所示。这样，下一个用户可以运行同一个单元并安装类似的包。

[![A user installing dependencies in the notebook cell using the pip install command.](img/1ee3cb2dc098408a4bad694ac772407f.png "Screenshot from 2020-10-28 11-18-16")](/sites/default/files/blog/2021/01/Screenshot-from-2020-10-28-11-18-16.png)

另一个潜在的策略是提供一个列出所有依赖项的`requirements.txt`文件，以便其他人可以在启动笔记本之前安装它们。图 2 显示了一个例子。

[![A sample list of software dependencies in a text file.](img/e343a4db7a4c9b6b89f20ceb3223aa2e.png "Screenshot from 2020-11-03 08-31-21")](/sites/default/files/blog/2021/01/Screenshot-from-2020-11-03-08-31-21.png)

Figure 2: A sample list of dependencies.

您认为这两种指定依赖关系的方法有什么问题吗？

两者都不支持再现性！

在第一个场景中，假设在新版本的库发布后，另一个用户试图重新运行同一个单元格。他们可能会体验到与最初笔记本输出不同的行为。

同样的问题也会出现在`requirements.txt`文件中，只是包名不同。即使您用确切的版本号陈述了直接依赖关系，这些依赖关系中的每一个都可能依赖于同样安装的其他所谓的*传递依赖关系*。

为了保证可再现性，您必须考虑所有具有特定版本号的直接和可传递依赖关系，包括出于安全原因用于验证软件包出处的所有哈希(查看这些[文档](https://thoth-station.ninja/docs/developers/adviser/provenance_checks.html)以了解更多关于软件堆栈中的安全性的信息)。更准确地说，Python 版本、操作系统和硬件都会影响代码的行为。您应该共享所有这些信息，以便其他用户可以体验相同的行为并获得类似的结果。

Thoth 项目旨在帮助您指定直接和可传递的依赖关系，以便您的应用程序总是可重复的，并且您可以专注于更紧迫的挑战。

## 使用 jupyterlab 的依赖性管理-需求

Thoth 团队已经引入了 [jupyterlab-requirements](https://github.com/thoth-station/jupyterlab-requirements) ，这是 jupyterlab 对依赖管理的扩展，目前专注于 [Python](/blog/category/python/) 生态系统。这个扩展允许您直接从 Jupyter 笔记本中管理项目的依赖项，如图 3 所示。

[![Screenshot of the jupyterlab-requirements extension with the Managed Dependencies menu item highlighted.](img/84287027a718215d9695320b30ef7a43.png "jupyterlab-requirements")](/sites/default/files/blog/2021/01/jupyterlab-requirements.jpg)

Figure 3: Managing dependencies in JupyterLab.

当您点击**管理依赖关系**时，您将看到如图 4 所示的对话框。

[![A dialog box stating ‘No dependencies found! Click button above to add new packages.’](img/2f353d91f8939fa4ce3ce5df50c818b2.png "Screenshot from 2021-01-11 18-31-00")](/sites/default/files/blog/2021/01/Screenshot-from-2021-01-11-18-31-00.png)

Figure 4: A new notebook with no dependencies identified.

最初，当您启动新笔记本时，扩展不会识别任何依赖关系；它检查笔记本元数据来检测它们。您可以通过点击带有加号(+)图标的按钮来添加您的包，如图 5 所示。

[![Screenshot of the Manage Dependencies screen with the option to add new package dependencies.](img/2c28c617af05d1d76fc46e0952908b90.png "Screenshot from 2021-01-11 18-31-48")](/sites/default/files/blog/2021/01/Screenshot-from-2021-01-11-18-31-48.png)

Figure 5: Adding new packages with the jupyterlab-requirements extension.

保存后，会出现一个**安装**按钮。您可以在安装依赖项之前检查包的名称和版本，如图 6 所示。

[![Screenshot of the Manage Dependencies screen with the Install button and sample packages added.](img/9d66fd9bcd92ec275a89575553b8d3e5.png "Screenshot from 2021-01-11 18-46-42")](/sites/default/files/blog/2021/01/Screenshot-from-2021-01-11-18-46-42.png)

Figure 6: The Manage Dependencies screen with sample packages added and ready to install.

单击 Install 后，您将看到如图 7 所示的屏幕。

[![Screenshot of the Manage Dependencies screen with requirements locked, saved, and installed.](img/be062504911d94dabe2dde0e4526a471.png "Screenshot from 2021-01-11 18-53-12")](/sites/default/files/blog/2021/01/Screenshot-from-2021-01-11-18-53-12.png)

Figure 7: The packages are locked, saved, and installed in the notebook metadata.

所有依赖关系(包括直接依赖关系和可传递依赖关系)都将被锁定、保存在笔记本元数据中并安装。更重要的是，该扩展会自动为您的笔记本创建和设置内核。没有人的干预是必要的，你已经准备好工作在你的项目上。

## 管理现有笔记本中的依赖关系

如果你已经有了带代码的笔记本，你仍然可以使用`jupyterlab-requirements`扩展来共享它们。 [invectio](https://pypi.org/project/invectio/) 库分析笔记本中的代码，并建议运行笔记本必须安装的库。图 8 显示了一个例子。

[![There are no dependencies in the notebook metadata, but the extension identifies three packages to be installed.](img/01c58e7a90acba138888bf65c8a305c8.png "Screenshot from 2021-03-18 12-43-05")](/sites/default/files/blog/2021/03/Screenshot-from-2021-03-18-12-43-05.png)

同样，您可以只安装依赖项并开始处理您的项目。

## 锁定与 Thoth 或 Pipenv 的依赖关系

用于锁定依赖关系的解析引擎提供了两个文件:一个`Pipfile`和一个`Pipfile.lock`。`Pipfile.lock`文件陈述了所有直接的和可传递的项目依赖，以及具体的版本和散列。笔记本元数据存储这些文件以及关于 Python 版本、操作系统和检测到的硬件的信息。这样，使用同一台笔记本的任何人都可以重新创建原始开发人员使用的环境。

目前有两种分辨率的引擎可供选择:[透思](https://thoth-station.ninja/)和 [Pipenv](https://github.com/pypa/pipenv) 。

目前默认使用 Thoth，Pipenv 作为备用。这种设置保证了用户将会收到软件栈来处理他们的项目。将来，用户将能够选择特定的解析引擎。

使用 Thoth resolution 引擎，您可以从 Thoth 推荐系统请求一个满足您需求的优化软件堆栈。您可以根据自己的特定需求从以下建议类型中进行选择:

*   最近的
*   表演
*   安全性
*   稳定的
*   测试

有关各种[推荐类型](https://thoth-station.ninja/recommendation-types/)的更多信息，请访问 Project Thoth 网站。

**注意**:笔记本元数据存储使用了哪个解析引擎，这样任何人都可以立即看到哪个引擎用于解析依赖关系。

### 配置运行时环境

使用 Thoth resolution 引擎时，您不需要担心运行时环境。Thoth 自动识别生成建议所需的信息，并创建一个包含以下参数的 [Thoth 配置文件](https://github.com/thoth-station/thamos):

```
host: {THOTH_SERVICE_HOST}
tls_verify: true
requirements_format: {requirements_format}

runtime_environments:
  - name: '{os_name}:{os_version}'
    operating_system:
      name: {os_name}
      version: '{os_version}'
    hardware:
      cpu_family: {cpu_family}
      cpu_model: {cpu_model}
      gpu_model: {gpu_model}
    python_version: '{python_version}'
    cuda_version: {cuda_version}
    recommendation_type: stable
    platform: '{platform}'
```

**注意**:如果您使用 Thoth resolution 引擎，笔记本元数据还将包含有关笔记本运行时环境的信息。通过这种方式，使用该笔记本的其他数据科学家将被警告使用不同的笔记本。

### 安装依赖项和创建内核

一旦使用 Thoth 或 Pipenv 创建了锁文件， [micropipenv](https://pypi.org/project/micropipenv/) 工具就会在虚拟环境中安装依赖项。`micropipenv`工具支持 Python 及更高版本中的依赖管理(“一个库来管理它们”)。

一旦所有的依赖项都安装到内核中，您就可以在笔记本上工作了。

您可以选择新内核的名称，并从下拉菜单中选择需求。一旦所有东西都安装好了，内核就会自动分配给当前的笔记本。

## 结论

jupyterlab 需求扩展是由 Thoth 团队维护的一个开源项目。我们目前正在探索 UI 的新特性，我们欢迎任何想对这个扩展做出贡献或给我们反馈的人。

看看[未解决的问题](https://github.com/thoth-station/jupyterlab-requirements/issues)，如果你喜欢这个项目或者你发现这个扩展有任何问题，就和团队联系。透特团队也有一个[公共频道](https://chat.google.com/room/AAAAVjnVXFk)，在那里你可以询问关于这个项目的问题。我们总是很乐意在任何[我们的仓库](https://github.com/thoth-station)上与社区合作。

*Last updated: October 7, 2022*