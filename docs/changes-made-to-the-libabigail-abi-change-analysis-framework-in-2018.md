# 2018 年对利比盖尔 made 变更分析框架所做的变更

> 原文：<https://developers.redhat.com/blog/2019/03/06/changes-made-to-the-libabigail-abi-change-analysis-framework-in-2018>

本文面向那些对软件系统的长期维护感兴趣的人，这些软件系统向其他系统公开应用程序二进制接口(也称为 ABIs)。长期维护包括检测和分析 ABIs 中不可避免的变化，并评估这些变化是否允许被维护的系统与它们交互的组件保持兼容。

在这篇文章中，我描述了我在 2018 年期间工作的 ABI 变化分析框架发生了什么:Abigail 库(Libabigail)及其相关的工具集。我们的目标不是列出在这一年中发生在整个版本 [1.2](https://sourceware.org/ml/libabigail/2018-q1/msg00015.html) 、 [1.3](https://sourceware.org/ml/libabigail/2018-q2/msg00010.html) 、 [1.4](https://sourceware.org/ml/libabigail/2018-q3/msg00000.html) 和 [1.5](https://sourceware.org/ml/libabigail/2018-q4/msg00019.html) 中的无数变化，但是我将带您浏览发生的主要变化，并正确看待它们。

## 核心功能改进

这些是对核心库的改进。因此，它们被传播到使用该库的所有工具。

### 对树叶变化报告的一般改进

几个 Libabigail 工具可以使用默认报告模式或叶更改报告模式发出更改报告。在后一种模式中，只报告类型、变量和函数的变化。与默认报告模式不同，不会报告这些更改的影响(例如，给定类型的更改影响了哪个功能以及如何影响)。换句话说，如果变更以树状方式相互链接，那么在这种模式下只报告叶子。例如， [kmidiff](https://sourceware.org/libabigail/manual/kmidiff.html) 工具默认使用这种模式。并且 [abidiff](https://sourceware.org/libabigail/manual/abidiff.html) 可以使用这种模式，使用`--leaf-changes-only`选项发出报告。

类型更改的源位置现在以此模式报告。

在此模式下，在变更报告开始时发出的变更的介绍性摘要已得到改进。

在这种模式下，变更报告的意义已经通过许多小的改变得到了提高，使得 kmidiff 工具的输出(特别是)更加有用。

### 改进的冗余检测

每当 Libabigail 发出变更报告时，它都会避免两次报告“相同的”变更。例如，假设我们有这样一个类型`struct Foo`:

> ```
>  struct Foo
>  {
>    int m0;
>  };
> ```

假设该类型由两个名为`function1`和`function2`的函数使用，如下所示:

> ```
> void
> function1(struct Foo *a)
> {
> }
> 
> void
> function2(struct Foo *b)
> {
> }
> ```

现在让我们看看 [abidiff](https://sourceware.org/libabigail/manual/abidiff.html) 如何比较包含`struct`、`function1`和`function2`定义的两个二进制版本的 ABI，其中唯一发生的变化是向`struct Foo`添加了一个数据成员，如下所示:

> ```
>  struct Foo
>  {
>    int m0;
> +  char m0;
>  };
> ```

对二进制文件的两个版本调用 [abidiff](https://sourceware.org/libabigail/manual/abidiff.html) 的结果如下:

> ```
> $ abidiff test-v0.o test-v1.o
> Functions changes summary: 0 Removed, 1 Changed (1 filtered out), 0 Added functions
> Variables changes summary: 0 Removed, 0 Changed, 0 Added variable
> 
> 1 function with some indirect sub-type change:
> 
> [C]'function void function1(Foo*)' at test-v1.cc:8:1 has some indirect sub-type changes:
>   parameter 1 of type 'Foo*' has sub-type changes:
>     in pointed to type 'struct Foo' at test-v1.cc:1:1:
>       type size changed from 32 to 64 (in bits)
>       1 data member insertion:
>         'char Foo::m1', at offset 32 (in bits) at test-v1.cc:4:1
> 
> $
> ```

请注意`struct Foo`的变化是如何影响`function1` *的。*但是`function2`也使用`struct Foo`，abidiff 主动避免在`function2`的上下文中报告`struct Foo`的变化，因为这样的报告是多余的。

但是，在某些情况下，我们希望报告多余的变更。例如，我们想要报告函数参数变化的所有实例，其中类型`const char*`被修改为`char*` *。*这些变化不应被视为多余。这是利阿比盖尔所缺乏的一个领域。这是过度过滤多余的变化。这一点现在有所改善。

冗余检测改进的另一个例子是在`enum`类型上。Libabigail 的报告传递未能检测到已经报告了对`enum`类型的给定更改。因此，在某些情况下，工具会报告给定的`enum`类型在不同的上下文中发生了多次变化。这是固定的。

### 改进的变更分类

每当 Libabigail 的比较引擎检测到一个 ABI 工件(例如，一个符号、类型或声明)发生了变化，这个变化就会在一个内部(内存中)表示中建模，也称为 *diff IR* 。diff IR 是一个图，其中的节点是工件更改(也称为 diff 节点)。diff IR 随后出于各种目的通过各种途径进行处理。其中一次传递的目的是*对每个 diff 节点所携带的变更进行分类*。每个 diff 节点所携带的每个变更最终都属于三大类别之一:有害的变更、无害的变更和未分类的变更。

稍后，当为了发出变更报告而遍历 diff IR 时，无害的变更可以例如缺省地被忽略。这有助于通过能够避免报告被认为对用户不重要的 ABI 变更来增加变更报告的信噪比。但是这种分类业务(尤其是我们试图改进的部分)似乎是永无止境的。

最近改进的一个例子是“无害的名称更改”类别。每当一个 diff IR 节点带有一个`typedef`或一个`enum`类型名称变化时，Libabigail 会认为这种变化是无害的。这反过来允许变更报告过程避免显示默认的`typedef`和`enum`名称变更，因为这些对我们正在查看的库的 ABI 没有影响。这一切都很好，除非 diff IR 节点还带有其他可能被认为无害的更改。因此，Libabigail 的变更分类引擎现在已经“收紧”了将`typedef`或`enum`类型的 diff IR 节点分类为无害名称变更的条件。对于`typedef`类型，现在只有在底层类型的文本表示没有变化的情况下，名称变化才被认为是无害的。对于`enum`类型，现在只有在`enum`中没有其他变化时，名称变化才被认为是无害的，例如，在枚举器或基础类型上。

默认情况下，函数返回类型的变化现在也被归类为无害的。注意，这些已经被归类为对函数参数类型无害。

每当一个`void*`指针被改变成一个更“类型化”的指针时，这种改变现在被默认归类为无害的。

### 支持匿名数据成员

匿名数据成员是没有名称的`struct`或联合的数据成员。这种数据成员的类型可以是`struct`或联合，例如:

> ```
> struct Foo
> {
>   int a;
>   struct /* <-- This is an anonymous data member. */
>    {
>      char b;
>     char c;
>    };
>    int d;
> };
> ```

由 [GCC](https://gcc.gnu.org) 以 [DWARF](http://dwarfstd.org/Dwarf5Std.php) 格式发出的调试信息描述了这样的匿名数据成员构造，但是 DWARF 阅读器和 Libabigail 的各种内部表示必须进行调整以支持它们。现在支持这一功能。

### 更好地支持 ELF 符号版本

一个 ELF 符号可以有多个版本，Libabigail 长期以来一直支持这个特性。但是当一个函数符号 *S* 有几个版本，并且几个不同名字的不同函数有这些不同版本的 *S* 作为底层符号时，Libabigail 可能会错误地把一个函数当成另一个函数。这是因为在某些情况下，Libabigail 使用符号名来标识函数，而不考虑版本名。这个问题现在已经得到了解决，Libabigail 在识别函数时总是会考虑符号版本。

### 抑制规范中对联合类型的支持

Libabigail(及其工具)允许用户根据自己的需要取消变更报告。用户可以提供一个文件，在这个文件中，他们描述了应该禁止变更的工件的种类。例如，用户可以说工具不应该报告对名为`FooPrivateType`的类型的更改。为此，用户将编写如下所示的抑制规范文件:

> ```
> [suppress_type]
>   name = FooPrivateType
> ```

然后使用适当的选项将抑制规范文件传递给 Libabigail 工具。

那种抑制规范现在也作用于*联合*类型。

## 新项目的默认抑制规范

Libabigail 安装默认的抑制规范，这些规范被像 [abidiff](https://sourceware.org/libabigail/manual/abidiff.html) 和 [abipkgdiff](https://sourceware.org/libabigail/manual/abipkgdiff.html) 这样的工具自动和隐式地使用，只要这些工具比较一些由它们的文件名或它们的 [soname](https://en.wikipedia.org/wiki/Soname) 标识的共享库。

每当一个特定的项目团队觉得需要定义一组特定的抑制规则时(例如，根据特定的命名方案，抑制在被认为是私有的类型或符号上检测到的更改)，项目成员可以联系 Libabigail 开发人员，以便我们一起为项目提出一个默认的抑制规范。例如，Libabigail 已经为几个系统库安装了默认的抑制规范。

本着这种精神，现在为 [krb5](https://web.mit.edu/kerberos/) 和 [libvirt](https://libvirt.org/) 项目及其库安装了一个新的默认抑制规范。

## 特定于工具的改进

### fedabipkgdiff

Fedabipkgdiff 是一个命令行工具，用来比较包含在 [Fedora](https://getfedora.org/) 包中的 ELF 二进制文件的 ABI。它与 [Fedora 构建系统](https://koji.fedoraproject.org/koji/)进行交互，以获得要操作的包。

这个工具被移植到 Python 3 上，作为在 Fedora 中移植到 Python 3 的总体努力的一部分。不过，它仍然可以用于 Python 2。

### abipkgdiff

abipkgdiff 是一个命令行工具，用于比较本地可用软件包中包含的 ELF 二进制文件的 ABI。

当一个 RPM 包含一个共享库，它的 [soname](https://en.wikipedia.org/wiki/Soname) 没有被 RPM 广告为“已提供”时，abipkgdiff 现在认为这个共享库是 RPM 私有的。因此，它会将该共享库从要比较的库中删除。这可以防止 abipkgdiff 发出关于被认为是包私有的库的 ABI 更改报告。

## 结论

2018 年期间，Libabigail 及其相关工具的代码库进行了其他几项修复和改进。这些之所以成为可能，是因为用户花时间[报告他们在使用框架时遇到的问题](http://sourceware.org/bugzilla/enter_bug.cgi?product=libabigail),或者请求他们在尝试使框架适应他们的环境时想到的增强。我要热情真诚地感谢他们。

Libabigail 开发人员一直致力于改进 Libabigail 静态分析框架的特性，我们希望每次您想联系我们时都能得到您的回复。

*Last updated: March 8, 2019*