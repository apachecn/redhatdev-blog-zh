# 用 Clang:优化选项定制编译过程

> 原文：<https://developers.redhat.com/blog/2019/08/05/customize-the-compilation-process-with-clang-optimization-options>

当使用 C++时，开发人员通常希望在不牺牲性能的情况下保持高水平的抽象。这是著名的格言“*无成本抽象*”然而，C++语言实际上在性能方面并没有给开发者很多保证。您可以保证复制省略或编译时评估，但是关键的优化，如*内联*、*展开*、*常数传播*或者，我敢说，*尾部调用消除*都受到标准最好的朋友:编译器的善意。

本文主要关注 [Clang](https://clang.llvm.org) 编译器和它提供的定制编译过程的各种标志。我已经尽力不让这份清单变得无聊，它当然也不是一份详尽的清单。

这篇文章是 2019 年 6 月 15 日在 [CPPP](https://cppp.fr/) 发表的演讲" [Merci le Compilo"](http://serge-sans-paille.github.io/talks/cppp2019/output/index.html) 的扩展版。

使用的`clang`版本基于 trunk，运行在 RHEL 7 上。

时不时地，我会使用 SQLite merging C 源代码作为大型第三方代码。让我们假设下面一行是从:

```
sq=https://raw.githubusercontent.com/azadkuh/sqlite-amalgamation/master/sqlite3.c

```

## 简介:陈述目标

下面的源代码是一个相对愚蠢的程序版本，它将从标准输入中读取的数字相加。很可能是内存受限，但仍有一些处理在进行:

```
#include <iostream>
int main(int argc, char** argv) {
  long s = 0;
  while (std::cin) {
    long tmp = 0;
    std::cin >> tmp;
    s += tmp;
  }
  std::cout << s << std::endl;
  return 0;
}

```

这是一个用 Python 编写的相对类似但不等同的程序。Python 默认使用大整数，所以它在溢出方面的表现不同，但是对于我们的目的来说已经足够了。

```
import sys
print(sum(int(x) for x in sys.stdin.readlines()))

```

让我们采用一种简单的方法，在一个相对较大的输入集上测量这两个程序的执行时间:

```
$ seq 1000000 > numbers
$ clang++ sum.cpp -o sum
$ time ./sum < numbers
0.61s user 0.01s system 94% cpu 0.659 total

$ time python sum.py < numbers
0.77s user 0.04s system 99% cpu 0.818 total

```

本机代码当然更快，但也快不了多少。我们不能从一次运行中得出太多的结论，但至少有一件事是肯定的:`clang`用户没有指定他们的意图，所以编译器只是生成了一个有效的二进制文件——谢天谢地这是一个硬约束——并没有试图为用户感兴趣的任何指标优化它。

如果用户想要优化执行速度，他们应该指定这个意图，比如说，通过`-O2`标志:

```
$ clang++ -O2 sum.cpp -o sum
$ time ./count < numbers
0.34s user 0.00s system 99% cpu 0.348 total

```

## 多准则优化

对于各种各样的代码库，不仅仅是为了速度而优化。有时候，你想限制二进制的大小；有时候，你可以用速度换取额外的安全。这也取决于您处于开发生命周期的哪个阶段。例如，在代码编辑期间，您希望快速分析代码，在错误跟踪期间，您希望获得尽可能多的调试信息，等等。

```
 #
 ##                           #
 ##                           ##
 ##            ##             ##
 ##            ##             ##
 ##            ##             ##
 ##    ##      ##             ##
 ##    ##      ##      #      ##
 ##    ##      ##      ##     ##
PERF  DEBUG   EDIT    SECU   SIZE

```

### 表演

*我希望生成的二进制文件运行得更快*对于编译器来说是一个非常常见的查询，因此以下是最常用的标志:

*   `-O0`:完全没有优化。
*   `-O1`:O1=(O0+O2)/(2)。我几乎不用这面旗。
*   `-O2`:尽可能地优化，不要冒显著增加二进制文件大小或降低性能的风险。
*   进一步优化，用二进制大小来换取速度，有时做出的决定可能会对性能产生负面影响。
*   `-O4`:O3=O4。这是一个神话。

**奖励:** `-O3 -mllvm -polly`激活多面体优化，如果 Clang 是用 Polly 支持编译的。

### 调试

*我想调试我的代码，我不在乎性能*很遗憾，这也是一个常见的请求:-/

*   `-g`:包含调试信息。
*   `-Og` : `== -O1 -g`。这已经是性能和可调试性之间的权衡了。

出于好奇，下面的代码片段验证了在传递`-g`标志时确实生成了调试信息部分:

```
$ curl $sq | clang -xc -c -g - -o sq.o
$ objdump -h sq.o | grep debug
  #  name            size      ...
   9 .debug_str      00012b2d  ...
  10 .debug_abbrev   0000038d  ...
  11 .debug_info     0005056c  ...
  12 .debug_ranges   00000240  ...
  13 .debug_macinfo  00000001  ...
  14 .debug_pubnames 0000c73a  ...
  15 .debug_pubtypes 00001068  ...
  19 .debug_line     00073402  ...

```

### 安全性

*我想保护我的代码不被他人窃取，而我自己*的重要性在这些日子里越来越大。在不影响性能的情况下影响安全性的标志并不多，但是值得一提的是`-D_FORTIFY_SOURCE=2`。这为一些函数选择了不同的声明，例如:

```
$ clang -xc -c -O2 - -S -emit-llvm -o - -D_FORTIFY_SOURCE=2 << EOF
#include <stdio.h>
void foo(char *s) {
  printf(s, s);
}
EOF
define void @foo(i8*) {
  %2 = tail call i32 (i32, i8*, ...) @__printf_chk(i32 1, i8* %0, i8* %0)
  ret void
}

```

宏定义启用了`printf`的强化版本，即`__printf_chk`，它也检查可变参数的数量。

### 大小

我想对我的二进制文件进行某种权重控制可能是某个嵌入式系统的有效需求。在这种情况下，您可以使用:

*   `-Os`:与`-O2`相同，具有额外的代码大小优化，包括不同的转换参数，如*内联*。
*   `-Oz`:与`-Os`相同，但有更多的尺寸优化，代价是性能下降。

让我们展示这些标志对合并二进制文件的影响:

```
$ curl $sq|clang -xc - -O2 -c -o-|wc -c
1488400
$ curl $sq|clang -xc - -Os -c -o-|wc -c
850696
$ curl $sq|clang -xc - -Oz -c -o-|wc -c
796976

```

### 编辑

编译器还通过一系列警告和代码编辑功能帮助生成更好的代码:

*   `-Wall`:(几乎)所有警告。
*   如果你认为一个警告应该是一个错误，你可以根据警告有选择地启用这个特性。
*   `-w`:如果你不知道它是做什么的，你可能也不想知道:-)
*   `-Xclang -code-completion-at`:一个内部标志，IDE 可以使用它来提供*智能*代码完成。

```
$ cat hello.cpp
#include <iostream> int main(int argc, char**argv) {
  std::co
$ clang++ -Xclang -code-completion-at=hello.cpp:3:10 -fsyntax-only hello.cpp
COMPLETION: codecvt : codecvt<<#typename _InternT#>, <#typename _ExternT#>, <#typename _StateT#>> COMPLETION: codecvt_base : codecvt_base
...
COMPLETION: cout : [#ostream#]cout

```

在这种情况下，`clang`输出在名称空间`std`中可用的以`co`开始的所有标识符。

在下一篇文章中，我们将研究优化中涉及的各种妥协和权衡，例如调试精度与二进制大小、优化级别对编译时间的影响，以及性能与安全性。敬请关注。

*Last updated: July 29, 2019*