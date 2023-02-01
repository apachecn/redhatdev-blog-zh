# 看看 LLVM 高级数据类型和普通的可复制类型

> 原文：<https://developers.redhat.com/blog/2019/04/01/a-look-at-llvm-advanced-data-types-and-trivially-copyable-types>

有几个 bug 长期潜伏在 [LLVM Bugzilla](https://bugs.llvm.org/) 中，即 [#39427](https://bugs.llvm.org/show_bug.cgi?id=39427) 和 [#35978、](https://bugs.llvm.org/show_bug.cgi?id=35978)，它们与 [`is_trivially_copyable`](https://en.cppreference.com/w/cpp/types/is_trivially_copyable) 数据类型的一个自定义实现有关，对 [LLVM](http://llvm.org/) 库的[应用二进制接口(ABI)](https://en.wikipedia.org/wiki/Application_binary_interface) 造成了不好的影响。在本文中，我将仔细研究这些问题，并描述潜在的解决方法。

LLVM 编译器基础设施依靠几种高级数据类型(ADT)来提供与标准模板库(STL)中的容器不同的速度/大小权衡。此外，这个 ADT 库提供了未来标准版本的特性，但是是在 LLVM 支持的 C++版本(目前是 C++11)中实现的。最后，这些 ADT 必须与 LLVM 代码库的编译器要求兼容；基本上 GCC 版本> = 4.8，Clang 版本> = 3.1。(如果你对 LLVM ADTs 感兴趣，Chandler Carruth 在 [CppCon 2016](https://www.youtube.com/watch?v=vElZc6zSIXM) 上就这个话题做了一次很好的演讲。)

在这些数据类型中，`llvm::SmallVector`类型是`std::vector`的替代类型，如果数组包含的元素少于`N + 1`元素，则使用就地存储，否则使用堆存储。有趣的是，`llvm::SmallVector`有一个专门化，当`T`被认为是平凡的可复制的，当修剪、复制数据或将数据从一个容器移动到另一个容器时，允许更少更快的数据移动。专门化基本上是这样的:

```
template<class T>
class SmallVectorBase {
   ... ;
};
template<class T>
class SmallVector : public SmallVectorBase<T> {
   ... ;
};

```

不幸的是，`std::is_trivially_copyable`不被 GCC 的旧版本所支持，所以 LLVM 代码库过去以这种(简化的)形式提供它自己的版本:

```
template<class T>
struct is_trivially_copyable {
    static constexpr bool value =
    #if defined(__GNUC__) && __GNUC__ >= 5
        std::is_trivially_copyable<T>::value
    #else
        !std::is_class<T>::value
    #endif
    ;
}

```

在实现中有一个固有的问题，这不是一个有效性问题。考虑以下编译单元:

```
// lib.cpp
#include 
struct DataType {
    struct SomeRandomType { int Value;};
    llvm::SmallVector<SomeRandomType> Data;
};

DataType Global;

```

```
// user.cpp
#include 
struct DataType {
    struct SomeRandomType { int Value;};
    llvm::SmallVector<SomeRandomType> Data;
};

extern DataType Global;
DataType Local;

```

如果`lib.cpp`用 GCC 4.9 编译而`user.cpp`用 GCC 5.1 编译会怎么样？快速查看符号表(例如，通过`nm -C`)显示`lib.o`定义了符号`llvm::SmallVectorTemplateBase::SmallVectorTemplateBase(unsigned long)`，而`user.o`定义了符号`llvm::SmallVectorTemplateBase::SmallVectorTemplateBase(unsigned long)`。这种方法会导致各种错误，从名称相同但布局不同的类型到链接错误。

这种情况可能发生在二进制发行版上，其中用于编译系统库的编译器和用于编译用户代码的编译器可能不同，这是 ABI 错误的可能实例之一。避免这种错误是软件打包员的任务之一。

## 变通办法

作为一种变通方法，`llvm::is_trivially_copyable`专门用于各种类型，以加强预期的属性，即使 trait 实现说的是相反的。这很难维护，而且容易出错(例如，从 C++11 开始，一对 int 就不容易复制；参见[https://godbolt.org/z/184QEc](https://godbolt.org/z/184QEc)。

那么解决办法是什么呢？避免`llvm::is_trivially_copyable`的每个编译器版本的实现，提供一个通用的实现。这不是一项容易的任务，因为它通常是作为编译器内置的来实现的(即，`__is_trivially_copyable`表示 clang)。幸运的是，有一条出路，但要理解它，我们需要理解什么是平凡的可复制性。一个普通的可复制类型验证以下属性:

*   每个复制构造函数都是琐碎的或被删除的
*   每个 move 构造函数都是琐碎的或者被删除了
*   每个复制赋值操作符都是琐碎的或被删除的
*   每个移动赋值操作符都是琐碎的或被删除的
*   平凡的非删除析构函数

从 C++17 开始，还要求至少必须存在一个复制/移动构造函数或赋值操作符，但是 LLVM 代码库不受此影响(还没有)。

检查构造函数或赋值是否已被删除通常可以通过“替换失败不是错误”(SFINAE)来实现，如:

```
template <class T>
struct is_copy_assignable {
    template <class F>
    static auto get(F*) -> decltype(std::declval() = std::declval(), std::true_type{});
    static std::false_type get(...);
    static constexpr bool value = decltype(get((T*)nullptr))::value;
};

```

检查这个实现是否是默认的要稍微复杂一些。幸运的是，从 C++11 开始:

*如果一个联合包含一个非静态数据成员，该成员具有一个非平凡的特殊*
*成员函数(复制/移动构造函数、复制/移动赋值函数或*
*析构函数)，该函数在联合中默认被删除，需要由程序员明确定义*
*。*

这意味着实例化以下类型的`is_copy_assignable`:

```
template
union trivial_helper {
    T t;
};

```

告诉我们关联的类型是否是可复制可赋值的。

一旦建立了这个工具，这个平凡的可复制特性可以总结为:

```
static constexpr bool value =
  has_trivial_destructor<T> &&
  (has_deleted_move_assign<T> || has_trivial_move_assign<T>) &&
  (has_deleted_move_constructor<T> || has_trivial_move_constructor<T>) &&
  (has_deleted_copy_assign<T> || has_trivial_copy_assign<T>) &&
  (has_deleted_copy_constructor<T> || has_trivial_copy_constructor<T>);

```

为了验证实现相对于`std`的一致性，可以添加以下受保护的静态断言:

```
#ifdef HAVE_STD_IS_TRIVIALLY_COPYABLE
  static_assert(value == std::is_trivially_copyable<T>::value,
                "inconsistent behavior between llvm:: and std:: implementation of is_trivially_copyable");
#endif

```

最终，我们修复了`llvm::SmallVector`实现的 ABI 不稳定性，万岁！作为一个悲伤(或快乐，取决于你的观点)的注释，LLVM 越来越接近要求 GCC 5.1 或更高，这使得整个探索过时了。

*Last updated: March 28, 2019*