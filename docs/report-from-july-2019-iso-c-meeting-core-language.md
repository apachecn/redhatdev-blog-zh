# 2019 年 7 月 ISO C++会议报告(核心语言)

> 原文：<https://developers.redhat.com/blog/2019/09/03/report-from-july-2019-iso-c-meeting-core-language>

2019 年夏季 [C++会议](https://isocpp.org/std/meetings-and-participation/upcoming-meetings)在德国科隆举行，距离我们上次在德国的会议已经过去了 10 年。像往常一样，Red Hat 派了我们三个人去参加会议:我参加了核心语言[工作组](https://isocpp.org/std/the-committee) (CWG)，图书馆(LWG)的 Jonathan Wakely 和 SG1 的 Thomas Rodgers(并行和并发)。

按照计划，在会议结束时，我们投票决定将 C++20 标准的草案发送给各个国家机构征求意见。最令人惊讶的是有人提议:

#### [从 C++20 中移除契约](http://wg21.link/p1823r0)

二月在科纳召开的会议上的分歧在这次会议上继续存在；由于在这一周内不可能达成共识，双方同意从 C++20 中移除契约特性，并在下一个标准中重新考虑它。为了继续讨论，成立了一个新的研究小组，由一个中立党派领导。

除此之外，草案的内容和预期的差不多。之前已经被 Evolution 批准的一些小的特性和修改在这次会议上通过了 Core 并进入了草案:

### 概念

#### [条件平凡的特殊成员函数](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0848r3.html)

这澄清了具有不同约束的构造函数和析构函数重载的语义，例如:

```
struct empty {}; 
template <typename T> class optional 
{ 
  bool engaged = false; 
  union { 
    empty _ = {};
    T value; 
  }; 

public: 
  constexpr optional() = default; // non-trivial due to default member initializer

  constexpr optional(optional const&) requires std::is_trivially_copy_constructible_v<T> = default; 
  constexpr optional(optional const& o): engaged(o.engaged) { if (o.engaged) new (&value) T(o.value); } 

  ~optional() requires std::is_trivially_destructible_v<T> = default; 
  ~optional() { if (engaged) value.~T(); } 

  // ... 
};
```

如果复制构造函数或析构函数的第一个重载的要求得到满足，它就有效地隐藏了约束较少的第二个重载，所以 optional <int>是简单的复制构造函数和简单的析构函数。</int>

#### [无约束模板模板参数和有约束模板](http://wg21.link/p1616r1)

通常，模板模板参数(TTP)必须至少与其模板模板参数一样专用。然而，这意味着没有办法编写一个可以接受任何模板而不受其约束的 TTP:

```
template <template <typename> typename TT> struct A {};
template <typename T> concept Any = true; 
template <Any> struct B; 
A<B> a; // previously error (TT is less constrained than B), now OK
```

正如我在讨论本文时所指出的，对于带有模板参数包的 TTP 来说，接受带有特定数量模板参数的实参模板已经是一个例外；这为约束添加了平行转义，因此没有任何约束的 TTP 将与受约束的参数模板相匹配。

#### [删除返回类型要求](http://wg21.link/p1452r2)

对于 requires-表达式中的复合需求，不清楚是否

```
  { expr } -> Type
```

ought to require the exact type or convertibility. So, use of a type here was removed for C++20, and users can choose for themselves with

```
  { expr } -> Same<Type>
```

or

```
  { expr } -> ConvertibleTo<Type>
```

### 模块

#### [减轻次要模块弊病](http://wg21.link/p1766r1)

第一，只有 typedef 名称用于链接目的的结构，例如:

```
typedef struct { int i; } foobar;
```

如果它们的任何成员受到这种链接的影响，例如成员函数或静态数据成员，都是脆弱的，我们最终使这样的成员格式不良。因为这是一个 C 兼容性问题，所以将它限制在 C 中实际出现的定义似乎是合理的。

其次，对于一个函数来说，在不同的翻译单元中有不同的缺省参数是不良的(不需要诊断)。

#### [放宽再出口重新定义限制](http://wg21.link/p1811r0)

允许在一个翻译单元中有一个实体的单一定义，即使一个定义也可以从一个导入的模块中获得，以减少由于包含多个遗留头文件而带来的麻烦。

#### [识别割台单元](http://wg21.link/p1703r1)

要求标题单元导入声明单独占一行，以 import 或 export import 开始，以便于部分预处理。

### 宇宙飞船(操作员<=>

#### 你什么时候真正使用运算符< = >？

如果提供了显式返回类型，则允许从运算符。

#### 宇宙飞船需要调整

阐明比较运算符合成以及重写的比较运算符如何参与重载决策。

### 康斯特布尔

#### [const expr](http://wg21.link/p1331r2)中的普通默认初始化

在 C++17 中，constexpr 函数中的变量必须初始化。本文删除了这一要求，取而代之的是在读取对象之前必须给它一个值。所以，

```
constexpr int f(int i) { 
  int j; // error in C++17, OK in C++20 
  j = i; // j now has a value 
  return j; // OK 
} 
constexpr int x = f(42); // OK

constexpr int g(int i) { 
  int j; // error in C++17, OK in C++20 
  return j; // ill-formed, no diagnostic required: will never produce a constant value 
}
constexpr int y = g(42); // error, non-constant initializer
```

#### [const expr 函数中未赋值的 ASM](http://wg21.link/p1668r1)

constexpr 函数中不再禁止的另一件事；它只是不产生一个常量值。

#### [添加 constinit 关键字](http://wg21.link/p1143r2)

要求对非常数变量进行常量(静态)初始化的新关键字:

```
constexpr int f(int x) { return x; } 
constinit int i = f(42); // OK 
constinit int j = f(i);  // error, i isn't constant
```

#### [更多 constexpr 容器](http://wg21.link/p0784r7)

允许在 constexpr 函数中使用 std::vector 这样的容器，方法是要求在常量求值期间省略分配/解除分配对(从 C++14 开始，动态求值代码中就允许这样做)。

#### [[[nodiscard("应该有理由")]]](http://wg21.link/p1301r4)

#### [[[nodiscard]]，用于构造函数](http://wg21.link/p1771r1)

填充[[nodiscard]]缺少的功能。

#### [聚合的类模板参数演绎(CTAD)](http://wg21.link/p1816r0)

#### [别名模板的 CTAD](http://wg21.link/p1814r0)

支持额外模板的 CTAD。还有一个提议是支持从继承的构造函数中推导，但是措辞没有及时准备好。

#### [使用枚举](http://wg21.link/p1099r5)

允许用户将限定范围的枚举的枚举数导入到当前范围，因此可以在没有显式范围的情况下命名它们。

```
  enum class A { a1 }; 
  using enum A; 
  int i = a1;
```

请注意，目前列表上有大量的[bike shedding](https://en.wiktionary.org/wiki/bikeshedding),所以在最终的 C++20 标准之前，语法可能会发生变化。

#### [转换为未知界限的数组](http://wg21.link/p0388r4)

允许在指针转换和引用绑定中从已知界限的数组转换为未知界限的数组。

```
  int arr[42];
  void f(int(&)[]); 
  void g(int(*)[]); 
  f(arr);    // now OK 
  g(&arr);   // now OK
```

#### [更含蓄的招式](http://wg21.link/p1825r0)

以各种方式放松了 C++11 在 throw 或 return 中隐式移动的条件:现在右值引用也将从隐式移动，隐式移动不限于构造函数对返回的表达式类型进行右值引用。

```
  struct Movable { Movable(Movable&&); }; 
  Movable f1(Movable &&r) { return r; } // now moves 
  struct Derived: Movable { }; 
  Movable f2(Derived d) { return d; }   // now moves 
  struct Proxy { Proxy(Movable); }; 
  Proxy f3(Movable m) { return m; }     // now moves
```

#### [贬低 volatile 的一些用途](http://wg21.link/p1152r4)

像++或+=这样同时包含可变左值的加载和存储的表达式是不推荐使用的，可变参数和返回类型也是如此。

#### [下标表达式中的弃用逗号](http://wg21.link/p1161r3)

作者希望能够在数组下标中使用逗号来分隔运算符[]的多个参数，而不是作为逗号运算符，因此现有的语义在 C++20 中被弃用，以便为未来标准中的新语义腾出空间。

```
  array[x,y]    // Deprecated, uses y as index/key
```

#### [memory _ order _ consume 与释放序列的交互](http://wg21.link/p0735r1)

对消费者语义做了一个相当微妙的调整，使它们在 ARM 上不那么容易被破坏。大多数实现将消费视为获取，因此这不会影响用户。

下一次会议将于 11 月在北爱尔兰的贝尔法斯特举行，届时我们将开始处理国家机构对 C++20 草案的意见。如果你有任何意见，请发给委员会成员转达。

*Last updated: August 30, 2019*