# 红帽介绍 JDK 10

> 原文：<https://developers.redhat.com/blog/2018/04/24/red-hat-introduces-jdk-10>

## 支持 Java 10

Java 10 现在得到了 Red Hat JBoss Developer Studio 11.3 的支持。

请注意，Red Hat JBoss Developer Studio 不能在 Java 9/10 虚拟机上运行，但是允许管理和构建 Java 9/10 项目和工件。因此，如果您想要管理和构建 Java 9/10 项目，您必须首先在您的工作空间中定义一个 Java 9/10 JDK。

由于 Java 10 是 Java 9 的扩展，请参考这篇[文章](https://developers.redhat.com/blog/2017/11/09/red-hat-introduces-jdk-9/)了解 Red Hat JBoss Developer Studio 中与 Java 9 相关的支持。

最大的部分是对局部变量类型推断的支持。

### 添加 Java 10 JRE

认识 Java 10 启动的基本必要性

![j10](img/362ff4ec81a0853367199a45eef6342f.png)

以及编译器兼容性选项 10

![](img/08bf8f69c69eec2444139d1745a7c3e5.png)

##### JEP 286 var -编译

支持编译变量，如下所示

![](img/636424aaf88112afaa24c2720ac41416.png)

按预期标记编译器错误，如下所示

![](img/a6b9eaf191868329c1744ddee3a97ac0.png)

允许在 var 位置完成

![](img/f0f4f433b7236d7b236535e90300f974.png)

不允许在 var 的地方完成

![](img/8de3c1500242d70120fc2c9f7dfc72e1.png)

悬停以显示 javadoc

![](img/b0885f9e5b49ea96f1697d127d0e2cd7.png)

使用快速助手将 var 转换为适当的类型

![](img/b946b35069a5ac5b432ee6f6744dfa95.png)

使用快速助手将类型转换为变量

![](img/e2b82b4d25c4c82b32bfc8abb53117fb.png)

*Last updated: March 19, 2020*