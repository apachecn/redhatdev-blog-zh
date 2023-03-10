# 2018 年 3 月 ISO C++会议行程报告(核心语言)

> 原文：<https://developers.redhat.com/blog/2018/04/02/march-2018-iso-c-meeting-trip-report-core-language>

今年三月的 C++ ISO 标准会议又回到了佛罗里达州的杰克逊维尔。像往常一样，红帽公司派了我们三个人去参加会议:托瓦尔·里格尔、托马斯·罗杰斯和我。乔纳森·韦克利通过扬声器出席了会议。本周初有 121 人出席全体会议。

这次会议主要是关于 C++20 的新特性，特别是何时以及如何将技术规范合并到标准草案中。在核心语言中，试图让 C++20 的是概念(已经部分合并)、协程和模块。围绕这三个问题进行了大量讨论。

### 概念

周一晚上，许多概念讨论都在晚间进行。会上讨论了几篇论文，主要是试图解决阻止来自[概念 TS](http://wg21.link/n4377) 的“自然”或“简洁”语法与 TS 的其余部分[合并的问题。](http://wg21.link/p0734)

[两个](http://wg21.link/p0791) [建议](http://wg21.link/p0807)解决了声明受约束的模板参数时的视觉模糊性:当你看到`template <MyConcept X>`时，模板参数`X`的种类取决于`MyConcept`的定义。提案建议我们将概念视为 C++14 模板参数语法的前缀“形容词”，例如`MyTypeConcept typename X`或`MyNumericConcept int Y`。对这个方向没有太多热情。

[另一篇论文](http://wg21.link/p0745)提出了一种“就地”语法来代替 TS 的约束类型说明符:当声明中的概念名称后面跟有包含一个或多个名称的大括号时，这些名称被声明为模板类型参数。所以，

```
Integral{T} fn(T, Integral{U});
```

相当于:

```
template <typename T, typename U>
 requires Integral<T> && Integral<U>
 T fn(T,U);
```

这个语法来源于 TS 中的模板介绍语法，但是可以在任何地方使用，而不仅仅是在声明的开始。人们对这项提议热情很高。

也有很多人支持[采用 TS](http://wg21.link/p0956) 的语法，并做了两处调整。第一个调整是改变在声明中多次使用的概念名称的含义。例如:

```
Integral fn(Integral, Integral)
```

在 TS 中，`Integral`的所有使用都命名了同一个类型。这次会议的共识是它们应该是不同的类型，就像泛型 lambdas 中的`auto`参数一样。

另一个调整是一个尚未指定的语法，它表明声明是一个模板，许多人仍然对此有强烈的感觉。这个小组发现上面的“就地”语法已经足够清楚了，但是不赞成把大括号变成可选的。

### 协同程序

在这次会议上，将[协程 TS](http://wg21.link/n4723) 合并到标准草案中是一个重要的推动因素，但是在[的一篇论文](http://wg21.link/p0973)中也提出了一些问题，关于 TS 中的设计在他们希望使用该功能的方式上存在各种问题。经过大量的讨论，主要的症结似乎是在协程使用中多久可以优化一次堆分配。与会者普遍同意，该文件的作者将在下次会议上提出修改意见。一个[提议](http://wg21.link/p0914)也被采用来使协程 promise 类构造函数更容易访问协程本身的参数。

### 模块

在这次会议上提出了许多对 ts 的调整。[](http://wg21.link/p0906)[的一些](http://wg21.link/p0923)变化继续阐明了作为导出另一个实体的结果，导出某些实体的什么语义属性。[其他](http://wg21.link/p0947)涉及到宏和遗留头文件的处理，这是发布的 TS 中的一个迁移问题。预计在下次会议上将有一项提案将这些调整合并到模块 TS [工作文件](http://wg21.link/n4720)中。

### 新功能

在这次会议上，标准草案中增加了几个较小的新功能:

*[p 0840](http://wg21.link/p0840),`[[no_unique_address]]`属性，允许用户使用非静态数据成员而不是基来获得空基优化。
* [P0780](http://wg21.link/p0780) ，允许可变λ初始捕获:

```
template<class... Args> void f(Args... args) {
  [...xs=std::move(args)] { return g(xs...); }();
}
```

* [P0634](http://wg21.link/p0634) ，在较少的地方需要“typename”关键字，例如:

```
template<class T> T::R f(); // OK, return type of a function declaration at global scope
```

* [P0479](http://wg21.link/p0479) ，增加`[[likely]]`和`[[unlikely]]`属性指导优化。

还有几个功能提案通过了一些核心审查:

* [P0722](http://wg21.link/p0722) ，销毁`operator delete`，以便更好地处理与固定缓冲区一起分配的类。

* [P0194](http://wg21.link/p0194) ，静态(编译时)反射。一段时间以来，反思一直是委员会研究的一个领域，几个相互竞争的提案仍在审议中，但这是第一次有人走到 CWG。

* [P0542](http://wg21.link/p0542) ，契约式编程。很长时间以来，合同也是一个研究领域，但看起来他们终于接近进入语言了。这将允许代码以一种可以在运行时可选地检查的方式正式指定函数的前置条件和后置条件:

```
void push(int x, queue & q)
  [[expects: !q.full()]]
  [[ensures: !q.empty()]]
{
  //...
  [[assert: q.is_valid()]];
  //...
}
```

* [P0892](http://wg21.link/p0892) ，`explicit(bool)`。这个提议为`explicit`添加了一个可选的表达式，允许库更容易地根据模板参数显式或不显式地创建成员函数。

* [P0482](http://wg21.link/p0482) 建议增加`char8_t`与`char16_t`和`char32_t`搭配，以区别 UTF-8 字符和现有字符类型。

* [P0784](http://wg21.link/p0784) 提议对`constexpr`进行扩展，以允许在常量表达式中使用标准容器，包括允许在常量表达式求值中匹配`new`和`delete`表达式，如果它们可以根据 [C++14 规则](http://wg21.link/n3664)省略的话。

### 核心问题

我们还考虑了针对该标准提出的一些问题。一些更有趣的例子:

* [Issue 2256](http://wg21.link/cwg2256) :平凡类型的对象的生存期应该遵循与非平凡类型的对象相同的规则，而不是与它们的存储持续时间相关联。GCC 优化已经朝着这个方向发展。

* [问题 2219](http://wg21.link/cwg2219) :优化器看到展开到达 catch 处理程序将总是调用`exit`，所以它移除了 catch 处理程序，所以 EH 运行时看不到处理程序并调用`terminate`而不是展开，所以我们实际上并没有调用`exit`。我们讨论了一会儿，但没有决定答案。现在我在想，把这样一个训导员变成一个`catch(...)`是否可行。

下一次会议将于 6 月在瑞士的拉普斯维尔-约纳举行。