# 面向开发人员的 Red Hat Enterprise Linux 8 Beta 备忘单

> 原文：<https://developers.redhat.com/blog/2019/03/04/red-hat-enterprise-linux-8-beta-cheat-sheet-for-developers>

我很高兴介绍我们新的面向开发人员的 Red Hat Enterprise Linux 8 测试版备忘单。

本文档面向以下人员:

1.  已经熟悉 RHEL 命令，但你想快速参考新的 RHEL 8 测试版
2.  刚到 RHEL，想开始探索 RHEL 8

这里有一个你可以访问的例子:通用 模块 命令。

*   列出所有模块

```
       $ yum module list
```

*   列出已安装的模块:

```
       $ yum module list installed
```

*   找到哪个模块提供了一个包:  

```
       $ yum module provides *package*
```

*   查看一个模块的详细信息:  

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

**[今天下载](https://developers.redhat.com/cheat-sheets/red-hat-enterprise-linux-8-beta/)红帽企业 Linux 8 小抄。**

通过加入红帽开发者，可以免费获得红帽企业 Linux 8 测试版。参见[developers.redhat.com/rhel8](https://developers.redhat.com/rhel8)

*Last updated: March 1, 2019*