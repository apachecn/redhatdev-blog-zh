# 用 Clang 定制编译过程:做出妥协

> 原文：<https://developers.redhat.com/blog/2019/08/06/customize-the-compilation-process-with-clang-making-compromises>

在这个由两部分组成的系列中，我们将讨论 Clang 编译器和定制编译过程的各种方法。这些文章是此次演讲的扩展版本，名为 [Merci le Compilo](http://serge-sans-paille.github.io/talks/cppp2019/output/index.html) ，于 6 月在 [CPPP](https://cppp.fr/) 举行。

在[第一部分](https://developers.redhat.com/blog/?p=614867)中，我们查看了定制的具体选项。在本文中，我们将看看不同方法中涉及的一些折衷和权衡的例子。

## 做出妥协

### 调试精度与尺寸

提高调试信息的准确性会产生更大的二进制文件。相反，降低调试信息的准确性会减小其大小。您可以通过以下方式控制这种行为:

*   `-g1`:精度较低。
*   `-g2`
*   `-g3`:精度更高。
*   `-fdebug-macro`:包含宏的调试信息！

回想一下，可以将调试信息提取到一个单独的文件中。像 Fedora、Red Hat Enterprise Linux 或 Debian 这样的发行版提供了独立的调试包:

*   `objcopy --only-keep-debug`提取调试信息。
*   `objcopy --compress-debug-sections`压缩它们。

与`gcc`不同，`clang`在我们的测试用例中对`-g2`和`-g3`没有任何区别:

```
$ for g in 1 2 3 ""
  do
    printf "-g$g: \t" && curl $sq | clang -c -O2 -g$g -xc - -o- | wc -c
  done
-g1   : 3168632
-g2   : 7025488
-g3   : 7025488
-g    : 7025488

```

**奖金** : `-fdebug-macro -g : 7167752`

### 优化级别对编译时间的影响

人们可以预期更多的优化需要更多的时间——编译器会更加努力地尝试。然而，下面的实验使这种直觉无效:

```
$ for O in 0 1 2 3
  do
  /usr/bin/time -f "-O$O: %e s" clang sqlite3.c -c -O$O
  done
-O0: 22.15 s
-O1: 24.02 s
-O2: 22.68 s
-O3: 22.36 s

```

这还是可以理解的；许多优化从代码中删除指令，这导致更小的输入，从而更快地被后面的优化步骤处理。

### 精度与性能

在某些情况下，这可能与牺牲(计算的)准确性来换取性能有关。对于浮点运算来说尤其如此:

*   `-ffp-contract=fast|on|off`:浮点表达式收缩。
*   `-ffast-math`:假设浮点运算是关联的，并且没有`NaN`、`inf`或反规范数。
*   `-freciprocal-math`:通过文字优化除法。
*   `-Ofast`:-O3+-ffast-math=-of ast。

以下示例说明了编译器如何将(慢)除法转换为(快)乘法:

```
$ clang -xc - -o- -S -emit-llvm -O2 -freciprocal-math << EOF
double rm(double x) {
  return x / 10.;
}
EOF
define double @rm(double) {
  %2 = fmul arcp double %0, 1.000000e-01
  ret double %2
}

```

这个例子表明，`clang`成功地对 double 的向量和进行了矢量化，利用`-Ofast`来改变指令顺序并对它们进行矢量化，正如`<2 x double>` LLVM 向量类型所指出的。

```
$ clang -xc++ - -o- -S -emit-llvm -Ofast << EOF
#include <numeric>
#include <vector>
using namespace std;
double acc(vector<double> const& some)
{
  return accumulate(
           some.begin(),
           some.end(),
           0.);
}
EOF
...
%95 = fadd fast <2 x double> %94, %93
...

```

### 便携性与性能

二进制文件可以是某个体系结构的通用文件，比如 x86_64，也可以利用某个指令集(例如 AVX)。以将二进制代码局限于特定的处理器家族为代价，用一种代码替换另一种代码可以极大地提高性能。

*   `-march=native`:使用主机架构上所有可用的指令。
*   `-mavx`:生成可以使用 AVX 指令集的代码(即使主机上没有)。

以下代码结合了特定于架构的功能，此处是融合乘加的可用性，以及浮点精度的放宽:

```
$ clang++ -O2 -S -o- -march=native -ffp-contract=fast << EOF
double fma(double x, double y, double z) {
  return x + y * z;
}
EOF
...
vfmadd213sd %xmm0, %xmm2, %xmm1

```

### 性能与安全性

`clang`编译器提供了几个清理程序，对程序的各个方面进行运行时检查。结合适当的测试套件，这是检测程序中问题的好方法。通常认为发布带有杀毒标志的软件是一个坏主意，因为它们会显著影响性能——尽管这种影响小于在未经测试的可执行文件上运行 Valgrind。

*   `-fsanitize=address`:仪器内存访问，添加越界检查。
*   `-fsanitize=memory`:跟踪对未初始化值的访问。
*   `-fsanitize=undefined`:跟踪未定义的行为。
*   `-fsanitize=thread`:检测多线程程序中的*数据竞争*。

为了说明插装的影响，让我们研究一下编译以下代码片段所生成的 LLVM 位代码:

```
// mem.cpp #include <memory>
double x(std::unique_ptr<double> y) {
  return *y;
}

```

```
$ clang++ -fsanitize=address mem.cpp -S -emit-llvm -o- -O2

```

就在通过`getelementptr`进行存储器访问之前，计算并查找一个关键字，以确定指针所引用的存储器位置的状态。然后代码分支进行检查，要么报告错误，要么继续。

```
...
%h = getelementptr inbounds %"class.std::unique_ptr", %"class.std::unique_ptr"* %y, i64 0, i32 0, i32 0, i32 0, i32 0, i32 0
%1 = ptrtoint double** %h to i64
%2 = lshr i64 %1, 3
%3 = add i64 %2, 2147450880
%4 = inttoptr i64 %3 to i8*
%5 = load i8, i8* %4
%6 = icmp ne i8 %5, 0
br i1 %6, label %7, label %8

; <label>:7: call void @__asan_report_load8(i64 %1)
call void asm sideeffect "", ""()
unreachable

; <label>:8: %9 = load double*, double** %h, align 8

```

###### `from __future__ import`

Clang 编译器支持不同版本的 C++标准，所以如果你在一个给定的代码库上工作，你可以控制你允许使用的语言特性。如果您计划拥有一个可由几个工具链编译的代码库，那么这个功能就特别重要:语言版本是一个坚实的共同点。

*   选择你的标准版本。
*   选择你的毒药，允许使用方言。
*   `-fcoroutines-ts`:启用实验技术规范。

使用 clang 本身捆绑的 clang CLI 自动完成特性，可以列出所有支持的标准:

```
$ clang --autocomplete=-std=,
...
c++2a
...
cuda
...
gnu1x
...
iso9899:2011

```

### 控制安全功能

还可以在代码中插入各种对策来防止利用缓冲区溢出或 ROP 的基本攻击。

*   添加一个堆栈金丝雀，以检测(一些)堆栈崩溃。
*   `-fstack-protector-strong`:同上，但适用于更多功能。
*   `-fstack-protector-all`:同上，但适用于所有功能。堆栈探测的开销不是特别大，但是这确实会使你的代码变得更慢更大。
*   `-fsanitize=safe-stack`:将堆叠分为 RO 堆叠和 RW 堆叠，以使堆叠更难粉碎
*   `-fsanitize=cfi`:仪表控制流，检测对手可能控制控制流的各种情况。存在各种保护方案，参见[控制流完整性文件](https://clang.llvm.org/docs/ControlFlowIntegrity.html)。

让我们来看看金丝雀的飞行:

```
$ clang -O2 -fstack-protector-all -S \ -o- -xc++ - << EOF
#include <array>
using namespace std;
auto access(array<__int128_t, 10> a,
            unsigned i)
{
  return a[i];
}
EOF
...
cmpq    (%rsp), %rcx
jne .LBB0_2
popq    %rcx
retq
.LBB0_2:
callq   __stack_chk_fail

```

在函数的末尾，`-fstack-protector-all`在一个值和堆栈金丝雀之间插入了一个检查，如果比较失败，就会调用`__stack_chk_fail`。

### 为编译器提供信息

直觉上，编译器拥有的信息越多，它就能更好地应用它的优化。您可以收集更多信息或提供编译器提示。

### 性能指标优化(PGO)

如果您的应用程序有一个相关的示例用例，并且您愿意基于该示例优化您的应用程序，那么您可以使用 Profile Guided Optimization (PGO)。

1.  用`-fprofile-generate`编译整个代码库。这会生成额外的代码来记录最常访问的函数和分支。
2.  在用例上运行生成的二进制文件。
3.  用`-fprofile-use`重新编译你的代码。

由于收集了这些信息，编译器可以更好地对函数进行分组和放置，对基本块进行分组和放置，并且对优化(如循环展开或内联)有更好的提示。

### 链接时间优化(LTO)

过去，由于内存限制，需要单独编译。现在，在编译期间将整个程序加载到内存中可能是一个有效的选择:这就是链接时间优化。

*   `-flto=full`:链接时，整个程序再次优化。内存需求更加重要，但这也带来了更多的优化机会。
*   对于每个函数，计算额外的摘要，编译器可以根据这些摘要做出决定，降低内存需求，代价是可能错过一些优化。

有趣的是，`-flto`标志实际上产生 LLVM 位代码，而不是 ELF 文件:

```
$ echo 'foo() { return 0;}' | clang -flto -O2 -xc - -c -ofoo.o
$ file foo.o
foo.o: LLVM bitcode

```

### 调谐优化

一些单独的过程接受额外的参数来控制阈值效果。最值得注意的是:

*   `-mllvm -inline-threshold=n`:控件内联。
*   `-mllvm -unroll-threshold=n`:控制展开。

阈值越大，内联的函数越多，展开的循环越多。

不幸的是，这适用于整个编译单元。对于更细粒度的控制，可以依赖编译器指令，*又名*杂注。

以下 pragmas 对循环有效，并控制循环优化的各个方面。它们的作用相对简单:控制编译器是否展开给定的循环，选择展开因子

```
#pragma clang loop unroll(enable|full)
#pragma clang loop unroll_count(8)

```

使用下面的 pragma 也有可能得到目标版本的`-ffp-contract`。在这种情况下，指定的契约策略仅对修饰指令有效，否则将应用默认契约(或通过命令行指定的契约)。

```
#pragma clang fp contract(fast)

```

更多的编译指令在[语言扩展文档](https://clang.llvm.org/docs/LanguageExtensions.html)中有详细说明

### 获得编译器反馈

众所周知，编译 C++会花费，比如说，**一些**时间。Clang 可以提供关于每个编译步骤花费了多少时间的详细反馈。相关标志是`-ftime-report`。

要详细跟踪优化过程，也可以使用*备注*机制，在每次优化的基础上，要求优化过程的详细输出:

*   `-Rpass=inline`
*   `-Rpass=unroll`
*   `-Rpass=loop-vectorize`

不过，这些标志往往会产生很多噪音:

```
$ { clang -xc++ - -c \
  -O2 -Rpass=inline << EOF
#include <numeric>
#include <vector>
using namespace std;
double acc(vector<double> const& some)
{
  return accumulate(
           some.begin(),
           some.end(),
           0.);
}
EOF
} 2>&1 | c++filt
...
... remark: __gnu_cxx::__normal_iterator<double const*, std::vector<double, std::allocator<double> > >::__normal_iterator(double const* const&) \ ... inlined into std::vector<double, std::allocator<double> >::begin() const with cost=-40 (threshold=337) [-Rpass=inline]

```

## 总结词

这两篇文章旨在展示编译器是一个复杂的软件，它有更多的功能，而不仅仅是让代码更快。编译速度、可执行文件大小、安全性、使开发过程更快，这些只是编译器努力覆盖的多个目标中的一部分。

探索各种旗帜是一项永无止境的任务，但这里有一个很有价值的任务:

```
$ clang --autocomplete=- | wc -l  # Count the number of compiler options that Clang accepts 3197

```

而且这还没有考虑到所有可以在 LLVM 级别上完成的低级调优！

*Last updated: July 29, 2019*