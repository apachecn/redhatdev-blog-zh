# 2020 年虚拟 ISO C++会议报告(核心语言)

> 原文：<https://developers.redhat.com/blog/2021/05/07/report-from-the-virtual-iso-c-meetings-in-2020-core-language>

2020 年的 C++标准化与早些年有了显著的不同。国际标准化组织(ISO)委员会的所有业务都是在虚拟环境下进行的，就像这次疫情期间的所有其他事情一样。本文总结了去年核心和演进工作组收到的 [C++](/topics/c) 标准化提案。

## 核心语言

C++核心工作组(CWG)已经在会议间隙举行了每月一次的 Zoom 电话会议；这就是我在《以前的时代》中遇到的变焦镜头。所以我们的过渡相当顺利。

在被取消的 11 月会议的某一天，我们确实在一个虚拟的全体委员会全体会议上移动了一些文件。

### (有符号)size_t 的文字后缀

这篇论文( [P0330](http://wg21.link/p0330) )在 2019 年 11 月的贝尔法斯特会议后就准备好了，但因为它是作为 C++23 的一个功能，所以我们不想在完成 C++20 之前提出来进行投票。这个提议使得编写一个`size_t`或`ptrdiff_t`类型的常量变得更加容易。这种做法很有用，例如，匹配一个`size()`函数的返回类型:

```
auto m = std::max (0, v.size()); // error, deduction mismatch int vs. size_t
auto m = std::max (0uz, v.size()); // OK, both arguments are size_t
```

这个提议的早期版本使用`t`表示`ptrdiff_t`，使用`z`表示`size_t`，就像`printf`转换说明符一样，但是最终版本使用`uz`表示`size_t`，使用`z`表示相应的有符号类型(通常与`ptrdiff_t`相同)。

### 字符和字符串中的数字和通用字符转义

本文( [P2029](http://wg21.link/p2029) )阐明了十六进制和八进制字符转义的处理，以标准化 GNU 编译器集合(GCC)行为，而不是(Microsoft Visual C++) MSVC 行为:也就是说，一个转义代码可以对应一个 UTF-8 代码单元，而不一定是一个完整的字符。

```
constexpr const char8_t c[] = u8"\xc3\x80"; // UTF-8 encoding of U+00C0 {LATIN CAPITAL LETTER A WITH GRAVE}
```

### 声明以及在哪里可以找到它们

在计划的 6 月会议的那一周，CWG 决定召开一整周的会议，每天两个小时，继续审查我们在布拉格开始审议的一份文件( [P1787](http://wg21.link/p1787) )，这是一份雄心勃勃的提案，旨在彻底修改声明范围和名称查找的措辞，从而解决 60 多个未决问题。我们还没看完整篇论文，一周就变成了三周。

本文的更改应该不会影响大量代码；许多修改是澄清，使措辞符合现行做法。有些是对大多数代码不依赖的极端情况的澄清，比如在一个 *conversion-type-id* 中的模糊查找。

一些更改允许以前格式不正确的代码:

*   *conversion-type-id* 被添加到来自[p 0634](http://wg21.link/p0634):

    ```
    template <class T> struct A { operator T::type(); }; // OK
    ```

    的仅类型上下文列表中
*   `::template`在仅类型上下文中也不是必需的:

    ```
    template <class T> auto f(T t) { return static_cast<T::X<int>>(t); } // OK
    ```

*   默认模板参数现在是完整的类上下文，就像默认函数参数:

    ```
    template <class T> struct A {
      template <int I = sizeof(t)> void g() { } // OK
      T t;
    };
    ```

一个更改可能会破坏少量的现有代码:因为在点(.)或箭头操作符(->)现在首先出现在对象的范围内，`.template`在`*dependent*.template X<...>`中是必需的，即使通过非限定查找可以找到 X 的定义:

```
template <int> struct X { void f(); };
template <class T> void g(T t) { t.X<2>::f(); } // error, needs .template
```

### 部分专业化的通用措辞

这篇文章( [P2096](http://wg21.link/p2096) )只是清理了标准中仍然只引用类的部分专门化的地方，所以这些地方也覆盖了可变的部分专门化。

### 打倒()！

这份文件( [P1102](http://wg21.link/p1102) )已经完成了核心审查，应该可以在下一次虚拟全体会议上投票表决了。本文建议对 lambda 语法进行更改，以避免在可变的 lambda 中需要()。

```
[x = 42] () mutable { ++x; }; // () are uselessly required in C++20
```

## 语言进化

C++ Evolution 工作组(EWG)一直在定期开会讨论未来的方向，但是作为一项政策，在面对面的会议召开之前，不会投票决定是否将论文提交给 CWG。最近，他们决定进行电子投票，以下文件将在二月份由 EWG 投票。

### 将上下文转换缩小到 bool

[CWG2039](http://wg21.link/cwg2039) 更改了`static_assert`的条件，以拒绝从大于 1 的整数到`bool`的收缩转换。这似乎是无意的，大多数编译器还没有实现它。因此本文( [P1401](http://wg21.link/p1401) )将特征改回，并对`if constexpr`的条件做了同样的更改:

```
static_assert (2, "two"); // OK again
```

### 强制申报订单布局

这篇文章( [P1847](http://wg21.link/p1847) )认为，因为没有编译器实际上使用不同的访问对数据成员进行重新排序，所以我们应该放弃这种权限。

### 如果 consteval

这个被提议的机制( [P1938](http://wg21.link/p1938) )很像`if (std::is_constant_evaluated())`，除了`if`的第一个块是一个立即函数上下文，允许使用依赖于当前(`constexpr`)函数参数的参数调用`consteval`函数。

### 使用 Unicode 标准附录 31 的 C++标识符语法

C++定期需要更改标识符中允许的 Unicode 字符列表；本文( [P1949](http://wg21.link/p1949) )提出采用实际 Unicode 标准所规定的集合，该集合自 C++11 以来已经稳定下来。

### 独立可选运算符新

本文( [P2013](http://wg21.link/p2013) )提出独立实现不需要提供可替换的`new`操作符的定义。

### 允许重复属性

C++11 属性不允许同一个属性在一个属性列表中重复出现，比如`[[nodiscard, nodiscard]]`。C 最近取消了这一限制，本文( [P2156](http://wg21.link/p2156) )建议对 C++也这么做。

### lambda 表达式的属性

本文( [P2173](http://wg21.link/p2173) )建议允许属性出现在 lambda-introducer 之后，例如:

```
[] [[nodiscard]] (int x) { return x; }
```

### 移除垃圾收集支持

这篇文章( [P2186](http://wg21.link/p2186) )指出，C++11“对垃圾收集的最小支持”没有实现，实际垃圾收集的几个 C++实现没有与 C++11 特性交互，因此建议删除它。

### 混合字符串文字连接

本文( [P2201](http://wg21.link/p2201) )建议将具有不同编码前缀的字符串文字的连接从有条件支持改为格式不良。

### 在线拼接前修剪空白

本文( [P2223](http://wg21.link/p2223) )建议忽略行尾反斜杠(\)后的任何空白，原始字符串除外。

## 结论

各工作组在年初几乎一直在开会，并在原计划 2 月份开会的那一周举行了另一次虚拟全体会议。目前的暂定计划是在 2021 年底之前继续举行虚拟会议，并在 2022 年 2 月再次举行面对面会议。

有关 C 和 C++的更多信息，请访问[红帽开发者主题页面](/topics/c)。

*Last updated: October 14, 2022*