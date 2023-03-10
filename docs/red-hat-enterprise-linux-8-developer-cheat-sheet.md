# Red Hat Enterprise Linux 8 开发者备忘单

> 原文：<https://developers.redhat.com/blog/2019/05/07/red-hat-enterprise-linux-8-developer-cheat-sheet>

随着 [Red Hat Enterprise Linux 8](https://wp.me/p8e0as-2rYd) 的发布，我很高兴为开发人员介绍我们新的 RHEL 8 备忘单。此版本已从测试版更新，以反映 RHEL 8 中的最终更新。本文档面向以下人员:

1.  已经熟悉 Red Hat Enterprise Linux，但是您想要快速参考新的 RHEL 8 命令。
2.  刚接触 Red Hat Enterprise Linux，想开始探索 RHEL 8。

下面是一些常见模块命令的示例，您可以在这个备忘单中找到。

*   列出所有模块:

```
       $ yum module list
```

*   列出已安装的模块:

```
       $ yum module list installed
```

*   查找哪个模块提供包:

```
       $ yum module provides *package*
```

*   检查模块的详细信息:

```
       $ yum module info *module*
```

*   列出模块概要文件安装的软件包:

```
       $ yum module info --profile *module:stream*
```

*   显示模块的当前状态:

```
       $ yum module list *module*
```

**今天就下载[红帽企业 Linux 8 备忘单](https://developers.redhat.com/cheat-sheets/red-hat-enterprise-linux-8/)。**

红帽企业版 Linux 8 可通过加入红帽开发者:[developers.redhat.com/rhel8](https://developers.redhat.com/rhel8)进行应用开发。

*Last updated: May 13, 2019*