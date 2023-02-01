# GCC 10 中的静态分析

> 原文：<https://developers.redhat.com/blog/2020/03/26/static-analysis-in-gcc-10>

我在 GCC 的 Red Hat 工作，GNU 编译器集合。对于 GCC 的下一个主要版本， [GCC 10](https://gcc.gnu.org/gcc-10/changes.html) ，我已经实现了一个新的`-fanalyzer`选项:一个静态分析过程，在编译时而不是运行时识别各种问题。

我的想法是，最好在编写代码时尽早发现问题，在编译-编辑-调试循环中使用编译器编写代码，而不是将静态分析作为额外的工具“放在一边”(可能是专有的)。因此，在编译器中内置一个静态分析器似乎是值得的，它可以看到与编译器看到的完全相同的代码——因为它*是*编译器。

这个问题当然是一个需要解决的大问题。对于这个版本，我关注的是在 C 代码中看到的各种问题——特别是[双自由 bug](https://cwe.mitre.org/data/definitions/415.html)——但是着眼于创建一个我们可以在后续版本中扩展的框架(当我们可以添加更多检查并支持 C 之外的语言时)。

我希望分析器能提供相当多的额外检查，同时又不会太贵。我的目标是`-fanalyzer`“仅仅”加倍编译时间，作为额外检查的合理折衷。我还没有成功，你会在下面看到，但我正在努力。

目前，代码在 GCC 的 GCC 10 主分支中，可以在[编译器资源管理器，也就是 godbolt.org](http://godbolt.org/)上试用。对于中小型的例子来说，它工作得很好，但是有一些 bug 意味着它还不能用于生产。我正在努力修复一些东西，希望在 GCC 10 发布时(可能在 4 月份)，这个特性能够有意义地用于 C 代码。

## 诊断路径

下面是一个最简单的双自由错误的例子:

```
#include <stdlib.h>

void test(void *ptr)
{
  free(ptr);
  free(ptr);
}

```

带有`-fanalyzer`的 GCC 10 报告如下:

```
$ gcc -c -fanalyzer double-free-1.c
double-free-1.c: In function ‘test’:
double-free-1.c:6:3: warning: double-‘free’ of ‘ptr’ [CWE-415] [-Wanalyzer-double-free]
    6 |   free(ptr);
      |   ^~~~~~~~~
  ‘test’: events 1-2
    |
    |    5 |   free(ptr);
    |      |   ^~~~~~~~~
    |      |   |
    |      |   (1) first ‘free’ here
    |    6 |   free(ptr);
    |      |   ~~~~~~~~~
    |      |   |
    |      |   (2) second ‘free’ here; first ‘free’ was at (1)
    |

```

这种反应说明海合会学会了一些新花样；首先，诊断具有[通用弱点枚举(CWE)标识符](https://cwe.mitre.org/)的能力。在本例中，双自由诊断标记有 [CWE-415](https://cwe.mitre.org/data/definitions/415.html) 。这个标记有望使输出更加清晰，提高精度，并让您可以简单地在搜索引擎中输入内容。到目前为止，只有来自`-fanalyzer`的诊断被贴上了 CWE 弱点标识符的标签。

如果你使用 GCC 10 和一个合适的终端(例如最近的 gnome-terminal)，CWE 标识符是一个可点击的超链接，带你到问题的[描述。说到超链接，对于许多版本，当 GCC 发出警告时，它会打印控制该警告的选项。从 GCC 10 开始，那个选项文本现在是一个可点击的超链接(同样，假设一个](https://cwe.mitre.org/data/definitions/415.html)[有足够能力的终端](https://gist.github.com/egmontkob/eb114294efbcd5adb1944c9f3cb5feda))，它应该带你到那个选项的文档(对于任何警告，不仅仅是那些与分析器相关的)。

其次，GCC 诊断现在可以有一系列与之相关的事件，描述触发问题的代码路径。鉴于上例中缺少控制流，它只有两个事件，但是您可以看到第二个事件是如何在描述中引用第一个事件的。

这里有一个更复杂的例子。您能在下面的代码中发现问题吗？(提示:这次不是双免):

```
#include <setjmp.h>
#include <stdlib.h>

static jmp_buf env;

static void inner(void)
{
  longjmp(env, 1);
}

static void middle(void)
{
  void *ptr = malloc(1024);
  inner();
  free(ptr);
}

void outer(void)
{
  int i;

  i = setjmp(env);
  if (i == 0)
    middle();
}

```

下面是 GCC 的`-fanalyzer`报告，它通过 ASCII art 显示了过程间控制流:

```
$ gcc -c -fanalyzer longjmp-demo.c
longjmp-demo.c: In function ‘inner’:
longjmp-demo.c:8:3: warning: leak of ‘ptr’ [CWE-401] [-Wanalyzer-malloc-leak]
    8 |   longjmp(env, 1);
      |   ^~~~~~~~~~~~~~~
  ‘outer’: event 1
    |
    |   18 | void outer(void)
    |      |      ^~~~~
    |      |      |
    |      |      (1) entry to ‘outer’
    |
  ‘outer’: event 2
    |
    |   22 |   i = setjmp(env);
    |      |       ^~~~~~
    |      |       |
    |      |       (2) ‘setjmp’ called here
    |
  ‘outer’: events 3-5
    |
    |   23 |   if (i == 0)
    |      |      ^
    |      |      |
    |      |      (3) following ‘true’ branch (when ‘i == 0’)...
    |   24 |     middle();
    |      |     ~~~~~~~~
    |      |     |
    |      |     (4) ...to here
    |      |     (5) calling ‘middle’ from ‘outer’
    |
    +--> ‘middle’: events 6-8
           |
           |   11 | static void middle(void)
           |      |             ^~~~~~
           |      |             |
           |      |             (6) entry to ‘middle’
           |   12 | {
           |   13 |   void *ptr = malloc(1024);
           |      |               ~~~~~~~~~~~~
           |      |               |
           |      |               (7) allocated here
           |   14 |   inner();
           |      |   ~~~~~~~
           |      |   |
           |      |   (8) calling ‘inner’ from ‘middle’
           |
           +--> ‘inner’: events 9-11
                  |
                  |    6 | static void inner(void)
                  |      |             ^~~~~
                  |      |             |
                  |      |             (9) entry to ‘inner’
                  |    7 | {
                  |    8 |   longjmp(env, 1);
                  |      |   ~~~~~~~~~~~~~~~
                  |      |   |
                  |      |   (10) ‘ptr’ leaks here; was allocated at (7)
                  |      |   (11) rewinding from ‘longjmp’ in ‘inner’...
                  |
    <-------------+
    |
  ‘outer’: event 12
    |
    |   22 |   i = setjmp(env);
    |      |       ^~~~~~
    |      |       |
    |      |       (12) ...to ‘setjmp’ in ‘outer’ (saved at (2))
    |

```

上面的内容相当冗长，尽管考虑到使用了`setjmp`和`longjmp`，它可能需要传达正在发生的事情。我希望描述得相当清楚:当对`longjmp`的调用越过`middle`中的清理点将堆栈退回到`outer`，而没有调用清理时，会发生内存泄漏。

如果您不喜欢上面的 ASCII 艺术，您可以使用`-fdiagnostics-path-format=separate-events`将事件视为单独的“注释”诊断:

```
$ gcc -c -fanalyzer -fdiagnostics-path-format=separate-events longjmp-demo.c
longjmp-demo.c: In function ‘inner’:
longjmp-demo.c:8:3: warning: leak of ‘ptr’ [CWE-401] [-Wanalyzer-malloc-leak]
    8 |   longjmp(env, 1);
      |   ^~~~~~~~~~~~~~~
longjmp-demo.c:18:6: note: (1) entry to ‘outer’
   18 | void outer(void)
      |      ^~~~~
In file included from longjmp-demo.c:1:
longjmp-demo.c:22:7: note: (2) ‘setjmp’ called here
   22 |   i = setjmp(env);
      |       ^~~~~~
longjmp-demo.c:23:6: note: (3) following ‘true’ branch (when ‘i == 0’)...
   23 |   if (i == 0)
      |      ^
longjmp-demo.c:24:5: note: (4) ...to here
   24 |     middle();
      |     ^~~~~~~~
longjmp-demo.c:24:5: note: (5) calling ‘middle’ from ‘outer’
longjmp-demo.c:11:13: note: (6) entry to ‘middle’
   11 | static void middle(void)
      |             ^~~~~~
longjmp-demo.c:13:15: note: (7) allocated here
   13 |   void *ptr = malloc(1024);
      |               ^~~~~~~~~~~~
longjmp-demo.c:14:3: note: (8) calling ‘inner’ from ‘middle’
   14 |   inner();
      |   ^~~~~~~
longjmp-demo.c:6:13: note: (9) entry to ‘inner’
    6 | static void inner(void)
      |             ^~~~~
longjmp-demo.c:8:3: note: (10) ‘ptr’ leaks here; was allocated at (7)
    8 |   longjmp(env, 1);
      |   ^~~~~~~~~~~~~~~
longjmp-demo.c:8:3: note: (11) rewinding from ‘longjmp’ in ‘inner’...
In file included from longjmp-demo.c:1:
longjmp-demo.c:22:7: note: (12) ...to ‘setjmp’ in ‘outer’ (saved at (2))
   22 |   i = setjmp(env);
      |       ^~~~~~

```

或者用`-fdiagnostics-path-format=none`把它们一起关掉。还有一种 JSON 输出格式。

所有的新诊断都有一个`-Wanalyzer-SOMETHING`名称:我们已经在上面看到了`-Wanalyzer-double-free`和`-Wanalyzer-malloc-leak`。当`-fanalyzer`被启用时，这些诊断都被启用，但是它们可以通过`-Wno-analyzer-SOMETHING`变体(例如，通过 pragmas)被选择性地禁用。

## 有哪些新的警示？

除了双自由检测，还有对`malloc`和`fopen`泄漏的检查:

```
#include <stdio.h>
#include <stdlib.h>

void test(const char *filename)
{
  FILE *f = fopen(filename, "r");
  void *p = malloc(1024);
  /* do stuff */
}

```

```
$ gcc -c -fanalyzer leak.c
leak.c: In function ‘test’:
leak.c:9:1: warning: leak of ‘p’ [CWE-401] [-Wanalyzer-malloc-leak]
    9 | }
      | ^
  ‘test’: events 1-2
    |
    |    7 |   void *p = malloc(1024);
    |      |             ^~~~~~~~~~~~
    |      |             |
    |      |             (1) allocated here
    |    8 |   /* do stuff */
    |    9 | }
    |      | ~
    |      | |
    |      | (2) ‘p’ leaks here; was allocated at (1)
    |
leak.c:9:1: warning: leak of FILE ‘f’ [CWE-775] [-Wanalyzer-file-leak]
    9 | }
      | ^
  ‘test’: events 1-2
    |
    |    6 |   FILE *f = fopen(filename, "r");
    |      |             ^~~~~~~~~~~~~~~~~~~~
    |      |             |
    |      |             (1) opened here
    |......
    |    9 | }
    |      | ~
    |      | |
    |      | (2) ‘f’ leaks here; was opened at (1)
    |

```

对于释放内存后使用内存:

```
#include <stdlib.h>

struct link { struct link *next; };

int free_a_list_badly(struct link *n)
{
  while (n) {
    free(n);
    n = n->next;
  }
}

```

```
$ gcc -c -fanalyzer use-after-free.c
use-after-free.c: In function ‘free_a_list_badly’:
use-after-free.c:9:7: warning: use after ‘free’ of ‘n’ [CWE-416] [-Wanalyzer-use-after-free]
    9 |     n = n->next;
      |     ~~^~~~~~~~~
  ‘free_a_list_badly’: events 1-4
    |
    |    7 |   while (n) {
    |      |         ^
    |      |         |
    |      |         (1) following ‘true’ branch (when ‘n’ is non-NULL)...
    |    8 |     free(n);
    |      |     ~~~~~~~
    |      |     |
    |      |     (2) ...to here
    |      |     (3) freed here
    |    9 |     n = n->next;
    |      |     ~~~~~~~~~~~
    |      |       |
    |      |       (4) use after ‘free’ of ‘n’; freed at (3)
    |

```

释放非堆指针:

```
#include <stdlib.h>

void test(int n)
{
  int buf[10];
  int *ptr;

  if (n < 10)
    ptr = buf;
  else
    ptr = (int *)malloc(sizeof (int) * n);

  /* do stuff.  */

  /* oops; this free should be conditionalized.  */
  free(ptr);
}

```

```
$ gcc -c -fanalyzer heap-vs-stack.c
heap-vs-stack.c: In function ‘test’:
heap-vs-stack.c:16:3: warning: ‘free’ of ‘ptr’ which points to memory not on the heap [CWE-590] [-Wanalyzer-free-of-non-heap]
   16 |   free(ptr);
      |   ^~~~~~~~~
  ‘test’: events 1-4
    |
    |    8 |   if (n < 10)
    |      |      ^
    |      |      |
    |      |      (1) following ‘true’ branch (when ‘n <= 9’)...
    |    9 |     ptr = buf;
    |      |     ~~~~~~~~~
    |      |         |
    |      |         (2) ...to here
    |      |         (3) pointer is from here
    |......
    |   16 |   free(ptr);
    |      |   ~~~~~~~~~
    |      |   |
    |      |   (4) call to ‘free’ here
    |

```

对于在信号处理程序中使用已知不安全的函数:

```
#include <stdio.h>
#include <signal.h>

extern void body_of_program(void);

void custom_logger(const char *msg)
{
  fprintf(stderr, "LOG: %s", msg);
}

static void handler(int signum)
{
  custom_logger("got signal");
}

int main(int argc, const char *argv)
{
  custom_logger("started");

  signal(SIGINT, handler);

  body_of_program();

  custom_logger("stopped");

  return 0;
}

```

```
$ gcc -c -fanalyzer signal.c
signal.c: In function ‘custom_logger’:
signal.c:8:3: warning: call to ‘fprintf’ from within signal handler [CWE-479] [-Wanalyzer-unsafe-call-within-signal-handler]
    8 |   fprintf(stderr, "LOG: %s", msg);
      |   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  ‘main’: events 1-2
    |
    |   16 | int main(int argc, const char *argv)
    |      |     ^~~~
    |      |     |
    |      |     (1) entry to ‘main’
    |......
    |   20 |   signal(SIGINT, handler);
    |      |   ~~~~~~~~~~~~~~~~~~~~~~~
    |      |   |
    |      |   (2) registering ‘handler’ as signal handler
    |
  event 3
    |
    |cc1:
    | (3): later on, when the signal is delivered to the process
    |
    +--> ‘handler’: events 4-5
           |
           |   11 | static void handler(int signum)
           |      |             ^~~~~~~
           |      |             |
           |      |             (4) entry to ‘handler’
           |   12 | {
           |   13 |   custom_logger("got signal");
           |      |   ~~~~~~~~~~~~~~~~~~~~~~~~~~~
           |      |   |
           |      |   (5) calling ‘custom_logger’ from ‘handler’
           |
           +--> ‘custom_logger’: events 6-7
                  |
                  |    6 | void custom_logger(const char *msg)
                  |      |      ^~~~~~~~~~~~~
                  |      |      |
                  |      |      (6) entry to ‘custom_logger’
                  |    7 | {
                  |    8 |   fprintf(stderr, "LOG: %s", msg);
                  |      |   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                  |      |   |
                  |      |   (7) call to ‘fprintf’ from within signal handler
                  |

```

连同[其他警告](https://gcc.gnu.org/onlinedocs/gcc/Static-Analyzer-Options.html)。

## 还能做什么？

就目前情况而言，该检查器在中小型示例中运行良好，但是当我将其扩展到真实的 C 代码时，遇到了两个问题。首先，我的状态管理代码中有一些错误。在检查器中有以抽象方式描述程序状态的类。检查器研究程序，用逻辑构建(点，状态)对的有向图，用于在控制流连接点简化状态和合并状态。

理论上，如果状态变得太复杂，检查器应该进入最少定义的状态，但是这种方法存在缺陷，会导致给定点的状态数量激增，从而导致检查器运行缓慢，最终达到安全极限，无法完全探索程序。为了解决这个问题，我已经重写了状态管理代码。我希望下周能在《大师》中看到重写本。

第二，即使我们完全探索了程序，通过由`-fanalyzer`生成的代码的路径有时是可笑的冗长。我见过的最糟糕的情况是编译 GCC 本身时报告的使用未初始化数据的 110 事件路径。我认为这是一个误报，但显然期望用户费力地完成这样的事情是不合理的。

分析器试图通过(点，状态)图找到最短的可行路径，从中生成一个事件链，然后试图简化这个链。实际上，它对事件链应用了一系列的窥视孔优化，以得出表达问题的最小链。

我最近实现了一种从路径中过滤不相关的控制流边的方法，这应该会有所帮助，并且我正在开发一个类似的补丁来消除冗余的过程间边。

举个具体的例子，我对一个真实的 bug(尽管是 15 年前的 bug)——[CVE-2005-1689](https://access.redhat.com/security/cve/cve-2005-1689)，krb5 1.4.1 中的一个双自由漏洞进行了测试。它正确地识别了 bug，没有误报，但是输出目前是 170 行 stderr。您可以在[链接](https://dmalcolm.fedorapeople.org/gcc/2020-02-28/recvauth.c.html)处看到输出，而不是在这里内联显示。

最初，上面是 1187 行 stderr。我修复了各种错误，并实现了更多的简化，使其减少到 170 行。部分问题在于，`free`是通过一个`krb5_xfree`宏完成的，路径打印代码显示了每次宏中发生事件时每个宏是如何展开的。也许每个诊断的输出应该只显示每个宏展开一次。此外，每个诊断中的前几个事件是过程间逻辑，与用户并不真正相关(我正在解决这个问题)。有了这些变化，输出应该大大缩短。

也许一个更好的界面可以写出一个单独的 HTML 文件，每个警告一个，并发出一个“注释”给出附加信息的位置？

我希望给最终用户足够的信息来对警告采取行动，但又不至于让他们不知所措。有没有更好的表达方式？请在评论中告诉我。

## 尝试一下

GCC 10 将在 Fedora 32 中发布，它将在几个月后发布。

对于简单的代码示例，您可以在[godbolt.org](https://godbolt.org/)在线试用新的 gcc(选择 GCC“主干”并将`-fanalyzer`添加到编译器选项中)。

玩得开心！

*Last updated: June 29, 2020*