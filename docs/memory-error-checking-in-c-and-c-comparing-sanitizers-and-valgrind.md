# C 和 C++中的内存错误检查:比较杀毒软件和 Valgrind

> 原文：<https://developers.redhat.com/blog/2021/05/05/memory-error-checking-in-c-and-c-comparing-sanitizers-and-valgrind>

这篇文章比较了两个工具，杀毒软件和 T2，这两个工具可以发现用 T4 语言编写的程序中的内存错误。这两个工具以非常不同的方式工作。因此，虽然杀毒软件(由谷歌工程师开发)比 Valgrind 有几个优点，但每一个都有优点和缺点。请注意，清理程序项目有一个复数名称，因为该套件由几个工具组成，我们将在本文中探讨这些工具。

内存检查工具适用于内存不安全的语言，如 [C 和 C++](/topics/c) ，不适用于 Java、Python 和类似的内存安全语言。在内存不安全的语言中，很容易错误地写入超过内存缓冲区的末尾，或者在内存被释放后读取内存。包含这种错误的程序可能在大多数时候运行得完美无缺，很少崩溃。捕捉这些错误是困难的，这就是为什么我们需要工具来达到这个目的。

Valgrind 对程序施加的减速比杀毒软件高得多。在 Valgrind 下运行的程序可能比常规产品慢 20 到 50 倍。这可能会成为 CPU 密集型程序的绊脚石。消毒剂生产的放缓通常比常规生产严重两到四倍。除了 Valgrind，您还可以在编译期间指定使用杀毒程序。

本文分为以下几个部分:

