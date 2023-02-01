# 使用 GCC 11 将您的代码移植到 C++17

> 原文：<https://developers.redhat.com/articles/2021/08/06/porting-your-code-c17-gcc-11>

[GNU 编译器集合](https://gcc.gnu.org/) (GCC)，是 [Fedora](https://getfedora.org/) 和 [Red Hat Enterprise Linux](/products/rhel) 、[等](https://gcc.gnu.org/gcc-11/) [GNU/Linux](/topics/linux) 发行版上的标准编译器，于 2021 年 4 月从 C++ 的版本 14 迁移到版本 17。因此，现在默认使用的是`-std=gnu++17`命令行选项。

[C++17 带来了许多新特性](https://gcc.gnu.org/projects/cxx-status.html#cxx17)，但也摒弃、移除或改变了某些构造的语义。本文着眼于您在切换到 GCC 11 时可能面临的一些问题。记住，通过指定`-std=gnu++14`选项，总是可以使用以前版本的 C++。此外，本文只涉及核心语言；我们不会讨论标准 C++库中已弃用或移除的特性(比如`auto_ptr`)。为了更广泛的了解，我鼓励大家去看看 T4 的论文《C++14 和 C++17 之间的变化》。有关切换到使用 GCC 11 的更多信息，请参见我们的上游文档[移植到 GCC 11](https://gcc.gnu.org/gcc-11/porting_to.html) 。

## 在 C++17 中被移除

我们将从 C++17 中删除的内容开始:Trigraphs、`register`关键字和`bool`类型的增量。

### 三字母

在 C++中，*三字符*是以`??`开头的三个字符的序列，可以表示单个标点字符。例如，`\`可以写成`??/`。这是有历史原因的:C 和 C++使用特殊字符，如`[`和`]`，这些字符在 ISO 646 字符集中没有定义(因此一些键盘缺少这些键)。这些字符在 ISO 表中的位置可能被国家 ISO 646 字符集中的不同字符占用，例如用`¥`代替`\`。

三字符意味着允许程序员输入键盘上没有的字符。但在实践中，三连字符很可能只是偶然被使用，所以它们在 C++17 中被[移除了。删除允许你玩“可爱”的游戏，如下面的测试。你能看出它为什么有效吗？](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4086.html)

```
bool cxx_with_trigraphs_p () {
  // Are we compiling in C++17??/
  return false;
  return true;
}
```

如果出于某种原因，你仍然需要在 C++17 中使用三字符(事实上，有些代码库[仍然使用三字符](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2910.pdf))，GCC 提供了`-trigraphs`命令行选项。

### register 关键字

`register`关键字在 C++11 中与 [CWG 809](https://wg21.link/cwg809) 一起被弃用，因为它没有实际作用。C++17 通过[彻底删除关键字](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/p0001r1.html)进一步清理了这一点(尽管它仍然保留供将来使用)。因此，如下代码会导致 GCC 11 默认发出警告，并在使用`-pedantic-errors`选项时发出错误:

```
void f () {
  register int i = 42; // warning: ISO C++17 does not allow 'register' storage class specifier
  int register; // error
}
```

在 C++14 中，您可以使用`-Wregister`命令编译器发出警告，这应该有助于您将代码迁移到 C++17。注意，GCC 仍然接受 GNU [显式寄存器变量](https://gcc.gnu.org/onlinedocs/gcc/Global-Register-Variables.html#Global-Register-Variables)扩展，没有警告:

```
register int g asm ("ebx");
```

### bool 类型的增量

在 C++中，`bool`类型的对象从来不支持`--`操作符。但是`bool`上的`++`原本是有效的，在 C++98 中被弃用但没有被移除。根据 C++17 的精神，后增量和前增量现在是被禁止的，所以 GCC 将对使用它们的代码给出一个错误:

```
template<typename T>
void f() {
  bool b = false;
  b++; // error: use of an operand of type 'bool' in 'operator++' is forbidden in C++17
  T t{};
  t++; // error: use of an operand of type 'bool' in 'operator++' is forbidden in C++17
}

void g() {
  f<bool>();
}
```

不使用`++`，根据具体情况简单使用`b = true`或`b |= true`。

## 异常规格变更

与异常相关的关键字已经被添加到 C++的各个版本中。C++17 引入了对`noexcept`规范的更改，同时删除了动态异常规范。

### 异常规范现在是类型系统的一部分

C++1 1 在 [N3050](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2010/n3050.html) 中增加了`noexcept`异常规范。从 C++17 开始，`noexcept`就成了类型的一部分。因此，以下两个函数在 C++14 中具有相同的类型，但在 C++17 中具有不同的类型:

```
int foo () noexcept;
int bar ();
```

然而，这并不意味着`noexcept`是函数签名的一部分。因此，不能像下面这样重载函数:

```
int baz();
int baz() noexcept; // error: different exception specifier (even in C++11)
```

另一个后果是，在 C++17 中，指向可能抛出的函数的指针不能转换为不能抛出的函数:

```
void (*p)();
void (*q)() noexcept = p; // OK in C++14, error in C++17
```

这种变化也会影响模板参数。下面的程序不能在 C++17 中编译，因为编译器为模板参数`T`推导出两种冲突的类型:

```
void g1 () noexcept;
void g2 ();
template<typename T> int foo (T, T);
void f() {
  foo (g1, g2); // error: void (*)() noexcept vs void (*)()
}
```

有意思的是，这个改动最早是 20 多年前在 [CWG 92](https://wg21.link/cwg92) 讨论的，最后被 [P0012R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/p0012r1.html) 采纳。

### 移除动态异常规范

C++11 在 [N3051](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2010/n3051.html) 中弃用了动态异常规范，C++17 在 [P0003R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0003r5.html) 中将其全部移除。例外的是`throw()`在 C++17 中继续工作，与`noexcept(true)`等效，虽然在 C++20 中已经被移除。而且，在 C++17 中，函数`std::unexpected`被移除了。过去，如果用`throw()`修饰的函数确实抛出了异常，就会调用这个函数。在 C++17 中，`std::terminate`被改为调用。

C++17 代码不能使用动态异常规范，应该用`noexcept`替换`throw()`。

以下示例可能有助于澄清用法:

```
void fn1() throw(int); // error in C++17, warning in C++14
void fn2() noexcept; // OK since C++11
void fn3() throw(); // deprecated but no warning in C++17
void fn4() throw() { throw; }
// In C++14, calls std::unexpected which calls std::unexpected_handler
// (which is std::terminate by default).
// In C++17, calls std::terminate directly.
```

## 新模板模板-参数匹配

C++17 提案 [P0522R0:匹配模板 template-arguments 排除兼容模板](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0522r0.html)，修复了 [DR 150](https://wg21.link/cwg150) ，在 GCC 7 中的`-fnew-ttp-matching`选项中实现，但默认关闭。因为 GCC 11 默认为 C++17，所以新的行为是默认启用的。

在旧的行为中，模板 template-parameter 的模板 template-argument 必须是具有参数列表的模板，该参数列表与模板参数的参数列表中的相应参数完全匹配。通过考虑匹配中模板模板参数的默认模板参数，新的行为不太严格。

对于标准库中的一些模板来说，这种改变是有问题的。例如，`std::deque`是一个具有以下默认模板参数的模板:

```
template<typename T, typename Allocator = std::allocator<T>>
class deque;
```

因此，以下代码仅适用于新的 GCC 11 行为:

```
#include <deque>

template <template <typename> class>
void fn() {}
template void fn<std::deque>();
```

解决方法是调整声明，使参数需要两个类型参数:

```
#include <deque>

template <template <typename, typename> class>
void fn() {}
template void fn<std::deque>();
```

但是，新行为也可能导致在旧行为中工作的代码停止编译，并出现错误:

```
template <int N, int M = N> class A { };
template <int N, int M> void fn(A<N, M>) {}
template <int N, template <int> typename T> void fn(T<N>);

void g ()
{
  A<3> a;
  fn (a); // ambiguous in C++17
}
```

原因是在新行为中，`A`被认为是`T`的有效参数。因此，这两个函数模板都是有效的候选对象，并且因为没有一个比另一个更专门化，所以对`fn`的函数调用变得不明确。

通过使用`-fno-new-ttp-matching`，甚至在 C++17 模式下也可以恢复到旧的行为。

## 静态 constexpr 和 consteval 类成员隐式内联

引入内联变量( [P0386R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0386r2.pdf) )的 C++17 提案对规范的[dcl.constexpr]部分进行了如下更改:“用`constexpr`或`consteval`说明符声明的函数或静态数据成员是隐式的内联函数或变量。”

因此，下例中的成员变量`A::n`是 C++17 中的一个定义。在 C++14 中，注释中标记为`#2`的声明是必需的。在 C++17 中，可以删除 `#2`声明:

```
struct A {
  static constexpr int n = 5; // #1, definition in C++17, declaration in C++14
};

constexpr int A::n; // #2, definition in C++14, deprecated redeclaration in C++17

auto g()
{
  return &A::n; // ODR-use of A::n -- needs a definition
}
```

## 对评估订单的更改

C++17 [P0145R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0145r3.pdf) 明确了各种表达式的求值顺序。如提案所述，以下表达式的计算方式是在`b`之前先计算`a`:

*   `a.b`
*   `a->b`
*   `a->*b`
*   `a (b1, b2, b3)`
*   `b op= a`
*   `a[b]`
*   `a << b`
*   `a >> b`

这些规则导致了 GCC 编译器中的一些变化，某些代码在 C++14 和 C++17 中可能会有不同的行为，如下面的测试所示。可以使用`-fstrong-eval-order={all|some|none}`编译时选项有选择地调整编译器行为，其中`all`是 C++17 中的默认值，`some`是 C++14 中的默认值。

```
int fn ()
{
  int ar[4]{ };
  int i = 0;
  ar[i++] = i;
  return ar[0]; // returns 0 in C++17, 1 in C++14
}

int fn2 ()
{
  int x = 2;
  return x << (x = 1, 2); // returns 8 in C++17, 4 in C++14
}

int fn3 ()
{
  int x = 6;
  return x >> (x = 5, 1); // returns 3 in C++17, 2 in C++14
}
```

前面提到的 [P0145R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0145r3.pdf) 论文还定义了当一些受影响的操作数重载并使用运算符语法时的求值顺序:它们遵循为内置运算符规定的顺序。

## 保证复制省略

C++17 要求有保证的复制省略，这意味着在某些情况下(例如当初始化器和目标的类型相同时)，复制或移动构造函数调用将被完全省略，即使调用有副作用。这意味着，从理论上讲，如果某个东西依赖于被实例化的构造函数，例如，通过复制函数参数，程序现在可能会失败，因为在 C++17 中构造函数可能不会被实例化。

即使在 C++14 模式下，GCC 也已经执行了复制/移动省略作为一种优化，所以这种失败在实践中不太可能发生。但是，不同之处在于，在 C++17 中，编译器不会对省略的构造函数执行访问检查，因此以前没有编译的代码现在可以编译了，如下面的代码片段所示:

```
class A {
  int i;
public:
  A() : i{42} {}
private:
  A(const A &);
};

struct B {
  A a;
  B() : a(A()) {} // OK in C++17, error in C++14
};
```

## 摘要

我希望这些注释对迁移到 GCC 11 的开发人员有用。与每个主要版本一样，我们添加了新的警告，这些警告可能会在切换编译器时出现。如果您在这些新警告中发现了一个错误，请不要犹豫，打开一个新的问题报告，如 [GCC 错误页面](https://gcc.gnu.org/bugs/)中所述。

*Last updated: January 6, 2023*