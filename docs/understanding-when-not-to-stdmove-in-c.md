# 理解在 C++中何时不要 std::move

> 原文：<https://developers.redhat.com/blog/2019/04/12/understanding-when-not-to-stdmove-in-c>

C++11 中引入的最重要的概念之一是 *[移动语义](https://en.cppreference.com/w/cpp/language/move_constructor)。*移动语义是一种避免昂贵的深度复制操作，并用更便宜的移动操作替换它们的方法。本质上，你可以把它想成是把深层拷贝变成浅层拷贝。

Move 语义附带了几个或多或少相关的特性，比如 [*【右值引用】*](https://en.cppreference.com/w/cpp/language/reference#Rvalue_references)[*x 值*](https://en.cppreference.com/w/cpp/language/value_category)[*转发引用*](https://en.cppreference.com/w/cpp/language/reference#Forwarding_references)[*完美转发*](https://en.cppreference.com/w/cpp/utility/forward) 等等。标准 C++库获得了一个名为`std::move`的函数模板，尽管名为 T0，但它并不移动任何东西。`std::move`仅仅将其参数转换为右值引用以允许移动它，但不保证移动操作。例如，我们可以使用`std::move`编写一个更有效的`swap`版本:

```
template<typename T>
void swap(T& a, T& b)
{
  T t(std::move (a));
  a = std::move (b);
  b = std::move (t);
}
```

这个版本的 swap 包含一个 move 构造和两个 move 赋值，不涉及任何深度复制。一切都好。然而，`std::move`必须谨慎使用；轻率地使用它可能会导致性能下降，或者仅仅是多余的，影响代码的可读性。幸运的是，编译器有时可以帮助找到`std::move`的这种错误用法。在本文中，我将介绍我为 GCC 9 实现的两个新警告，它们处理`std::move`的不正确用法。

### -Wpessimizing-移动

当返回与函数返回类型相同的类类型的局部变量时，如果我们返回的变量是非易失性自动对象并且不是函数参数，编译器可以自由地省略任何复制或移动(即执行 [*复制/移动省略*](https://en.cppreference.com/w/cpp/language/copy_elision) )。在这种情况下，编译器可以直接在其最终目的地(即，在调用者的堆栈框架中)构造对象。即使移动/复制构造有副作用，编译器也可以自由地执行这种优化。此外，C++17 说复制省略在某些情况下是强制性的。这就是我们所说的*命名返回值优化* (NRVO)。(注意，这种优化不依赖于任何`-O`级别。)例如:

```
struct T {
  // ...
};

T fn()
{
  T t;
  return t;
}

T t = fn ();
```

函数返回的对象不需要名字。例如，上面函数`fn`中的 return 语句可能是`return T();`，复制省略仍然适用。在这种情况下，这个优化就是简单的*返回值优化* (RVO)。

一些程序员可能会试图通过将`std::move`放入 return 语句来“优化”代码，如下所示:

```
T fn()
{
  T t;
  return std::move (t);
}
```

但是，这里对`std::move`的调用排除了 NRVO，因为它打破了 C++标准中规定的条件，即[*【class . copy . elision】*](http://eel.is/c++draft/class.copy.elision):返回的表达式必须是名称。这样做的原因是`std::move`返回一个引用，一般来说，编译器无法知道函数返回引用的对象是什么。所以 GCC 9 会发出警告(当`-Wall`生效时):

```
t.C:8:20: warning: moving a local object in a return statement prevents copy elision [-Wpessimizing-move]
8 | return std::move (t);
  |        ~~~~~~~~~~^~~
t.C:8:20: note: remove ‘std::move’ call
```

### -可怜的家伙-走开

当函数返回的类对象是函数参数时，复制省略是不可能的。然而，当 RVO 的所有其他条件都满足时，C++(根据[核心问题 1148](https://wg21.link/cwg1148) 的解析)说应该使用 move 操作:重载解析就像对象是一个右值一样执行(这就是所谓的*两阶段重载解析*)。该参数是一个左值(因为它有一个名称)，但它即将被销毁。因此，编译器应该将 is 视为一个右值。

例如:

```
struct T {
  T(const T&) = delete;
  T(T&&);
};

T fn(T t)
{
  return t; // move used implicitly
}
```

在这里显式地使用`return std::move (t);`不会让人讨厌——在任何情况下都会使用 move 它只是多余的。编译器现在可以使用由`-Wextra`启用的新警告`-Wredundant-move`指出这一点:

```
r.C:8:21: warning: redundant move in return statement [-Wredundant-move]
8 | return std::move(t); // move used implicitly
  |        ~~~~~~~~~^~~
r.C:8:21: note: remove ‘std::move’ call
```

因为 GNU C++编译器实现了[核心问题 1579](http://wg21.link/cwg1579) ，所以下面对`std::move`的调用也是多余的:

```
struct U { };
struct T { operator U(); };

U f()
{
  T t;
  return std::move (t);
}

```

这里不可能复制省略，因为类型`T`和`U`不匹配。但是，隐式右值处理的规则没有 RVO 的规则严格，并且对`std::move`的调用是不必要的。

然而，有些情况下返回`std::move (expr)`是有意义的。隐式移动的规则要求选定的构造函数接受对返回对象类型的右值引用。有时情况并非如此。例如，当一个函数返回一个对象，该对象的类型是从该函数返回的类类型派生的类。在这种情况下，第二次执行重载决策，这次将对象视为左值:

```
struct U { };
struct T : U { };

U f()
{
  T t;
  return std::move (t);
}

```

虽然总的来说`std::move`是这门语言的一个很好的补充，但使用它并不总是合适的，而且，可悲的是，规则相当复杂。幸运的是，编译器能够识别对`std::move`的调用会阻止移动或复制的省略(或者实际上不会产生影响)的上下文，并适当地发出警告。因此，我们建议启用这些警告，并调整基本代码。回报可能是较小的性能提升和更干净的代码。GCC 9 将是 [Fedora 30](https://fedoraproject.org/wiki/Releases/30/Schedule) 的一部分，但你现在可以在 [Godbolt](https://gcc.godbolt.org/) 上试试。

*Last updated: April 23, 2019*