# Node.js 社区发生了什么

> 原文：<https://developers.redhat.com/blog/2021/04/08/whats-happening-in-the-node-js-community>

好奇 [Node.js](/topics/nodejs/) 社区发生了什么？

Node.js 16 将于 2021 年 4 月发布，2021 年 10 月推广为长期支持。我们也很快接近 Node.js 10 的寿终正寝日期。2021 年 4 月之后，Node.js 10 发行版将不再提供更多补丁或安全修复。如果你还没有，你应该计划尽快升级到 Node.js 12 或者 Node.js 14。参见图 1 中的 [Node.js 发布时间表](https://github.com/nodejs/release#release-schedule)。

![Node.js release timeline spanning October 2020 to January 2023.](img/1aae987f5c169a737712200cd1b49bc8.png)
图 1:node . js 发布时间线概述。

## Node.js 15 中的新特性

“当前”发布版本 Node.js 15 首先选择了贡献给运行时的新特性。Node.js 15 中现在可用的特性包括:

*   [T2`crypto.randomUUID()`](https://github.com/nodejs/node/pull/36729)
*   [`fsPromises.watch()`](https://github.com/nodejs/node/pull/37179) ，返回`AsyncIterator`的`fs.watch()`的替代版本
*   [新的`perf_hooks.createHistogram()` API](https://github.com/nodejs/node/pull/37155) ，用于创建允许用户记录的直方图实例
*   [npm 7.5](https://github.com/nodejs/node/pull/37117) ，包括新的`npm diff`命令
*   [对源地图的支持](https://github.com/nodejs/node/pull/37362)已经从实验状态升级到稳定状态(由 Benjamin Coe 提出)

## Node.js 社区中的热门话题

以下问题最近在 Node.js 社区引发了讨论:

*   [为 Apple Silicon](https://github.com/nodejs/node/issues/37309) 生产 native Node.js 二进制文件的工作进展顺利，将作为 macOS 的单一“fat”(多架构)二进制文件发布
*   [围绕在 Node.js 核心](https://github.com/nodejs/node/issues/19393#)中包含`fetch()`或类似`fetch()`的 API 的新讨论
*   [提议将实验性原料药`AsyncResource`和`AsyncLocalStorage`提升至稳定状态](https://github.com/nodejs/node/issues/35286#)

## Node.js 包维护生态系统

Node.js 包维护工作组旨在以多种方式帮助维护者。两项积极努力是:

*   **鼓励发布包支持信息**:发布包支持信息有助于维护人员在支持的 Node.js 版本、一般支持可用性以及给定包背后的支持方面设定期望。工作组在 [`PACKAGE-SUPPORT.md`](https://github.com/nodejs/package-maintenance/blob/main/docs/PACKAGE-SUPPORT.md) 中定义了推荐元数据。本月新增:在[支持](https://github.com/pkgjs/support)工具中添加了一个`create`命令，这使得维护人员可以更容易地将这些元数据添加到他们的包中。`npx @pkgjs/support create`将指导您将推荐的元数据添加到您的包中。Nodeshift 项目最近向我们的模块添加了包支持信息。你可以在本文中了解更多关于我们的经验。
*   **威比测试工具**:工作组继续开发威比(“我会打垮你吗？”).仍然在早期开发阶段，这个工具帮助包维护者测试他们包中的变化是否会破坏依赖他们的其他包。如果你对模块测试感兴趣，或者你是一个模块维护者，想要测试你的改变对你的下游依赖者的影响，你可以观看一个[演示](https://youtu.be/m4SMPUshtzY?t=47)和/或关注 [GitHub 库](https://github.com/pkgjs/wiby)的进展。

## Node.js 的未来 10 年

Node.js 项目记录了我们认为使 Node.js 的下一个 10 年像第一个 10 年一样成功的重要因素。 [Next-10](https://github.com/nodejs/next-10) 工作的重点是定义项目的技术价值和支持者，为未来的讨论奠定基础。我们已经进行了多次对话，并记录了我们最初的想法，但现在我们需要你的帮助。该项目发起了一项调查，以确认这些价值观和支持者符合我们用户的需求。通过参加[调查](https://www.surveymonkey.com/r/8PFGKV5)，你可以帮助指引 Node.js 的未来。

## 即将举行的虚拟活动

虽然在过去的一年里我们无法见面，但 Node.js 社区仍然会在虚拟活动中聚会。即将举办的活动包括:

*   [OpenJS 世界](https://openjsf.org/openjs-world-2021/)(2021 年 6 月 2 日)
*   [NodeConf 远程](https://www.nodeconfremote.com/)(2021 年 10 月 18 日-21 日)

## 在 Node.js 上保持最新状态

*   [红帽开发者上的 node . js](/topics/nodejs)
*   [IBM Developer 上的 node . js](https://developer.ibm.com/languages/node-js/)
*   [node . js 项目博客](https://nodejs.org/en/blog/)

*Last updated: October 14, 2022*