# 开发者为什么要关心 GitOps？

> 原文：<https://developers.redhat.com/blog/2021/05/13/why-should-developers-care-about-gitops>

作为一名软件开发人员，你的主要任务是交付比特:按照设计和预期工作的可执行的 1 和 0 的片段。这些位如何进入[容器](/topics/containers)或虚拟机(VM)？谁在乎呢。

你。你在乎。我知道你有，因为开发者天生就是这样的。显然，在某些时候，您已经观察到您的代码工作正常(因为您所有的代码都没有 bug，对吗？)，所以您很想看到它在测试、试运行和生产中以完全相同的方式运行。

这么说吧:如果代码在你的机器上运行，而*在测试中没有以同样的方式运行*，你的第一个问题是，“有什么不同？”

GitOps 可以回答这个问题。更准确也更重要的是，GitOps 可以帮助您确保一切正常。

现在 [Red Hat OpenShift 管道和 Red Hat OpenShift GitOps 已经普遍可用](https://www.openshift.com/blog/openshift-pipelines-and-openshift-gitops-are-now-generally-available)，这是从 [Red Hat OpenShift](/products/openshift/getting-started) 开发者的角度深入 gitop 的好时机。

## 什么是 GitOps？

将所有内容存储在 Git 中。

好吧，这是一个可怕的过度简化，但这是 GitOps 的本质:您在您的 Git 存储库中存储您的代码和基础设施并构建配置信息(“repo”)。诸如 OpenShift Pipelines、ArgoCD 和 Kustomize 之类的工具与这一概念一起工作，并在这一概念内使事情发生并把所有事情拉在一起。

可以这么说，如果你采用 GitOps 和所有与这个想法和技术相关的东西，一切都变成了代码。

现在，作为一名开发人员，考虑一下这个问题。代码。这是你的专长；这是你发光的地方。想象一下，能够控制开发的每个方面，从测试到编写代码，到构建解决方案，再到部署解决方案。突然，作为一个开发人员，你从头到尾都在掌控之中。

这并不是说不涉及运营。相反:这意味着你与运营部门合作，构建一个完整的解决方案。开发人员和运营人员一起工作。开发商。运营。戴夫。行动组。 [DevOps](/topics/devops) ！

## 从哪里开始？

*   尝试[这些 Katacoda 场景](https://learn.openshift.com/gitops)了解更多信息。(它们在浏览器中运行，所以不需要安装。)
*   下载我们的免费 DevOps 书， [*DevOps 带 OpenShift*](/topics/devops) 。
*   在 Red Hat Developer 查看[更多 GitOps 文章。](/blog/tag/gitops)

软件开发和运营正在融合，云原生的 [CI/CD 空间](/topics/ci-cd)正在快速发展。现在是升级你的游戏和自动化所有事情的绝佳时机。

*Last updated: May 10, 2021*