*   [快速消毒指南](#tldr)
*   [消毒剂的性能优势](#performance)
*   [安装 debuginfo](#debuginfo)
*   [示例案例:缓冲区溢出](#bufferoverrun)
*   [ASAN:堆栈变量中的越界访问](#outofboundsstack)
*   [ASAN:全局变量中的越界访问](#outofboundsglobal)
*   [MSAN:未初始化的内存读取](#uninit)
*   [ASAN:返回后堆栈使用](#stackafterreturn)
*   [UBSAN:未定义的行为](#undefined)
*   [LSAN:内存泄漏](#leak)
*   [LSAN:特定库内存泄漏(glib2)](#leakglib)
*   TSAN:数据竞赛
*   [重新编译库](#recompilation)
*   [杀毒软件与 _FORTIFY_SOURCE 的交互](#fortifysource)
*   [结论](#conclusion)

## 快速消毒指南

下面列出的规则和建议总结了本文中的一些信息，可以帮助熟悉杀毒软件的读者:

*   Clang/GCC 选项:

    ```
    -fsanitize=address -fsanitize=undefined -fno-sanitize-recover=all -fsanitize=float-divide-by-zero -fsanitize=float-cast-overflow -fno-sanitize=null -fno-sanitize=alignment
    ```

*   对于 LLDB/GDB，为了防止非常短的堆栈跟踪和通常的错误泄漏检测:

    ```
    $ export ASAN_OPTIONS=abort_on_error=1:fast_unwind_on_malloc=0:detect_leaks=0 UBSAN_OPTIONS=print_stacktrace=1
    ```

*   修复使用[glib 2](https://en.wikipedia.org/wiki/GLib):

    ```
    $ export G_SLICE=always-malloc G_DEBUG=gc-friendly
    ```

    时的错误内存泄漏报告
*   Clang 选项捕捉未初始化的内存读取:`-fsanitize=memory`。该选项不能与`-fsanitize=address`结合使用。
*   rpmbuild `*.spec`文件应该另外使用:`-Wp,-U_FORTIFY_SOURCE`。

## 消毒剂的性能优势

Valgrind 在编译时使用动态插装而不是静态插装，这会导致高性能开销，这对 CPU 密集型应用程序来说是不切实际的。清理程序使用静态工具，并允许以较低的开销进行类似的检查。

表 1 给出了一个更完整的[比较](https://github.com/google/sanitizers/wiki/AddressSanitizerComparisonOfMemoryTools),显示了两组工具的特性和运行时变慢的情况。

**Table 1: Comparison between Sanitizers and Valgrind**

|   | 消毒剂 | Valgrind |
| [ASAN](https://clang.llvm.org/docs/AddressSanitizer.html) :一般运行时减速 | 2 次- 4 次 | 20 次- 50 次 |
| [ASAN](https://clang.llvm.org/docs/AddressSanitizer.html) : [减速测试回路](https://godbolt.org/z/vedx5q) | 1(=相同) | 3.45 倍(+23 秒=12 倍) |
| [ASAN](https://clang.llvm.org/docs/AddressSanitizer.html) : [减速测试分配](https://godbolt.org/z/zjTG8q) | 17 次 | 101 次(+23s=256 次) |
| [ASAN](https://clang.llvm.org/docs/AddressSanitizer.html) : [减速测试 AVX2](https://godbolt.org/z/Y5EPc3) | 1.05 (=慢了 5%) | 53 次(+23s=60 次) |
| [TSAN](https://clang.llvm.org/docs/ThreadSanitizer.html) : [减速测试数据赛](https://godbolt.org/z/6zMWEe) | 9 次 | 340 次(+31 次=404 次) |
| 变量溢出([栈](#outofboundsstack)，[全局](#outofboundsglobal)) | 被抓( [ASAN](https://clang.llvm.org/docs/AddressSanitizer.html) ) | 未命中([内存检查](https://www.valgrind.org/docs/manual/mc-manual.html)) |
| [未初始化的内存](#uninit) | 被抓( [MSAN](https://clang.llvm.org/docs/MemorySanitizer.html) ) | 被捕获([内存检查](https://www.valgrind.org/docs/manual/mc-manual.html)) |
| [返回后堆栈使用](#stackafterreturn) | 被抓( [ASAN](https://clang.llvm.org/docs/AddressSanitizer.html) ) | 未命中([内存检查](https://www.valgrind.org/docs/manual/mc-manual.html)) |
| [未定义的行为](#undefined) | 被捕([瑞银](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)) | 未命中([内存检查](https://www.valgrind.org/docs/manual/mc-manual.html)) |
| [内存泄漏](#leak) | 被抓( [LSAN](https://clang.llvm.org/docs/LeakSanitizer.html) ) | 被捕获([内存检查](https://www.valgrind.org/docs/manual/mc-manual.html)) |
| [数据竞争](#datarace) | 被抓( [TSAN](https://clang.llvm.org/docs/ThreadSanitizer.html) ) | 被抓([海尔格伦](https://www.valgrind.org/docs/manual/hg-manual.html)) |
| [重新编译](#recompilation) | 需要 | 不需要 |

在我的测试中，Valgrind 在启动时花了 23 秒读取系统调试信息文件。这种开销可以通过重命名`/usr/lib/debug`目录来暂时缓解。请不要忘记将其重新命名。否则，系统调试信息可能会丢失，同时不再可安装(因为它已经安装了):

```
$ sudo mv /usr/lib/debug /usr/lib/debug-x; sleep 1h; sudo mv /usr/lib/debug-x /usr/lib/debug
```

减速测试通过以下命令编译:

```
$ clang++ -g -O3 -march=native -ffast-math ...
clang-11.0.0-2.fc33.x86_64
valgrind-3.16.1-5.fc33.x86_64
```

这些命令使用普通的`-fsanitize=address`、`-fsanitize=undefined`或`-fsanitize=thread`，没有我在[杀毒软件操作指南](#tldr)中建议的任何额外选项。

本文使用了 Clang 编译器。GCC 对杀毒程序有同样的支持，除了编译器缺少对`-fsanitize=memory`的支持(在 [MSAN:未初始化内存读取](#uninit)一节中讨论过)。

## 安装 debuginfo

本文中的例子假设已经安装了`*-debuginfo.rpm`文件。你可以使用`dnf debuginfo-install *packagename*`来安装它们。在[红帽企业版 Linux](/products/rhel/overview) (RHEL)的部分版本上使用`yum`代替`dnf`。当你在`gdb`启动这个程序时，它会告诉你是否还缺少一些 rpm:

```
$ cat >vector.cpp <<'EOF'
// Here should be your program you are debugging,
// this std::vector sample code is just an example.
#include <vector>
int main() {
  std::vector<int> v;
}
EOF
$ clang++ -o vector vector.cpp -Wall -g
$ gdb ./vector
Reading symbols from ./vector...
(gdb) start
...
Missing separate debuginfos, use: dnf debuginfo-install glibc-2.32-2.fc33.x86_64
...
Missing separate debuginfos, use: dnf debuginfo-install libgcc-10.2.1-9.fc33.x86_64 libstdc++-10.2.1-9.fc33.x86_64
(gdb) quit
$ sudo dnf debuginfo-install glibc-2.32-2.fc33.x86_64
...
$ sudo dnf debuginfo-install libgcc-10.2.1-9.fc33.x86_64 libstdc++-10.2.1-9.fc33.x86_64
... 
```

## 示例案例:缓冲区溢出

下面的程序运行良好，尽管有一个缓冲区溢出错误。虽然这里的 bug 看起来很明显，但在真实世界的程序中，这样的 bug 更加隐蔽，难以发现。c 开发人员通常使用 [Valgrind Memcheck](https://www.valgrind.org/docs/manual/mc-manual.html) ，它可以捕捉到大多数错误。使用 Valgrind Memcheck 运行程序显示:

```
$ cat >overrun.c <<'EOF'
#include <stdlib.h>
int main() {
  char *p = malloc(16);
  p[24] = 1; // buffer overrun, p has only 16 bytes
  free(p); // free(): invalid pointer
  return 0;
}
EOF
$ clang -o overrun overrun.c -Wall -g
$ **valgrind** ./overrun
...
==60988== Invalid write of size 1
==60988==    at 0x401154: main (overrun.c:4)
==60988==  Address 0x4a52058 is 8 bytes after a block of size 16 alloc'd
==60988==    at 0x4839809: malloc (vg_replace_malloc.c:307)
==60988==    by 0x401147: main (overrun.c:3) 
```

**注意**:您可以[试用这段代码，并在编译器资源管理器中查看其输出](https://godbolt.org/z/1Wa6n4)。

因为这个程序在没有 Valgrind 的情况下运行良好，所以有人可能会认为没有 bug。在添加了模拟一个重要程序的代码后，这个示例即使在单独运行时也会崩溃。请注意消息`free(): invalid pointer`来自[glibc](https://en.wikipedia.org/wiki/GNU_C_Library)([GNU C 标准系统库](https://en.wikipedia.org/wiki/GNU_C_Library))而不是来自任何杀毒软件或 Valgrind 仪器:

```
$ cat >overrun.c <<'EOF'
#include <stdlib.h>
int main() {
  char *p = malloc(16);
  char *p2 = malloc(16);
  p[24] = 1; // buffer overrun, p has only 16 bytes
  free(p2); // free(): invalid pointer
  free(p);
  return 0;
}
EOF
$ clang -o overrun overrun.c -Wall -g
$ ./overrun
free(): invalid pointer
Aborted 
```

**注意**:您可以[试用这段代码，并在编译器资源管理器中查看其输出](https://godbolt.org/z/rvEjaK)。

针对此内存错误使用 Valgrind 对应的杀毒程序中的工具是 [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) 。不同之处在于，Valgrind 使用常规的可执行文件运行，而 AddressSanitizer 需要重新编译代码，然后直接执行，无需任何额外的工具:

```
$ clang -o overrun overrun.c -Wall -g **-fsanitize=address**
$ ./overrun
=================================================================
==61268==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000000028 at pc 0x0000004011b8 bp 0x7fff37c8aa70 sp 0x7fff37c8aa68
WRITE of size 1 at 0x602000000028 thread T0
    #0 0x4011b7 in main overrun.c:4
    #1 0x7f4c94a2d1e1 in __libc_start_main ../csu/libc-start.c:314
    #2 0x4010ad in _start (overrun+0x4010ad)

0x602000000028 is located 8 bytes to the right of 16-byte region [0x602000000010,0x602000000020)
allocated by thread T0 here:
    #0 0x7f4c94c7b3cf in __interceptor_malloc (/lib64/libasan.so.6+0xab3cf)
    #1 0x401177 in main overrun.c:3
    #2 0x7f4c94a2d1e1 in __libc_start_main ../csu/libc-start.c:314

SUMMARY: AddressSanitizer: heap-buffer-overflow overrun.c:4 in main
... 
```

**注意**:您可以[试用这段代码，并在编译器资源管理器中查看其输出](https://godbolt.org/z/3ns4WM)。

## ASAN:堆栈变量中的越界访问

因为 Valgrind 不需要重新编译程序，所以它不能检测一些无效的内存访问。其中一个错误是访问自动(局部)变量和全局变量范围之外的内存。(参见 [AddressSanitizer 堆栈越界文档](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleStackOutOfBounds)。)

因为 Valgrind 只在运行时参与，所以它从 [malloc](https://en.wikipedia.org/wiki/C_dynamic_memory_allocation#Overview_of_functions) 分配中隔离和跟踪内存。不幸的是，堆栈上的变量分配是已经编译好的程序所固有的，不需要调用任何外部函数，如`malloc`，所以 Valgrind 无法发现对堆栈内存的访问是否有效，或者只是一个不同堆栈对象的意外逃逸。另一方面，清理程序在编译时检测所有代码，此时编译器仍然知道程序试图访问堆栈上的哪个特定变量，以及该变量的正确堆栈边界是什么:

```
$ cat >stack.c <<'EOF'
int main(int argc, char **argv) {
  int a[100];
  return a[argc + 100];
}
EOF
$ clang -o stack stack.c -Wall -g **-fsanitize=address**
$ ./stack
==88682==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7fff54f500f4 at pc 0x0000004f4c51 bp 0x7fff54f4ff30 sp 0x7fff54f4ff28
READ of size 4 at 0x7fff54f500f4 thread T0
    #0 0x4f4c50 in main /tmp/stack.c:3:10
    #1 0x7f9983c7e1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #2 0x41c41d in _start (/tmp/stack+0x41c41d)

Address 0x7fff54f500f4 is located in stack of thread T0 at offset 436 in frame
    #0 0x4f4a9f in main /tmp/stack.c:1

  This frame has 1 object(s):
    [32, 432) 'a' (line 2) <== Memory access at offset 436 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism, swapcontext or vfork
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow /tmp/stack.c:3:10 in main
...
$ clang -o stack stack.c -Wall -g
$ **valgrind** ./stack
...
(nothing found by Valgrind) 
```

**注意**:您可以[试用这段代码，并在编译器资源管理器中查看其输出](https://godbolt.org/z/451nrc)。

## ASAN:全局变量中的越界访问

与堆栈上的变量一样，Valgrind 无法检测到全局变量溢出，因为它不会重新编译程序。(参见 [AddressSanitizer 全局越界文档](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleGlobalOutOfBounds)。)

我[之前描述过](#outofboundsstack)为什么 Valgrind 不能捕捉到这样的错误。以下是 AddressSanitizer 和 Valgrind 的输出:

```
$ cat >global.c <<'EOF'
int a[100];
int main(int argc, char **argv) {
  return a[argc + 100];
}
EOF
$ clang -o global global.c -Wall -g **-fsanitize=address**
$ ./global
=================================================================
==88735==ERROR: AddressSanitizer: global-buffer-overflow on address 0x000000dcee74 at pc 0x0000004f4b04 bp 0x7ffd5292b580 sp 0x7ffd5292b578
READ of size 4 at 0x000000dcee74 thread T0
    #0 0x4f4b03 in main /tmp/global.c:3:10
    #1 0x7fd416cda1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #2 0x41c41d in _start (/tmp/global+0x41c41d)

0x000000dcee74 is located 4 bytes to the right of global variable 'a' defined in 'global.c:1:5' (0xdcece0) of size 400
SUMMARY: AddressSanitizer: global-buffer-overflow /tmp/global.c:3:10 in main
...
$ clang -o global global.c -Wall -g
$ **valgrind** ./global
...
(nothing found by Valgrind) 
```

**注意**:您可以[试用这段代码，并在编译器资源管理器中查看其输出](https://godbolt.org/z/Ycx1qP)。

## MSAN:未初始化的内存读取

AddressSanitizer 不会检测对未初始化内存的读取。内存初始化器就是为此而开发的。它需要单独编译和运行。(参见[记忆启动器文档](https://clang.llvm.org/docs/MemorySanitizer.html)。)我不清楚为什么 AddressSanitizer 没有包括 MemorySanitizer 的功能，而且[我不是唯一一个](https://stackoverflow.com/a/38523606/2995591)这样的人。

内存初始化器运行如下:

```
$ cat >uninit.c <<'EOF'
int main(int argc, char **argv) {
  int a[2];
  if (a[argc != 1])
    return 1;
  else
    return 0;
}
EOF
$ clang -o uninit uninit.c -Wall -g -fsanitize=address -fsanitize=memory
clang-11: error: invalid argument '-fsanitize=address' not allowed with '-fsanitize=memory'
$ clang -o uninit uninit.c -Wall -g **-fsanitize=memory**
$ ./uninit
==63929==WARNING: MemorySanitizer: use-of-uninitialized-value
    #0 0x4985a9 in main /tmp/uninit.c:3:7
    #1 0x7f93e232c1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #2 0x41c39d in _start (/tmp/uninit+0x41c39d)
SUMMARY: MemorySanitizer: use-of-uninitialized-value /tmp/uninit.c:3:7 in main 
```

使用 Valgrind 可以更容易地捕获这个 bug，默认情况下它会报告未初始化的内存读取:

```
$ clang -o uninit uninit.c -Wall -g
$ **valgrind** ./uninit
...
==87991== Conditional jump or move depends on uninitialised value(s)
==87991==    at 0x401136: main (uninit.c:3)
... 
```

**注意**:您可以[试用这段代码，并在编译器资源管理器中查看其输出](https://godbolt.org/z/hW5xoa)。

## ASAN:返回后使用堆栈

AddressSanitizer 需要在运行时启用`ASAN_OPTIONS=detect_stack_use_after_return=1`，因为这个特性会带来额外的运行时开销。(参见 [AddressSanitizer 返回后使用文档](https://github.com/google/sanitizers/wiki/AddressSanitizerUseAfterReturn)。)以下是一个示例程序，它本身或使用 Valgrind 运行时没有错误，但使用 AddressSanitizer 运行时会显示错误:

```
$ cat >uar.cpp <<'EOF'
int *f() {
  int i = 42;
  int *p = &i;
  return p;
}
int g(int *p) {
  return *p;
}
int main() {
  return g(f());
}
EOF
$ clang++ -o uar uar.cpp -Wall -g **-fsanitize=address**
$ ./uar
(nothing found by default)
$ ASAN_OPTIONS=detect_stack_use_after_return=1 ./uar
=================================================================
==164341==ERROR: AddressSanitizer: stack-use-after-return on address 0x7fb71a561020 at pc 0x0000004f78e1 bp 0x7ffc299184c0 sp 0x7ffc299184b8
READ of size 4 at 0x7fb71a561020 thread T0
    #0 0x4f78e0 in g(int*) /home/lace/src/uar.cpp:7:10
    #1 0x4f790b in main /home/lace/src/uar.cpp:10:10
    #2 0x7fb71dbde1e1 in __libc_start_main (/lib64/libc.so.6+0x281e1)
    #3 0x41c41d in _start (/home/lace/src/uar+0x41c41d)

Address 0x7fb71a561020 is located in stack of thread T0 at offset 32 in frame
    #0 0x4f771f in f() /home/lace/src/uar.cpp:1

  This frame has 1 object(s):
    [32, 36) 'i' (line 2) <== Memory access at offset 32 is inside this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism, swapcontext or vfork
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-use-after-return /home/lace/src/uar.cpp:7:10 in g(int*)
...
$ clang++ -o uar uar.cpp -Wall -g
$ **valgrind** ./uar
...
(nothing found by Valgrind) 
```

## UBSAN:未定义的行为

UndefinedBehaviorSanitizer 保护代码免受语言标准所禁止的计算。(参见[未定义行为初始化器文档](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)。)出于性能原因，一些未定义的计算可能不会在运行时被捕获，但是如果它们被包含在内，没有人能保证程序的任何东西。最常见的是，这种数值表达式只是计算出一个意外的结果。未定义的行为初始化器可以检测和报告此类操作。

UndefinedBehaviorSanitizer 可以与最常见的杀毒软件 AddressSanitizer 一起使用:

```
$ cat >undefined.cpp <<'EOF'
int main(int argc, char **argv) {
  return 0x7fffffff + argc;
}
EOF
$ clang++ -o undefined undefined.cpp -Wall -g **-fsanitize=undefined**
$ export UBSAN_OPTIONS=print_stacktrace=1
$ ./undefined
undefined.cpp:2:21: runtime error: signed integer overflow: 2147483647 + 1 cannot be represented in type 'int'
    #0 0x429269 in main /tmp/undefined.cpp:2:21
    #1 0x7f1212a3e1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #2 0x40345d in _start (/tmp/undefined+0x40345d)

SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior undefined.cpp:2:21 in
$ **valgrind** ./undefined
...
(nothing found by Valgrind) 
```

**注意**:您可以[试用这段代码，并在编译器资源管理器中查看其输出](https://godbolt.org/z/aMP4sj)。

我个人倾向于在第一次出现这种情况时中止程序，因为否则很难找到 bug。因此我使用`-fno-sanitize-recover=all`。我也更喜欢通过包含:`-fsanitize=float-divide-by-zero -fsanitize=float-cast-overflow`来扩展未定义的行为初始化器的覆盖范围。

## LSAN:内存泄漏

LeakSanitizer 报告在程序完成之前尚未释放的已分配内存。(参见[防漏剂文档](https://clang.llvm.org/docs/LeakSanitizer.html)。)这样的行为[不一定是 bug](https://stackoverflow.com/q/654754/2995591) 。但是，释放所有已分配的内存会更容易，例如，捕捉真实的、意外的内存泄漏:

```
$ cat >leak.cpp <<'EOF'
#include <stdlib.h>
int main() {
  void *p = malloc(10);
  return p == nullptr;
}
EOF
$ clang++ -o leak leak.cpp -Wall -g **-fsanitize=address**
$ ./leak
=================================================================
==188539==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 10 byte(s) in 1 object(s) allocated from:
    #0 0x4bfcdf in malloc (/tmp/leak+0x4bfcdf)
    #1 0x4f7728 in main /tmp/leak.cpp:3:13
    #2 0x7fd5a7a781e1 in __libc_start_main (/lib64/libc.so.6+0x281e1)

SUMMARY: AddressSanitizer: 10 byte(s) leaked in 1 allocation(s).
$ clang++ -o leak leak.cpp -Wall -g
$ **valgrind --leak-check=full** ./leak
...
==188524== 10 bytes in 1 blocks are definitely lost in loss record 1 of 1
==188524==    at 0x4839809: malloc (vg_replace_malloc.c:307)
==188524==    by 0x401148: main (leak.cpp:3)
... 
```

**注意**:您可以[试用这段代码，并在编译器资源管理器中查看其输出](https://godbolt.org/z/chaf5j)。

## LSAN:特定库的内存泄漏(glib2)

一些框架有定制的内存分配器来阻止 LeakSanitizer 完成它的工作。下面的例子使用了这样一个框架， [glib2](https://en.wikipedia.org/wiki/GLib) (不是 [glibc](https://en.wikipedia.org/wiki/GNU_C_Library) )。其他库可能有其他运行时或编译时选项。LeakSanitizer 和 Valgrind 的输出如下:

```
$ cat >gc.c <<'EOF'
#include <glib.h>
int main(void) {
    GHashTable *ht = g_hash_table_new(g_str_hash, g_str_equal);
    g_hash_table_insert(ht, "foo", "bar");
//    g_hash_table_destroy(ht); // leak through glib2
    g_malloc(100); // direct leak
    return 0;
}
EOF
$ clang -o gc gc.c -Wall -g $(pkg-config --cflags --libs glib-2.0) **-fsanitize=address**
$ ./gc
=================================================================
==233215==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 100 byte(s) in 1 object(s) allocated from:
    #0 0x4bfd2f in malloc (/tmp/gc+0x4bfd2f)
    #1 0x7f1fcf12b908 in g_malloc (/lib64/libglib-2.0.so.0+0x5b908)
    #2 0x7f1fced961e1 in __libc_start_main (/lib64/libc.so.6+0x281e1)

SUMMARY: AddressSanitizer: 100 byte(s) leaked in 1 allocation(s).
$ clang -o gc gc.c -Wall -g $(pkg-config --cflags --libs glib-2.0)
$ **valgrind --leak-check=full** ./gc
...
==233250== 100 bytes in 1 blocks are definitely lost in loss record 8 of 11
==233250==    at 0x4839809: malloc (vg_replace_malloc.c:307)
==233250==    by 0x48DF908: g_malloc (in /usr/lib64/libglib-2.0.so.0.6600.3)
==233250==    by 0x4011C5: main (gc.c:6)
==233250==
==233250== 256 (96 direct, 160 indirect) bytes in 1 blocks are definitely lost in loss record 9 of 11
==233250==    at 0x4839809: malloc (vg_replace_malloc.c:307)
==233250==    by 0x48DF908: g_malloc (in /usr/lib64/libglib-2.0.so.0.6600.3)
==233250==    by 0x48F71C1: g_slice_alloc (in /usr/lib64/libglib-2.0.so.0.6600.3)
==233250==    by 0x48C5A51: g_hash_table_new_full (in /usr/lib64/libglib-2.0.so.0.6600.3)
==233250==    by 0x401197: main (gc.c:3)
... 
```

泄露的哈希表不是由 LeakSanitizer 报告的，而是由 Valgrind 报告的。这是因为 glib2 专门检测 Valgrind，并在 Valgrind 出现时关闭其自定义内存分配器( [g_slice](https://developer.gnome.org/glib/stable/glib-Memory-Slices.html) )。然而，即使使用了 LeakSanitizer，也可以强制 glib2 具有调试友好性:

```
$ clang -o gc gc.c -Wall -g $(pkg-config --cflags --libs glib-2.0) **-fsanitize=address**
# otherwise the backtraces would have only 2 entries:
$ export ASAN_OPTIONS=fast_unwind_on_malloc=0
# Show all glib2 memory leaks:
$ export G_SLICE=always-malloc G_DEBUG=gc-friendly
$ ./gc
=================================================================
==233921==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 100 byte(s) in 1 object(s) allocated from:
    #0 0x4bfd2f in malloc (/tmp/gc+0x4bfd2f)
    #1 0x7f2a7c302908 in g_malloc ../glib/gmem.c:106:13
    #2 0x4f4b35 in main /tmp/gc.c:6:5
    #3 0x7f2a7bf6d1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #4 0x41c46d in _start (/tmp/gc+0x41c46d)

Direct leak of 96 byte(s) in 1 object(s) allocated from:
    #0 0x4bfd2f in malloc (/tmp/gc+0x4bfd2f)
    #1 0x7f2a7c302908 in g_malloc ../glib/gmem.c:106:13
    #2 0x7f2a7c31a1c1 in g_slice_alloc ../glib/gslice.c:1069:11
    #3 0x7f2a7c2e8a51 in g_hash_table_new_full ../glib/ghash.c:1072:16
    #4 0x4f4b07 in main /tmp/gc.c:3:22
    #5 0x7f2a7bf6d1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #6 0x41c46d in _start (/tmp/gc+0x41c46d)

Indirect leak of 32 byte(s) in 1 object(s) allocated from:
    #0 0x4bfd2f in malloc (/tmp/gc+0x4bfd2f)
    #1 0x7f2a7c302908 in g_malloc ../glib/gmem.c:106:13
    #2 0x7f2a7c317ce1  ../glib/gstrfuncs.c:392:17
    #3 0x7f2a7c317ce1 in g_memdup ../glib/gstrfuncs.c:385:1
    #4 0x7f2a7c2e8b65 in g_hash_table_ensure_keyval_fits ../glib/ghash.c:974:36
    #5 0x7f2a7c2e8b65 in g_hash_table_insert_node ../glib/ghash.c:1327:3
    #6 0x7f2a7c2e930f in g_hash_table_insert_internal ../glib/ghash.c:1601:10
    #7 0x7f2a7c2e930f in g_hash_table_insert ../glib/ghash.c:1630:10
    #8 0x4f4b28 in main /tmp/gc.c:4:5
    #9 0x7f2a7bf6d1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #10 0x41c46d in _start (/tmp/gc+0x41c46d)

Indirect leak of 32 byte(s) in 1 object(s) allocated from:
    #0 0x4bfed7 in calloc (/tmp/gc+0x4bfed7)
    #1 0x7f2a7c302e20 in g_malloc0 ../glib/gmem.c:136:13
    #2 0x7f2a7c2e50ef in g_hash_table_setup_storage ../glib/ghash.c:592:24
    #3 0x7f2a7c2e8a90 in g_hash_table_new_full ../glib/ghash.c:1084:3
    #4 0x4f4b07 in main /tmp/gc.c:3:22
    #5 0x7f2a7bf6d1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #6 0x41c46d in _start (/tmp/gc+0x41c46d)

Indirect leak of 32 byte(s) in 1 object(s) allocated from:
    #0 0x4c0098 in realloc (/tmp/gc+0x4c0098)
    #1 0x7f2a7c302f5f in g_realloc ../glib/gmem.c:171:16
    #2 0x7f2a7c2e50da in g_hash_table_realloc_key_or_value_array ../glib/ghash.c:380:10
    #3 0x7f2a7c2e50da in g_hash_table_setup_storage ../glib/ghash.c:590:24
    #4 0x7f2a7c2e8a90 in g_hash_table_new_full ../glib/ghash.c:1084:3
    #5 0x4f4b07 in main /tmp/gc.c:3:22
    #6 0x7f2a7bf6d1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #7 0x41c46d in _start (/tmp/gc+0x41c46d)

SUMMARY: AddressSanitizer: 292 byte(s) leaked in 5 allocation(s). 
```

## TSAN:数据竞赛

ThreadSanitizer 报告多个线程在没有线程竞争保护的情况下访问数据的数据竞争。(参见 [ThreadSanitizer 文档](https://clang.llvm.org/docs/ThreadSanitizer.html)。)下面是一个例子:

```
$ cat >tiny.cpp <<'EOF'
#include <thread>

static volatile bool flip1{false};
static volatile bool flip2{false};

int main() {
  std::thread t([&]() {
    while (!flip1);
    flip2 = true;
  });
  flip1 = true;
  while (!flip2);
  t.join();
}
EOF
$ clang++ -o tiny tiny.cpp -Wall -g -pthread **-fsanitize=thread**
$ ./tiny
==================
WARNING: ThreadSanitizer: data race (pid=4057433)
  Write of size 1 at 0x000000fb4b09 by thread T1:
    #0 main::$_0::operator()() const /tmp/tiny.cpp:9:11 (tiny+0x4cfc98)
    #1 void std::__invoke_impl<void, main::$_0>(std::__invoke_other, main::$_0&&) /usr/lib/gcc/x86_64-redhat-linux/10/../../../../include/c++/10/bits/invoke.h:60:14 (tiny+0x4cfc30)
    #2 std::__invoke_result<main::$_0>::type std::__invoke<main::$_0>(main::$_0&&) /usr/lib/gcc/x86_64-redhat-linux/10/../../../../include/c++/10/bits/invoke.h:95:14 (tiny+0x4cfb40)
    #3 void std::thread::_Invoker<std::tuple<main::$_0> >::_M_invoke<0ul>(std::_Index_tuple<0ul>) /usr/lib/gcc/x86_64-redhat-linux/10/../../../../include/c++/10/thread:264:13 (tiny+0x4cfae8)
    #4 std::thread::_Invoker<std::tuple<main::$_0> >::operator()() /usr/lib/gcc/x86_64-redhat-linux/10/../../../../include/c++/10/thread:271:11 (tiny+0x4cfa88)
    #5 std::thread::_State_impl<std::thread::_Invoker<std::tuple<main::$_0> > >::_M_run() /usr/lib/gcc/x86_64-redhat-linux/10/../../../../include/c++/10/thread:215:13 (tiny+0x4cf97f)
    #6 execute_native_thread_routine ../../../../../libstdc++-v3/src/c++11/thread.cc:80:18 (libstdc++.so.6+0xd65f3)

  Previous read of size 1 at 0x000000fb4b09 by main thread:
    #0 main /tmp/tiny.cpp:12:11 (tiny+0x4cf51f)

  Location is global 'flip2' of size 1 at 0x000000fb4b09 (tiny+0x000000fb4b09)

  Thread T1 (tid=4057435, running) created by main thread at:
    #0 pthread_create <null> (tiny+0x488b7d)
    #1 <null> /usr/src/debug/gcc-10.2.1-9.fc33.x86_64/obj-x86_64-redhat-linux/x86_64-redhat-linux/libstdc++-v3/include/x86_64-redhat-linux/bits/gthr-default.h:663:35 (libstdc++.so.6+0xd6898)
    #2 std::thread::_M_start_thread(std::unique_ptr<std::thread::_State, std::default_delete<std::thread::_State> >, void (*)()) ../../../../../libstdc++-v3/src/c++11/thread.cc:135:37 (libstdc++.so.6+0xd6898)
    #3 main /tmp/tiny.cpp:7:15 (tiny+0x4cf4f4)

SUMMARY: ThreadSanitizer: data race /tmp/tiny.cpp:9:11 in main::$_0::operator()() const
==================
ThreadSanitizer: reported 1 warnings
$ clang++ -o tiny tiny.cpp -Wall -g -pthread
$ **valgrind --tool=helgrind** ./tiny
...
==4057510== ----------------------------------------------------------------
==4057510==
==4057510== Possible data race during write of size 1 at 0x40406D by thread #1
==4057510== Locks held: none
==4057510==    at 0x4011DC: main (tiny.cpp:11)
==4057510==
==4057510== This conflicts with a previous read of size 1 by thread #2
==4057510== Locks held: none
==4057510==    at 0x4015F8: main::$_0::operator()() const (tiny.cpp:8)
==4057510==    by 0x4015DC: void std::__invoke_impl<void, main::$_0>(std::__invoke_other, main::$_0&&) (invoke.h:60)
==4057510==    by 0x40156C: std::__invoke_result<main::$_0>::type std::__invoke<main::$_0>(main::$_0&&) (invoke.h:95)
==4057510==    by 0x401544: void std::thread::_Invoker<std::tuple<main::$_0> >::_M_invoke<0ul>(std::_Index_tuple<0ul>) (thread:264)
==4057510==    by 0x401514: std::thread::_Invoker<std::tuple<main::$_0> >::operator()() (thread:271)
==4057510==    by 0x40148D: std::thread::_State_impl<std::thread::_Invoker<std::tuple<main::$_0> > >::_M_run() (thread:215)
==4057510==    by 0x49575F3: execute_native_thread_routine (thread.cc:80)
==4057510==    by 0x4840737: mythread_wrapper (hg_intercepts.c:387)
==4057510==  Address 0x40406d is 0 bytes inside data symbol "_ZL5flip1"
==4057510==
==4057510== ----------------------------------------------------------------
==4057510==
==4057510== Possible data race during read of size 1 at 0x40406D by thread #2
==4057510== Locks held: none
==4057510==    at 0x4015F8: main::$_0::operator()() const (tiny.cpp:8)
==4057510==    by 0x4015DC: void std::__invoke_impl<void, main::$_0>(std::__invoke_other, main::$_0&&) (invoke.h:60)
==4057510==    by 0x40156C: std::__invoke_result<main::$_0>::type std::__invoke<main::$_0>(main::$_0&&) (invoke.h:95)
==4057510==    by 0x401544: void std::thread::_Invoker<std::tuple<main::$_0> >::_M_invoke<0ul>(std::_Index_tuple<0ul>) (thread:264)
==4057510==    by 0x401514: std::thread::_Invoker<std::tuple<main::$_0> >::operator()() (thread:271)
==4057510==    by 0x40148D: std::thread::_State_impl<std::thread::_Invoker<std::tuple<main::$_0> > >::_M_run() (thread:215)
==4057510==    by 0x49575F3: execute_native_thread_routine (thread.cc:80)
==4057510==    by 0x4840737: mythread_wrapper (hg_intercepts.c:387)
==4057510==    by 0x4BD33F8: start_thread (pthread_create.c:463)
==4057510==    by 0x4CED902: clone (clone.S:95)
==4057510==
==4057510== This conflicts with a previous write of size 1 by thread #1
==4057510== Locks held: none
==4057510==    at 0x4011DC: main (tiny.cpp:11)
==4057510==  Address 0x40406d is 0 bytes inside data symbol "_ZL5flip1"
==4057510==
==4057510== ----------------------------------------------------------------
==4057510==
==4057510== Possible data race during write of size 1 at 0x40406E by thread #2
==4057510== Locks held: none
==4057510==    at 0x401613: main::$_0::operator()() const (tiny.cpp:9)
==4057510==    by 0x4015DC: void std::__invoke_impl<void, main::$_0>(std::__invoke_other, main::$_0&&) (invoke.h:60)
==4057510==    by 0x40156C: std::__invoke_result<main::$_0>::type std::__invoke<main::$_0>(main::$_0&&) (invoke.h:95)
==4057510==    by 0x401544: void std::thread::_Invoker<std::tuple<main::$_0> >::_M_invoke<0ul>(std::_Index_tuple<0ul>) (thread:264)
==4057510==    by 0x401514: std::thread::_Invoker<std::tuple<main::$_0> >::operator()() (thread:271)
==4057510==    by 0x40148D: std::thread::_State_impl<std::thread::_Invoker<std::tuple<main::$_0> > >::_M_run() (thread:215)
==4057510==    by 0x49575F3: execute_native_thread_routine (thread.cc:80)
==4057510==    by 0x4840737: mythread_wrapper (hg_intercepts.c:387)
==4057510==    by 0x4BD33F8: start_thread (pthread_create.c:463)
==4057510==    by 0x4CED902: clone (clone.S:95)
==4057510==
==4057510== This conflicts with a previous read of size 1 by thread #1
==4057510== Locks held: none
==4057510==    at 0x4011E4: main (tiny.cpp:12)
==4057510==  Address 0x40406e is 0 bytes inside data symbol "_ZL5flip2"
==4057510==
==4057510== ----------------------------------------------------------------
==4057510==
==4057510== Possible data race during read of size 1 at 0x40406E by thread #1
==4057510== Locks held: none
==4057510==    at 0x4011E4: main (tiny.cpp:12)
==4057510==
==4057510== This conflicts with a previous write of size 1 by thread #2
==4057510== Locks held: none
==4057510==    at 0x401613: main::$_0::operator()() const (tiny.cpp:9)
==4057510==    by 0x4015DC: void std::__invoke_impl<void, main::$_0>(std::__invoke_other, main::$_0&&) (invoke.h:60)
==4057510==    by 0x40156C: std::__invoke_result<main::$_0>::type std::__invoke<main::$_0>(main::$_0&&) (invoke.h:95)
==4057510==    by 0x401544: void std::thread::_Invoker<std::tuple<main::$_0> >::_M_invoke<0ul>(std::_Index_tuple<0ul>) (thread:264)
==4057510==    by 0x401514: std::thread::_Invoker<std::tuple<main::$_0> >::operator()() (thread:271)
==4057510==    by 0x40148D: std::thread::_State_impl<std::thread::_Invoker<std::tuple<main::$_0> > >::_M_run() (thread:215)
==4057510==    by 0x49575F3: execute_native_thread_routine (thread.cc:80)
==4057510==    by 0x4840737: mythread_wrapper (hg_intercepts.c:387)
==4057510==  Address 0x40406e is 0 bytes inside data symbol "_ZL5flip2"
... 
```

**注意**:您可以[试用这段代码，并在编译器资源管理器中查看其输出](https://godbolt.org/z/xKjK5P)。

## 重新编译库

AddressSanitizer 自动处理所有打给 [glibc](https://en.wikipedia.org/wiki/GNU_C_Library) 的电话。对于其他系统或用户库，情况并非如此。为了让 AddressSanitizer 发挥最佳功能，还应该用`-fsanitize=address`重新编译这样的库。这在 Valgrind 中是不需要的。

由于 glibc 拦截器的存在，`libuser.c`库中的以下 bug 仍然会被 AddressSanitizer 捕获，即使该库不是用 AddressSanitizer 编译的:

```
$ cat >library.c <<'EOF'
#include <string.h>
void library(char *s) {
  strcpy(s,"string");
}
EOF
$ cat >libuser.c <<'EOF'
#include <stdlib.h>
void library(char *s);
int main(void) {
  char *s = malloc(1);
  library(s);
  free(s);
}
EOF
$ clang -o library.so library.c -Wall -g -shared -fPIC
$ clang -o libuser libuser.c -Wall -g ./library.so **-fsanitize=address**
$ ./libuser
=================================================================
==128657==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000000011 at pc 0x000000484a6d bp 0x7fff13a4ace0 sp 0x7fff13a4a490
WRITE of size 7 at 0x602000000011 thread T0
    #0 0x484a6c in __interceptor_strcpy.part.0 (/tmp/libuser+0x484a6c)
    #1 0x7fae9f53512b in library /tmp/library.c:3:3
    #2 0x4f4abe in main /tmp/libuser.c:5:3
    #3 0x7fae9f1be1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #4 0x41c42d in _start (/tmp/libuser+0x41c42d)

0x602000000011 is located 0 bytes to the right of 1-byte region [0x602000000010,0x602000000011)
allocated by thread T0 here:
    #0 0x4bfcef in malloc (/tmp/libuser+0x4bfcef)
    #1 0x4f4ab1 in main /tmp/libuser.c:4:13
    #2 0x7fae9f1be1e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16

SUMMARY: AddressSanitizer: heap-buffer-overflow (/tmp/libuser+0x484a6c) in __interceptor_strcpy.part.0
... 
```

在以下情况下，当库没有用 AddressSanitizer 重新编译时，AddressSanitizer 会错过内存损坏:

```
$ cat >library.c <<'EOF'
void library(char *s) {
  const char *cs = "string";
  while (*cs)
    *s++ = *cs++;
  *s = 0;
}
EOF
$ cat >libuser.c <<'EOF'
#include <stdlib.h>
void library(char *s);
int main(void) {
  char *s = malloc(1);
  library(s);
  free(s);
}
EOF
$ clang -o library.so library.c -Wall -g -shared -fPIC
$ clang -o libuser libuser.c -Wall -g ./library.so **-fsanitize=address**
$ ./libuser
(nothing found by AddressSanitizer) 
```

Valgrind 无需任何重新编译就能找到 bug:

```
$ clang -o library.so library.c -Wall -g -shared -fPIC; clang -o libuser libuser.c -Wall -g ./library.so; valgrind ./libuser
...
==128708== Invalid write of size 1
==128708==    at 0x4849146: library (library.c:4)
==128708==    by 0x40116E: main (libuser.c:5)
==128708==  Address 0x4a57041 is 0 bytes after a block of size 1 alloc'd
==128708==    at 0x4839809: malloc (vg_replace_malloc.c:307)
==128708==    by 0x401161: main (libuser.c:4)
... 
```

AddressSanitizer 也能发现 bug，只要我们用 AddressSanitizer 重新编译库:

```
$ clang -o library.so library.c -Wall -g -shared -fPIC **-fsanitize=address**
$ clang -o libuser libuser.c -Wall -g ./library.so **-fsanitize=address**
$ ./libuser
=================================================================
==128719==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000000011 at pc 0x7f7e4e68b269 bp 0x7ffc40c0dc30 sp 0x7ffc40c0dc28
WRITE of size 1 at 0x602000000011 thread T0
    #0 0x7f7e4e68b268 in library /tmp/library.c:4:10
    #1 0x4f4abe in main /tmp/libuser.c:5:3
    #2 0x7f7e4e3141e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16
    #3 0x41c42d in _start (/tmp/libuser+0x41c42d)

0x602000000011 is located 0 bytes to the right of 1-byte region [0x602000000010,0x602000000011)
allocated by thread T0 here:
    #0 0x4bfcef in malloc (/tmp/libuser+0x4bfcef)
    #1 0x4f4ab1 in main /tmp/libuser.c:4:13
    #2 0x7f7e4e3141e1 in __libc_start_main /usr/src/debug/glibc-2.32-20-g5c36293f06/csu/../csu/libc-start.c:314:16

SUMMARY: AddressSanitizer: heap-buffer-overflow /tmp/library.c:4:10 in library
... 
```

## 杀毒程序与 _FORTIFY_SOURCE 的交互

默认情况下，`rpmbuild`使用`-Wp,-D_FORTIFY_SOURCE=2`选项，它实现了自己的内存访问健全性检查。不幸的是，它禁用了 AddressSanitizer 所做的一些内存检查。这个问题[可能会在未来](https://github.com/google/sanitizers/issues/247)得到解决。目前，为了准备杀毒程序的检查，只需使用`-Wp,-U_FORTIFY_SOURCE`(这是简单`-D_FORTIFY_SOURCE=0`的一种更通用的形式)禁用`_FORTIFY_SOURCE`:

```
$ cat >strcpyfrom.spec <<'EOF'
Summary: strcpyfrom
Name: strcpyfrom
Version: 1
Release: 1
License: GPLv3+
%description
%build
cat >strcpyfrom.c <<'EOH'
#include <stdlib.h>
#include <string.h>
int main(void) {
  char *s = malloc(1);
  char d[0x1000];
  strcpy(d, s);
  return 0;
}
EOH
gcc -o strcpyfrom strcpyfrom.c $RPM_OPT_FLAGS **-fsanitize=address**
echo no error caught:
./strcpyfrom
gcc -o strcpyfrom strcpyfrom.c $RPM_OPT_FLAGS **-fsanitize=address -Wp,-U_FORTIFY_SOURCE**
echo error caught:
./strcpyfrom
EOF
$ rpmbuild -bb strcpyfrom.spec 
Executing(%build): /bin/sh -e /var/tmp/rpm-tmp.KTLr7c
+ umask 022
+ cd src/rpm/BUILD
+ cat
+ gcc -o strcpyfrom strcpyfrom.c -O2 -flto=auto -ffat-lto-objects -fexceptions -g -grecord-gcc-switches -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -fstack-protector-strong -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -fsanitize=address
+ echo no error caught:
no error caught:
+ ./strcpyfrom
+ gcc -o strcpyfrom strcpyfrom.c -O2 -flto=auto -ffat-lto-objects -fexceptions -g -grecord-gcc-switches -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -fstack-protector-strong -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -fsanitize=address -Wp,-U_FORTIFY_SOURCE
annobin: strcpyfrom.c: Warning: -D_FORTIFY_SOURCE defined as 0
+ echo error caught:
error caught:
+ ./strcpyfrom
=================================================================
==412157==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000000011 at pc 0x7fe75d8b2075 bp 0x7ffccf5dd1e0 sp 0x7ffccf5dc990
READ of size 2 at 0x602000000011 thread T0
    #0 0x7fe75d8b2074  (/lib64/libasan.so.6+0x52074)
    #1 0x4011be in main strcpyfrom.c:6
    #2 0x7fe75d6bd1e1 in __libc_start_main ../csu/libc-start.c:314
    #3 0x40127d in _start (strcpyfrom+0x40127d)

0x602000000011 is located 0 bytes to the right of 1-byte region [0x602000000010,0x602000000011)
allocated by thread T0 here:
    #0 0x7fe75d90b3cf in __interceptor_malloc (/lib64/libasan.so.6+0xab3cf)
    #1 0x4011b2 in main strcpyfrom.c:4
    #2 0x40200f  (strcpyfrom+0x40200f)

SUMMARY: AddressSanitizer: heap-buffer-overflow (/lib64/libasan.so.6+0x52074) 
...
error: Bad exit status from /var/tmp/rpm-tmp.KTLr7c (%build)

RPM build errors:
    Bad exit status from /var/tmp/rpm-tmp.KTLr7c (%build) 
```

## 结论

如果你已经习惯了 Valgrind，试试 address sanitizer——只需添加`-fsanitize=address`编译和链接参数(也就是说，添加到所有的`CFLAGS`、`CXXFLAGS`和`LDFLAGS`)作为第一次尝试。如果你觉得它很棒，请查看“[快速消毒指南](#tldr)”部分来微调体验。

*Last updated: October 14, 2022*