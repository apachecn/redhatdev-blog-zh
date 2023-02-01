# GCC 10 中新的 C++特性

> 原文：<https://developers.redhat.com/blog/2020/09/24/new-c-features-in-gcc-10>

GNU 编译器集合(GCC) 10.1 于 2020 年 5 月发布。像其他 GCC 版本一样，这个版本带来了许多添加、改进、错误修复和新特性。Fedora 32 已经将 GCC 10 作为系统编译器，但也有可能在其他平台上尝试 GCC 10(例如，参见 godbolt.org)。[红帽企业 Linux](https://developers.redhat.com/products/rhel/overview) (RHEL)用户将在红帽开发者工具集(RHEL 7)中获得 GCC 10，或者红帽 GCC 工具集(RHEL 8)。

本文主要关注 GCC 编译器中我花了大部分时间的部分:C++前端。我的目标是展示 C++应用程序程序员可能感兴趣的新特性。注意，我没有讨论 C++语言本身的发展，尽管一些语言更新与编译器更新重叠。我也不讨论 GCC 10 附带的标准 C++库的变化。

我们在 GCC 10 中实现了许多 C++20 提案。为了简洁起见，我不会非常详细地描述它们。GCC 10 中默认的方言是`-std=gnu++14`；要启用 C++20 特性，请使用`-std=c++20`或`-std=gnu++20`命令行选项。(注意，后一个选项允许 GNU 扩展。)

## C++概念

虽然以前版本的 GCC (GCC 6 是第一个)最初实现了 [C++概念](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0734r0.pdf)，但 GCC 10 更新了概念以符合 C++20 规范。此更新还改进了编译时间。后续补丁改进了与概念相关的诊断。

在 C++模板参数中，`typename`表示任何类型。但是大多数模板必须以某种方式进行约束；例如，您希望只接受具有某些属性的类型，而不是任何类型。未能使用正确的类型通常会导致可怕而冗长的错误消息。在 C++20 中，您可以通过使用一个*概念*来约束一个类型，这是一个编译时谓词，它指定了可以应用于该类型的一组操作。使用 GCC 10，您可以定义自己的概念(或者使用在`<concepts>`中定义的概念):

```
#include <type_traits>
// Require that T be an integral type.
template<typename T> concept C = std::is_integral_v<T>;

```

然后像这样使用它:

```
template<C T> void f(T) { }
void g ()
{
  f (1); // OK
  f (1.2); // error: use of function with unsatisfied constraints
}

```

从 GCC 10 开始，C++编译器也支持[约束自动](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1141r2.html)。使用我们上面的概念，你现在可以写:

```
int fn1 ();
double fn2 ();

void h ()
{
  C auto x1 = fn1 (); // OK
  C auto x2 = fn2 (); // error: deduced initializer does not satisfy placeholder constraints
}
```

## 协同程序

GCC 10 支持[无堆栈函数](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0912r5.html)，这些函数可以被暂停，稍后可以恢复，而不会丢失它们的状态。这个特性让我们可以异步执行顺序代码。它需要`-fcoroutines`命令行选项。

### `constexpr`函数中未赋值的内嵌汇编

像这样的代码现在可以编译:

```
constexpr int
foo (int a, int b)
{
  if (std::is_constant_evaluated ())
    return a + b;
  // Not in a constexpr context.
  asm ("/* assembly */");
  return a;
}
```

详见[提案](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1668r1.html)。

### 数组下标表达式中的逗号表达式

这种类型的表达式现在[已被弃用](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1161r3.html)，所以 GCC 10 对这样的代码发出警告:

```
int f (int arr[], int a, int b)
{
  return arr[a, b];
}

```

然而，只有顶级逗号被弃用，所以`arr[(a, b)]`编译时没有警告。

### 结构化绑定

GCC 10 改进并且[扩展了](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1091r3.html)结构化绑定。例如，现在可以标记它们`static`:

```
struct S { int a, b, c; } s;
static auto [ x, y, z ] = s;

```

这个例子不能用 GCC 9 编译，但是可以用 GCC 10 编译。

## `constinit`关键字

GCC 10 使用 [C++20 说明符](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1143r2.html) `constinit`来确保(静态存储持续时间)变量由常量初始化器初始化。

这可能会缓解静态初始化顺序失败的问题。然而，该变量不是恒定的:在初始化发生后可以修改它。考虑:

```
constexpr int fn1 () { return 42; }
int fn2 () { return -1; }
constinit int i1 = fn1 (); // OK
constinit int i2 = fn2 (); // error: constinit variable does not have a constant initializer

```

GCC 不支持 Clang 的`require_constant_initialization`属性，所以你可以在早期的 C++模式中使用`__constinit`，作为一个扩展，来获得类似的效果。

## `volatile`的不推荐使用

包含加载和存储左值的表达式，如`++`或`+=`，被[否决](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1152r4.html)。`volatile`-限定的参数和返回类型也不推荐使用，所以 GCC 10 会警告:

```
void fn ()
{
  volatile int v = 42;
  // Load + store or just a load?
  ++v; // warning: deprecated
}

```

### 转换为未知界限的数组

现在[允许](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0388r4.html)转换为未知界限的数组，所以下面的代码可以编译:

```
void f(int(&)[]);
int arr[1];
void g() { f(arr); }
int(&r)[] = arr;

```

**注意**:从未知边界的数组到另一个方向的*转换，目前 C++标准不允许。*

### `constexpr new`

这个[特性](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0784r7.html)允许在编译时在`constexpr`上下文中动态分配内存:

```
constexpr auto fn ()
{
  int *p = new int{10};
  // ... use p ...
  delete p;
  return 0;
}

int main ()
{
  constexpr auto i = fn ();
}

```

注意，编译时在`constexpr`上下文中分配的存储空间也必须在编译时释放。并且，鉴于`constexpr`不允许未定义的行为，`use-after-free`是一个编译时错误。这个`new`表情也不能丢。这一特性为`<vector>`和`<string>`等`constexpr`标准集装箱铺平了道路。

### `[[nodiscard]]`属性

`[[nodiscard]]`属性现在支持可选参数，比如:

```
[[nodiscard("unsafe")]] int *fn ();

```

详见[提案](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1301r4.html)。

## CTAD 扩展公司

C++20 类模板参数演绎(CTAD)现在也适用于[别名模板](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1814r0.html)和[聚合](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1816r0.pdf):

```
template <typename T>
struct Aggr { T x; };
Aggr a = { 1 }; // works in GCC 10

template <typename T>
struct NonAggr { NonAggr(); T x; };
NonAggr n = { 1 }; // error: deduction fails

```

## 聚集的带括号初始化

现在，您可以使用一个[带括号的值列表](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0960r3.html)，比如`(1, 2, 3)`，来初始化一个聚合。行为类似于`{1, 2, 3}`，但是在带括号的初始化中，以下例外情况适用:

*   允许收缩转换。
*   不允许使用指示符(如`.a = 10`)。
*   绑定到引用的临时对象没有延长其生存期。
*   没有大括号省略。

这里有一个例子:

```
struct A { int a, b; };
A a1{1, 2};
A a2(1, 2); // OK in GCC 10 -std=c++20
auto a3 = new A(1, 2); // OK in GCC 10 -std=c++20

```

### 在`constexpr`上下文中的普通默认初始化

这种用法现在在 C++20 中被允许使用。因此，`constexpr`构造函数不一定要初始化所有的字段(但是读取未初始化的对象当然还是被禁止的):

```
struct S {
  int i;
  int u;
  constexpr S() : i{1} { }
};

constexpr S s; // error: refers to an incompletely initialized variable
S s2; // OK

constexpr int fn (int n)
{
  int a;
  a = 5;
  return a + n;
}

```

## `constexpr dynamic_cast`

在`constexpr`上下文中，你现在可以在编译时评估一个`dynamic_cast` [。常量表达式中的虚函数调用已经被允许了，所以这个提议使得使用`constexpr` `dynamic_cast`变得有效，就像这样:](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1327r1.html)

```
struct B { virtual void baz () {} };
struct D : B { };
constexpr D d;
constexpr B *b = const_cast<D*>(&d);
static_assert(dynamic_cast<D*>(b) == &d);

```

以前，`constexpr` `dynamic_cast`需要运行时调用 C++运行时库定义的函数。类似地，多态的`typeid`现在也允许出现在`constexpr`上下文中。

**注意**:GCC 10 不支持 C++20 模块；他们仍在进行中(这里是[的一个相关提案](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1103r3.pdf))。我们希望将它们纳入 GCC 11。

## 其他更新

在非 C++20 新闻中，C++编译器现在检测到在`constexpr`求值中修改常量对象，这是未定义的行为:

```
constexpr int
fn ()
{
  const int i = 5;
  const_cast<int &>(i) = 10; // error: modifying a const object
  return i;
}
constexpr int i = fn ();

```

GCC 还处理构造中的常量对象被修改的情况，在这种情况下不会发出错误。

### 收缩转换

收缩转换在某些上下文中是无效的，例如列表初始化:

```
int i{1.2};

```

GCC 10 能够在更多无效的上下文中检测收缩，例如,`switch`语句中的 case 值:

```
void g(int i)
{
  switch (i)
  case __INT_MAX__ + 1u:;
}

```

### `noexcept`说明符

GCC 10 正确地将`noexcept`说明符视为*完整类上下文*。与成员函数体、默认参数和非静态数据成员初始化器一样，您可以在类体的后面声明成员函数的`noexcept`说明符中使用的名称。以下有效的 C++代码不能用 GCC 9 编译，但可以用 GCC 10 编译:

```
struct S {
  void foo() noexcept(b);
  static constexpr auto b = true;
};

```

### `deprecated`属性

现在可以在名称空间上使用`deprecated`属性:

```
namespace v0 [[deprecated("oh no")]] { int fn (); }

void g ()
{
  int x = v0::fn (); // warning: v0 is deprecated
}

```

GCC 9 编译了这段代码，但是忽略了属性，而 GCC 10 正确地警告了不推荐使用的名称空间中的实体。

### 缺陷报告解决方案

我们在 GCC 10 中解决了几个缺陷报告(DRs)。一个例子是 [DR 1710](http://wg21.link/cwg1710) ，它说当我们命名一个类型时，`template`关键字是可选的。因此，下面的测试编译 GCC 10 时没有错误:

```
template<typename T> struct S {
  void fn(typename T::template B<int>::template C<int>);
  void fn2(typename T::B<int>::template C<int>);
  void fn3(typename T::template B<int>::C<int>);
  void fn4(typename T::B<int>::C<int>);
};

```

另一个有趣的 DR 是 [DR 1307](http://wg21.link/cwg1307) ，它阐明了当根据数组大小初始化列表选择一个更好的候选者时，重载决策应该如何表现。考虑以下测试:

```
void f(int const(&)[2]);
void f(int const(&)[3]) = delete;

void g()
{
  f({1, 2});
}

```

GCC 9 拒绝这个测试，因为它不能决定哪个候选人更好。然而，似乎很明显，第一个候选人是最好的(别管那部分；删除的函数参与重载决策)。GCC 10 选择这个选项，让代码编译。

**注**:您可以在 GCC 页面的 [C++缺陷报告支持中找到总体缺陷解决状态。](https://gcc.gnu.org/projects/cxx-dr-status.html)

## 结论

在 GCC 11 中，我们计划完成剩余的 C++20 特性。关于迄今为止的进展，请参见[GCC 中的 C++标准支持](https://gcc.gnu.org/projects/cxx-status.html)页面上的 [C++2a 语言特性](https://gcc.gnu.org/projects/cxx-status.html#cxx2a)表。GCC 11 也会把默认方言切换到 C++17(已经发生了)。同时，请不要犹豫[提交 bug](https://gcc.gnu.org/bugs/)，帮助我们把 GCC 做得更好！

## 承认

我要感谢我在 Red Hat 的同事，他们让 GNU C++编译器变得更好，特别是 Jason Merrill、Jakub Jelinek、Patrick Palka 和 Jonathan Wakely。

*Last updated: September 23, 2020*