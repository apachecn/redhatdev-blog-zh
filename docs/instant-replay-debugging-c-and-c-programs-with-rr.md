# 如何用 rr 调试 C 和 C++程序

> 原文：<https://developers.redhat.com/blog/2021/05/03/instant-replay-debugging-c-and-c-programs-with-rr>

许多时间旅行电影的共同主题是回到过去，找出错误并修复它。开发人员也希望回到过去，找到代码出错的原因并修复它。但是，通常情况下，出问题的关键步骤发生在很久以前，并且信息不再可用。

[rr 项目](https://rr-project.org/)让程序员检查一个 [C 或 C++](/topics/c) 程序运行的整个生命周期，并重放代码执行，以查看过去是什么行为导致了“可怕的错误”`rr`与[Red Hat Enterprise Linux](/products/rhel/overview)(RHEL)中[Enterprise Linux(EPEL)](https://fedoraproject.org/wiki/EPEL)的额外软件包打包在一起，并与 Fedora 31、32、33 和 34 打包在一起。

记录关于应用程序执行的跟踪信息。这些信息允许您重复重放一个特定的失败记录，并在 [GNU 调试器](https://www.gnu.org/software/gdb/) (GDB)中检查它，以便更好地调查原因。除了重放跟踪之外，`rr`还允许您反向运行程序，本质上允许您“倒带”来查看在程序执行的早期发生了什么。

`rr`提供的用于记录再现者以供进一步检查的技术可以是对传统核心转储和回溯的有益补充，传统核心转储和回溯给出了特定时刻问题的快照。`rr`记录可以为开发人员提供一种方法来进一步调查间歇性问题，其中只有一些应用程序运行失败。

让我们看看如何设置`rr`并在一个例子中使用它来更好地说明它的效用。

## 要求和设置

因为`rr`使用了许多更新的 Linux 内核特性和特定的处理器性能监控硬件，所以它运行的环境是有限的。较新的 Fedora 内核拥有所需的支持。然而，为了正确地跟踪和重新创建线程中何时发生异步事件，`rr`使用性能监控硬件事件计数器来提供线程何时被异步事件中断的确定性计数。目前，`rr`仅支持使用 Westmere 或更新微体系结构的英特尔处理器的这些计数。

在 Fedora 上安装`rr`是一项简单的任务，只需要一个 RPM。如果你在 RHEL，你需要启用 EPEL。一旦您可以访问适当的存储库，您就可以用下面的命令安装`rr`:

```
$ sudo dnf -y install rr
```

由于硬件性能监控计数器可能会泄漏特权信息，许多内核默认限制它们的监控能力。您需要运行以下命令来允许`rr`访问性能监控计数器:

```
$ sudo sh -c 'echo 1 >/proc/sys/kernel/perf_event_paranoid'
```

如果您想使性能监控硬件的设置永久化，您也可以在`/etc/sysctl.conf`中添加下面一行:

```
kernel.perf_event_paranoid = 1
```

## 一个简单的调试例子

下面是一个玩具程序，它在一个数组中计算 2 乘以 0、1、2 和 3，并打印信息:

```
#include <stdio.h>
#define SIZE 4

void zero(char *a, int size)
{
	while (size>0)
		a[size--] = 0; 
}

void initialize(char *a, int size)
{
	zero(a, size);
}

void multiply(char *a, int size, int mult)
{
	int i;
	for (i=0; i<size; i++)
		a[i] = i * mult;
}

void pr_array(char *a, int size)
{
	int i;
	for (i=0; i<size; i++)
		printf("f(%d)=%d\n", i, a[i]);
}

int main(int argc, char **argv)
{
	char a[SIZE];
	int mult = 2;

	initialize(a, SIZE);
	multiply(a, SIZE, mult);
	pr_array(a, SIZE);
	return 0;
}

```

用通常的命令编译它，如下所示，并包含调试选项`-g`以允许以后的调试:

```
$ gcc -g multiply.c -o multiply
```

当你运行`multiply`时，结果可能会非常令人惊讶。所有结果都是零。主函数将 2 传递给`multiply`函数。该函数中的循环非常简单。这怎么可能出错呢？

```
$ ./multiply
f(0)=0
f(1)=0
f(2)=0
f(3)=0
```

### 记录问题

可以调查一下`rr`哪里出问题了。用下面的命令记录一个演示问题的`multiply`程序的运行。如果问题是间歇性的，您可以多次运行，直到问题出现:

```
$ rr record ./multiply
rr: Saving execution to trace directory `/home/wcohen/.local/share/rr/multiply-0`.
f(0)=0
f(1)=0
f(2)=0
f(3)=0

```

### 重现和调查问题

要开始调试这个问题，请使用`rr replay`命令，这将使您进入一个 GDB 会话:

```
$ rr replay

```

你在执行的开始:

```
(rr) where
#0 0x00007f7f56b1f110 in _start () from /lib64/ld-linux-x86-64.so.2
#1 0x0000000000000001 in ?? ()
#2 0x00007ffe0bea6889 in ?? ()
#3 0x0000000000000000 in ?? ()

```

您可以从头开始继续该程序，并看到它具有与之前相同的结果:

```
(rr) c
Continuing.
f(0)=0
f(1)=0
f(2)=0
f(3)=0

Program received signal SIGKILL, Killed.
0x0000000070000002 in ?? ()

```

您可以在`main`函数中的`return 0`处设置一个断点，并从那里向后操作:

```
(rr) break multiply.c:37
Breakpoint 1 at 0x401258: file multiply.c, line 37.
(rr) c
Continuing.
f(0)=0
f(1)=0
f(2)=0
f(3)=0

Breakpoint 1, main (argc=1, argv=0x7ffe0bea5c58) at multiply.c:37
37 return 0;

```

首先，我们检查数组`a`中的值。可能`pr_array`功能打印了错误的值。但这不是问题所在，因为根据 GDB 的理论，这些值都是 0:

```
(rr) print a[0]
$5 = 0 '\000'
(rr) print a[1]
$6 = 0 '\000'
(rr) print a[2]
$7 = 0 '\000'
(rr) print a[3]
$8 = 0 '\000'

```

也许`pr_array`破坏了价值观。让我们在`pr_array`的入口设置一个断点，用`reverse-continue`命令向后执行，看看在`pr_array`函数执行之前是什么状态。看起来`pr_array`正在打印正确的值:

```
(rr) break pr_array
Breakpoint 2 at 0x4011cc: file multiply.c, line 25.
(rr) reverse-continue
Continuing.

Breakpoint 2, pr_array (a=0x7ffe0bea5b58 "", size=4) at multiply.c:25
25 for (i=0; i<size; i++)
(rr) print a[0]
$9 = 0 '\000'
(rr) print a[1]
$10 = 0 '\000'
(rr) print a[2]
$11 = 0 '\000'
(rr) print a[3]
$12 = 0 '\000'

```

可能是`multiply`函数出问题了。让我们在它上面设置一个断点并给它设置`reverse-continue`:

```
rr) break multiply
Breakpoint 3 at 0x401184: file multiply.c, line 18.
(rr) reverse-continue
Continuing.

Breakpoint 3, multiply (a=0x7ffe0bea5b58 "", size=4, mult=0) at multiply.c:18
18 for (i=0; i0)
(rr) c
Continuing.

```

### 等等！

`mult`论证怎么了？应该是 2。零乘以任何东西都是零。这解释了结果。然而，`main`函数的局部变量`mult`怎么会最终为零呢？它在`main`中初始化，并且只传递给计算函数。让我们在`mult`上设置一个硬件观察点，继续程序的逆向执行:

```
(rr) up
#1  0x0000000000401247 in main (argc=1, argv=0x7ffe0bea5c58) at multiply.c:35
35		multiply(a, SIZE, mult);
(rr) watch -l mult
Hardware watchpoint 4: -location mult
(rr) reverse-continue
Continuing.

Hardware watchpoint 4: -location mult

Old value = 0
New value = 2
zero (a=0x7ffe0bea5b58 "", size=3) at multiply.c:7
7			a[size--] = 0; 

```

啊，现在事情变得越来越清楚了。`zero`函数超过了`a`数组的末尾，并覆盖了`mult`变量，尽管它没有被传入。`size--`的说法应该是`--size`。我们可以看到对隐藏在`initialize`函数中的`zero`的调用:

```
(rr) where
#0  zero (a=0x7ffe0bea5b58 "", size=3) at multiply.c:7
#1  0x0000000000401173 in initialize (a=0x7ffe0bea5b58 "", size=4)
    at multiply.c:12
#2  0x0000000000401233 in main (argc=1, argv=0x7ffe0bea5c58) at multiply.c:34

```

如果我们愿意，我们可以使用常规的 GDB 继续，向前播放，并再次通过我们之前设置的断点:

```
(rr) c
Continuing.

Hardware watchpoint 4: -location mult

Old value = 2
New value = 0
zero (a=0x7ffe0bea5b58 "", size=3) at multiply.c:6
6		while (size>0)
(rr) c
Continuing.

Breakpoint 3, multiply (a=0x7ffe0bea5b58 "", size=4, mult=0) at multiply.c:18
18		for (i=0; i<size; i++)
(rr) c
Continuing.

Breakpoint 2, pr_array (a=0x7ffe0bea5b58 "", size=4) at multiply.c:25
25		for (i=0; i<size; i++)

```

### 修复问题

既然我们已经确定了问题，我们可以修复“新的和改进的”`multiply2.c`程序中的`zero`函数，事情就像预期的那样工作了:

```
$ gcc -g multiply2.c -o multiply2
$ ./multiply2
f(0)=0
f(1)=2
f(2)=4
f(3)=6

```

## rr 的局限性

虽然`rr`是帮助开发人员发现程序问题的有用工具，但它也有局限性:

*   `rr`记录的所有线程都在一个内核上运行，所以多线程应用会比较慢。
*   系统调用监控并不完整，所以一些系统调用可能会从漏洞中溜走，而不会被记录在跟踪中。
*   `rr`使用`ptrace`来监控应用程序，但不能与使用`ptrace`的应用程序配合使用。
*   不监视它所记录的子进程之外的进程，并且错过通过共享内存与外部进程的任何通信。
*   仅在非常特殊的处理器微体系结构上运行。

## 结论

`rr`在程序执行过程中向后追溯以调查导致问题的早期事件的能力是对开发人员工具集的有益补充。本文中的例子说明了如何使用`rr`来追踪问题。这个例子只是一个玩具，但是`rr`已经被用于跟踪更实际的应用程序中的问题，比如 [JavaScriptCore](https://blog.ret2.io/2018/06/19/pwn2own-2018-root-cause-analysis/) 、 [Apache httpd 服务器](https://animal0day.blogspot.com/2017/07/from-fuzzing-apache-httpd-server-to-cve.html)和 [Ruby on Rails](https://www.freshworks.com/saas/debugging-memory-corruption-in-production-rails-app-using-mozilla-rr-blog/) 。

有关其他信息，请参见以下资源:

*   [RR-项目页面](https://rr-project.org/)
*   [可部署性扩展技术报告的工程记录和回放](https://arxiv.org/pdf/1705.05937.pdf)

*Last updated: October 14, 2022*