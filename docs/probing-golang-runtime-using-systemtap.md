# 使用 SystemTap 探测 golang 运行时

> 原文：<https://developers.redhat.com/blog/2019/07/24/probing-golang-runtime-using-systemtap>

我最近看到了优步工程的一篇[文章，描述了他们面临的延迟增加的问题。优步的工程师怀疑他们的代码耗尽了堆栈空间，导致 golang 运行时发出堆栈增长，这将由于内存分配和复制而引入额外的延迟。工程师们最终用额外的工具修改了 golang 运行时，以报告这些堆栈增长来证实他们的怀疑。这种情况是可以使用 SystemTap 的一个很好的例子。](https://eng.uber.com/optimizing-m3/)

SystemTap 是一个工具，可以用来对正在运行的程序进行实时分析。它能够中断正常的控制流并执行 SystemTap 脚本指定的代码，这允许用户临时修改正在运行的程序，而不必更改源代码并重新编译。

下面是一个 SystemTap 脚本，可用于复制优步文章中的补丁:

```
global printstackgrow

probe process(*PATH*).statement("runtime.newstack@/usr/lib/golang/src/runtime/stack.go:936")
{
  shouldPrintStack = printstackgrow % 1000 == 0
  printstackgrow++

  if (shouldPrintStack) {
    oldSize = $thisg->m->curg->stack->hi - $thisg->m->curg->stack->lo
    newSize = oldSize * 2
    printf("runtime: newstack: %d -> %d\n", oldSize, newSize)
    print_ubacktrace_fileline($thisg->m->morebuf->pc,
                              $thisg->m->morebuf->sp,
                              $thisg->m->morebuf->lr)
  }
}

```

这个脚本在 stack.go 的第 936 行的`runtime.newstack()`函数中引入了一个探针(这个位置取决于您的 golang 安装),用于由*路径*指定的可执行文件。当我们执行到这一点时，SystemTap 将重定向并执行上面显示的探针处理程序中的代码。处理程序执行与优步文章补丁中相同的步骤，打印出栈大小的变化和每 1000 次点击`runtime.newstack()`的回溯。

现在让我们看看这个脚本的运行情况。我将使用下面的 golang 示例来演示。这个程序打印出一些文本并生成 go 例程来做同样的事情，只是为了到达一个需要增加堆栈并调用`runtime.newstack()`的点。

```
$ cat example.go
package main

import (
  "fmt"
  "time"
)

func say(s string)
{
  for i := 0; i < 5; i++ {
    time.Sleep(1 * time.Millisecond)
    fmt.Println(s)
  }
}

func main()
{
  go say("world")
  say("hello")
}

```

在这个演示中，我将稍微简化 SystemTap 脚本。不需要限制输出的采样代码，因为这个例子不会有太多的输出。现在我们也有了一个实际的可执行文件，使用`go build example.go`生成，可以用来替换*路径*。

```
$ cat newstack.stp
probe process("./example").statement("runtime.newstack@/usr/lib/golang/src/runtime/stack.go:936")
{
  oldSize = $thisg->m->curg->stack->hi - $thisg->m->curg->stack->lo
  newSize = oldSize * 2
  printf("runtime: newstack: %d -> %d\n", oldSize, newSize)
  print_ubacktrace_fileline($thisg->m->morebuf->pc, 
                            $thisg->m->morebuf->sp, 
                            $thisg->m->morebuf->lr)
}

```

当我们运行这个脚本时，我们得到:

```
$ ls
newstack.stp  example  example.go
$ stap newstack.stp
                        <== execute "./example" in another terminal
    :
runtime: newstack: 2048 -> 4096
 0x40d748 : runtime.mallocgc+0x548/0x9d0 at /usr/lib/golang/src/runtime/malloc.go:740 [/home/juddin/example]
 0x40dd48 : runtime.newobject+0x38/0x60 at /usr/lib/golang/src/runtime/malloc.go:839 [/home/juddin/example]
    :
 0x47c4cb : fmt.Fprintln+0x8b/0x100 at /usr/lib/golang/src/fmt/print.go:255 [/home/juddin/example]
 0x47c597 : fmt.Println+0x57/0x90 at /usr/lib/golang/src/fmt/print.go:264 [/home/juddin/example]
 0x4827a4 : main.say+0xa4/0xd0 at /home/juddin/example.go:9 [/home/juddin/example]
 0x44ecb1 : runtime.goexit+0x1/0x10 at /usr/lib/golang/src/runtime/asm_amd64.s:2362 [/home/juddin/example]

```

为了使事情更清楚，我省略了一些输出，并把重点放在感兴趣的地方。从回溯可以看出，堆栈增长是从`main.say()`开始触发的。因此，我们能够确定我们的代码确实导致了堆栈的增长，而且我们这样做并不需要修改目标程序，更不用说 go 运行时或编译器了。

*Last updated: July 23, 2019*