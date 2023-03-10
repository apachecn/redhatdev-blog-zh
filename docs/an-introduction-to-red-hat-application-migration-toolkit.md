# Red Hat 应用程序迁移工具包简介

> 原文：<https://developers.redhat.com/blog/2018/06/01/an-introduction-to-red-hat-application-migration-toolkit>

[如果你没有关注 [Red Hat JBoss 中间件博客](https://middlewareblog.redhat.com/)，我们将[在 developers.redhat.com 上重新发布 Red Hat 应用程序迁移工具包的介绍](https://middlewareblog.redhat.com/2018/05/24/an-introduction-to-red-hat-application-migration-toolkit/)

应用程序迁移和现代化可能是一项艰巨的任务。您不仅必须用新的库和 API 更新遗留应用程序，而且通常还必须处理新的框架、基础设施和架构，同时保持资源专用于新的特性和版本。

Red Hat Application Migration Toolkit(RHAMT)，以前称为 Windup，提供了一组实用程序来简化这一过程。可以通过命令行界面(CLI)、基于 web 的界面或直接在 Eclipse 中分析应用程序，从而允许立即修改源代码。

这些实用程序允许您同时快速了解数千个应用程序。它们识别迁移挑战和应用程序之间共享的代码或依赖性，并加速进行必要的代码更改，以使您的应用程序在最新的中间件平台上运行。

## 选择正确的发行版

您已经阅读了介绍，可能还看过视频，并且渴望通过这个过程运行您的第一个应用程序。你从哪里开始？

RHAMT 提供了许多不同的发行版来满足您的需求，并且所有发行版都包含了详细的报告，这些报告通过工作量估算突出了迁移问题。每一个都总结如下。

## 硬币指示器 （coin-levelindicator 的缩写）命令行界面（Command Line Interface for batch scripting）

[CLI 下载](https://developers.redhat.com/download-manager/file/migrationtoolkit-rhamt-cli-4.0.1-offline.zip?sc_cid=7016000000154B7AAI)–[产品文档](https://access.redhat.com/documentation/en-us/red_hat_application_migration_toolkit/4.0/html-single/cli_guide/?sc_cid=7016000000154B7AAI)

CLI 是一个命令行工具，它提供了对报告的访问，而没有其他工具的开销。它包括一系列定制选项，允许您微调 RHAMT 分析选项或者与外部自动化工具集成。

## Web 控制台

[Web 控制台下载](https://developers.redhat.com/download-manager/file/migrationtoolkit-rhamt-web-distribution-4.0.1-with-authentication.zip?sc_cid=7016000000154B7AAI)–[产品文档](https://access.redhat.com/documentation/en-us/red_hat_application_migration_toolkit/4.0/html-single/web_console_guide/?sc_cid=7016000000154B7AAI)

web 控制台是一个基于 web 的系统，允许一组用户评估迁移和现代化工作并确定其优先级。此外，可以将应用程序分组到项目中进行分析。

## Eclipse 插件

[Eclipse 插件下载](https://developers.redhat.com/download-manager/file/migrationtoolkit-rhamt-eclipse-plugin-repository-4.0.1.zip?sc_cid=7016000000154B7AAI)–[产品文档](https://access.redhat.com/documentation/en-us/red_hat_application_migration_toolkit/4.0/html-single/eclipse_plugin_guide/?sc_cid=7016000000154B7AAI)

Eclipse 插件直接在 Eclipse 和 Red Hat JBoss Developer Studio(JBDS)中提供帮助，它允许开发人员直接在源代码中看到迁移问题。Eclipse 插件还提供解决问题的指导，并在可能的情况下提供自动代码替换。

## 从选择发行版开始

*   如果您所在的团队需要对报告进行并发访问，或者您有大量的应用程序要分析，那么请选择 web 控制台。
*   如果您是熟悉 Eclipse 或 JBDS 的开发人员，并且想要实时反馈，那么从 Eclipse 插件开始吧。
*   否则，我们建议从 CLI 开始。

遵循所选发行版的下载链接，然后查看相应指南的前几章，以安装和运行该工具。

# 分析应用程序

假设您有一个 RHAMT 的本地安装，位于 RHAMT_HOME，还有一个您想要分析的应用程序。出于本博客的目的，我们还假设您选择了 CLI。说完了，让我们开始吧。

通过调用`rhamt-cli`并在应用程序中传递它以及任何所需的选项来执行分析，如下例所示。

```
$ bin/rhamt-cli --sourceMode --input /path/to/source_folder/ --output /path/to/output_folder/ --target eap7
```

选项很简单:

*   `–sourceMode`表示输入文件是源文件，而不是编译的二进制文件。
*   `–input`指定包含待分析文件的文件或目录的路径。
*   `–output`指定包含报告的目录的路径。
*   `–target`指定要迁移到的技术；它用于确定分析的规则。

分析完成后，将在控制台中看到一条消息，指示报告的路径。

`Report created: /path/to/output_folder/index.html
Access it at this URL: file:///path/to/output_folder/index.html`

# 规则

RHAMT 的所有发行版都利用相同的规则引擎来分析您计划迁移的应用程序所使用的 API、技术和架构。该引擎从档案中提取文件，反编译类，扫描和分类文件类型，分析 XML 和其他文件内容，分析应用程序代码，然后生成报告。

每个操作都由定义的规则处理，这些规则由一组一旦满足条件就要执行的操作组成。在后续的文章中，我们将更深入地研究规则是如何工作的，以及如何创建您自己的定制规则，但是现在，我们知道 RHAMT 包含了一套全面的标准迁移规则来帮助您入门。

# 就想“举一反三”？

提升和转移，或*重新托管*，一个应用程序可能是迁移它的第一步。这个过程包括将应用程序移动到不同的目标运行时或基础设施上。此阶段的一个共同最终目标是进行最少的更改，以使应用程序在云环境中成功运行。

一旦应用程序成功地在云中运行，下一步就是对应用程序进行现代化，以便它是为云环境而设计的。这一步不是简单地重新托管应用程序，而是重新设计它，将不必要的依赖项和库移出应用程序。

无论您处于哪一步，RHAMT 都通过提供一组云就绪规则来协助这两个步骤。一旦针对应用程序执行，就会创建一个详细的报告，指出应该进行哪些更改。对于熟悉使用 RHAMT 迁移中间件平台的人来说，过程是相似的:检查报告并根据反馈调整应用程序。

就这么简单。

# 摘要

无论您处于迁移过程的哪个阶段，我都推荐您查看 RHAMT。它的设置非常简单，并且附带了许多默认规则来帮助迁移和现代化过程的任何部分。此外，RHAMT 便于一次性解决独特的问题；确定给定的解决方案后，可以创建自定义规则来捕获该解决方案，从而极大地简化迁移过程。

请继续关注我们的下一次更新，我们将讨论如何创建定制规则，以便在您的环境中更好地利用 RHAMT。

# 参考

*   [红帽应用迁移工具包](https://developers.redhat.com/products/rhamt/overview/?sc_cid=7016000000154B7AAI)
*   [Red Hat 应用程序迁移工具包的产品文档](https://access.redhat.com/documentation/en-us/red_hat_application_migration_toolkit/?sc_cid=7016000000154B7AAI)