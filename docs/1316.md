# Java 中的接口

> 原文：<https://developers.redhat.com/blog/2017/11/10/interfaces-in-java>

Java 中的接口通常是一种允许多个类共享多个方法和常量的机制。它也是在 Java 中实现多态性的最佳机制之一。

因此，如果你是那种在 Java 8 到来之前就非常熟悉接口的人，那么发现一些接口现在可以在 Java 8 中做的很酷的事情是很有意义的。让我们开始吧。

Java 8 之前的接口通常只包含抽象方法和常量，任何实现接口的类都必须实现它的所有方法或者被声明为抽象的。嗯，这是 Java 8 之前。有了 Java 8，这个已经升级了。

接口现在可以包含具有实现的方法。是的，真的。这种拥有接口实现权限的方法称为默认方法。这些允许实现接口的所有类默认实现一个方法，并为实现类提供重写该方法的机会(如果它们愿意的话)。

```
public interface Bank{

default String getBankName(){

return “Azibit Bank”;

}

}
```

请注意默认方法的语法。它必须以 default 关键字开头。

有了这个介绍，每个 Java 开发人员主要关心的是这个新概念如何管理来自不同接口的多重继承，或者一个类和一个接口是否有相同的方法。这种担心是预料到的，所以当不止一个接口有相同的方法并且一个类实现了这两个方法时，下面的规则适用。

1.  方法的类实现优先于默认方法。因此，如果类已经有了与接口相同的方法，那么实现的接口的默认方法就不会生效。
2.  但是，如果两个接口实现相同的默认方法，那么就会有冲突。
3.  如果一个接口继承另一个接口，并且两个接口都实现默认方法，则实现类将使用子接口的默认方法。
4.  此外，可以使用 super 从实现类内部显式调用接口默认方法。例如 Interface.super.defaultMethod()。

此外，Java 8 在接口中引入了静态方法。因此，接口现在可以创建静态方法，并从任何地方引用这些静态方法，只需调用接口名，后跟方法名。例如:InterfaceName.staticMethodName()。

要创建静态方法，您需要在创建方法时添加 static 关键字。

```
public interface Test{ 
static String getStaticNumber(){ 
return 1; 
} 
}
```

参考:Java 完整参考，作者赫伯特·席尔德

* * *

[**加入红帽开发者计划**](https://developers.redhat.com/?intcmp=70160000000xZNgAAM) **(免费)并获得相关的备忘单、书籍和产品下载。**

* * *

**点击这里下载 [OpenJDK](https://developers.redhat.com/products/openjdk/download/?intcmp=7016000000124gfAAA) ，这是一个 Java 平台的免费开源实现。**

*Last updated: November 9, 2017*