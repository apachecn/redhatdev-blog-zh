# 您喜欢使用 Red Hat OpenShift 数据科学的 4 个原因

> 原文：<https://developers.redhat.com/blog/2021/04/27/4-reasons-youll-love-using-red-hat-openshift-data-science>

[Red Hat Open shift Data Science](https://www.redhat.com/en/technologies/cloud-computing/openshift/openshift-data-science)是一个托管云服务，由上游[开放数据中心](https://opendatahub.io/)项目的一组精选组件构建而成。它旨在提供一个稳定的沙箱，数据科学家可以在其中开发、培训和测试他们的机器学习(ML)工作负载，然后以容器就绪的格式部署结果。本文总结了在您的[机器学习](/topics/ai-ml)项目中使用 OpenShift 数据科学的优势。

## 容器让数据科学变得简单

虽然像 JupyterLab(如图 1 所示)这样的工具已经为数据科学家在他们的机器上开发模型提供了直观的方法，但协作和共享工作总是存在固有的复杂性。此外，当您必须购买和维护自己的硬件时，使用强大的 GPU 等专用硬件可能会非常昂贵。OpenShift Data Science 中包含的 JupyterHub 允许数据科学家将他们的开发环境带到云上。因为所有的工作负载都是作为容器运行的，所以协作就像与您的团队成员共享一个映像一样简单，甚至只需[将它添加到他们可以使用的默认容器列表](https://www.youtube.com/watch?v=p8WGxiH55lE)中。GPU 和大容量内存也突然变得更容易访问，因为你不再受笔记本电脑所能支持的限制。所有这一切，你可以保持同样的 UX 和你一直喜欢的开发工作流程。

[![An open JupyterLab notebook.](img/857524a7c7dfa61c65ced9f6a8598683.png "jupyterlab-notebook")](/sites/default/files/blog/2021/04/jupyterlab-notebook.png)Figure 1: A JupyterLab notebook

Figure 1: A JupyterLab notebook.

## 安全构建的笔记本电脑映像

软件栈，尤其是那些涉及机器学习的，往往是复杂的猛兽。在 [Python](/blog/category/python/) 生态系统中有许多可以使用的模块和库，所以决定使用哪个版本的库是非常具有挑战性的。如图 2 所示，OpenShift Data Science 附带了许多打包的笔记本图像，这些图像是根据数据科学家和推荐引擎(如 [Thoth adviser](/blog/2020/09/30/ai-software-stack-inspection-with-thoth-and-tensorflow/) )的见解构建的。这使得数据科学家可以快速启动新项目，而不必担心从随机的上游存储库中下载未经证实且可能不安全的图像。

[![The dialog to create and start a notebook server in JupyterHub.](img/ab0474f2918a7a9e13d9df92355ec9a5.png "notebook-selection")](/sites/default/files/blog/2021/04/notebook-selection.png)Figure 2: Notebook images available in JupyterHub.

Figure 2: Notebook images available in JupyterHub.

## 与第三方机器学习工具的集成

我们都遇到过这样的情况，我们最喜欢的工具或服务不能很好地相互配合。OpenShift Data Science 的设计考虑了灵活性。如图 3 所示，大量开源和第三方 [AI/ML](/topics/ai-ml) 工具可以与 OpenShift Data Science 一起使用。这些工具支持完整的机器学习生命周期，从数据工程和特征提取到模型部署和管理。再也不会丢下你最喜欢的玩具了。

[![Explore third-party AI/ML tools available for use with OpenShift Data Science.](img/a5e975f46a629089adbeeba57afc02f2.png "ISV-Ecosystem")](/sites/default/files/blog/2021/04/ISV-Ecosystem-1.png)

Figure 3: Third-party integrations available to use with OpenShift Data Science.

## 先操作后测试

开放数据中心是一个[开源](/topics/open-source)社区项目，由 30 多个 AI/ML 工具组成，涵盖了任何机器学习计划可能需要的整个生命周期。 [Operate First](https://www.operate-first.cloud/) 计划旨在开放环境中部署最常用组件的子集，以获得额外的运营专业知识，并帮助强化上游项目。OpenShift Data Science 采用最常用的*和*稳定组件的核心集，并在 [Red Hat OpenShift 专用](https://www.openshift.com/products/dedicated/)和[Red Hat open shift AWS 服务](https://www.openshift.com/products/amazon-openshift)上将其作为托管云服务交付。这意味着数据科学家可以专注于快速迭代和实验，同时利用 Red Hat 在 [Red Hat OpenShift](/products/openshift/overview) 上运行复杂工作负载的经验。

## 结论

[了解有关 OpenShift 数据科学的更多信息](http://www.redhat.com/en/technologies/cloud-computing/openshift/openshift-data-science)或[观看视频演示](http://www.openshift.com/DataScienceVideoDemo)了解实际操作。你可以在 https://opendatahub.io/亲自尝试上游开放数据枢纽项目。

*Last updated: October 14, 2022*