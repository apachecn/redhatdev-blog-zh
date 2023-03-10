# 扩展 gdbserver 以支持 strace 客户端

> 原文：<https://developers.redhat.com/blog/2020/03/16/extending-gdbserver-to-support-an-strace-client>

命令跟踪系统调用和信号，将它们和它们相应的参数决定为一种符号形式。开发人员经常提出的一个调试请求是允许`strace`跟踪一个程序的系统调用，这个程序也正在被 GDB 调试，就像这样:

```
% gdb --args test-program
(gdb) b main
Breakpoint 1 at 0x40128e: file test-program.c, line 22.
(gdb) run
Starting program: test-program
Breakpoint 1, main (argc=3, argv=0x7fffffffdb98) at test-program.c:22
22 int thread_count = 2;
(gdb)
```

在另一个终端窗口中，我们在 GDB 正在调试的同一个进程上调用`strace`:

```
% strace -p $(pgrep -f test-program)
strace: attach: ptrace(PTRACE_SEIZE, 27882): Operation not permitted
```

这里的罪魁祸首是`ptrace`系统调用，GDB 和`strace`都用它来控制程序的执行，不允许`strace`和 GDB 控制同一个进程。

一个解决方案是使用`gdbserver`来支持`gdb`客户端和`strace`客户端。

## 扩展`strace`以支持`gdbserver`

这个过程的第一步是扩展`strace`来支持`gdbserver`。使用`ptrace`调用拦截系统调用来实现`strace`命令。Gdbserver 还能够拦截系统调用。为了使`strace`能够使用`gdbserver`，需要添加一个新的后端，它使用`gdbserver`而不是`ptrace`来做系统调用拦截。此`gdbserver`后端提供与`ptrace`后端相似的功能。

现在，让我们看一些例子。

### 使用`gdbserver`的`strace`示例

使用另一个`gdbserver`后端运行`strace`的一个例子是:

```
% strace -G '|/usr/bin/gdbserver --once --multi stdio' test-program
```

这个例子告诉我们:

1.  使用`-G`选项连接到`gdbserver`。
2.  仅运行`gdbserver`一次。
3.  用 stdio 与 strace 通信。
4.  使用 gdbserver 追踪`test-program`。

运行时，输出提供(省略了一些输出):

```
...
[pid 15603] close(-1) = -1 EBADF (Bad file descriptor)
[pid 15603] chroot(".") = -1 EPERM (Operation not permitted)
[pid 15603] pipe([3, 4]) = 0
[pid 15603] write(4, "a\0", 2) = 2
[pid 15603] read(3, "a\0", 2) = 2
[pid 15603] madvise(0x7ffff75bb000, 8368128, MADV_DONTNEED) = 0
[pid 15595] --- stopped by 255 ---
[pid 15595] +++ exited with 0 +++
```

### 使用独立`gdbserver`的`strace`示例

或者`strace`可以使用独立的`gdbserver`:

1.  开始`gdbserver` :

    ```
    % gdbserver --multi :65432
    ```

2.  其次，启动`strace`客户端，使用 TCP 端口 65432 与`gdbserver`通信。
3.  告诉`strace`追踪测试程序:
    T1

### 使用遥控器`gdbserver`的`strace`示例

在远程机器上用独立的`gdbserver`运行`strace`的一个例子是首先在远程机器上启动`gdbserver`:

```
% gdbserver --once :65432 test-program
```

其次，启动`strace`客户端，并使用`gdbserver`命令中给出的 TCP 端口与远程主机上的`gdbserver`通信:

```
% strace -G remote-host.org.com:65432
```

注意，我们既没有给出`-p PID`选项，也没有给出测试程序选项。该信息由`gdbserver`指定，并且`strace`将继承该测试程序连接:

```
...
brk(NULL) = 0x63a000
arch_prctl(0x3001 /* ARCH_??? */, 0x7fffffffdb40) = -1 EINVAL (Invalid argument)
--- stopped by 255 ---
+++ exited with 0 +++
```

## 扩展`gdbserver`以支持`gdb`和`strace`客户端

Gdbserver 当前处理单个客户端连接。为了同时支持`gdb`和`strace`客户端，有必要启用`gdbserver`来处理多个客户端连接。为此，`gdbserver`必须交替进行:

1.  客户端等待。
2.  Gdbserver 向`strace`发送一个`syscall`包。
3.  Strace 通过`gdbserver`继续运行。
4.  当`strace`客户端等待时，Gdbserver 与`gdb`客户端交互。

Gdbserver 目前保存单个客户端的状态信息。使`gdbserver`成为多客户机感知的需要添加`gdb`客户机和`strace`客户机的状态信息——例如，连接的文件描述符和关于通过连接发送的数据包的状态信息。

与分时的情况一样，一个客户端将始终是活动客户端。客户端数据包请求(继续、步进、获取内存等。)和当前客户端状态(活动或等待)决定客户端的下一个状态。例如:

1.  **状态:** GDB 客户端是活动的，`strace`客户端正在等待。
2.  **请求:** GDB 客户端通过系统调用发出下一个请求。
3.  **状态:** GDB 客户端正在等待，`strace`客户端处于活动状态。
4.  **请求:** Strace 接收 syscall 并通过`gdbserver`继续运行。
5.  **状态:** GDB 客户端是活动的，`strace`客户端正在等待。

### `gdbserver`和`strace`客户端的示例

这个例子说启动`gdbserver`并使用 TCP 端口 65432 与客户端通信:

1.  开始`gdbserver`:

```
% gdbserver --multi :65432
```

2.  为 GDB 执行以下操作:

    1.  在另一个终端窗口中启动`gdb`客户端。
    2.  告诉 GDB 使用 TCP 端口 65432 与`gdbserver`通信。
    3.  设置断点并运行测试程序:

```
% gdb
(gdb) file test-program
(gdb) target extended-remote localhost:65432
(gdb) set remote exec-file test-program
(gdb) b thread_worker
(gdb) run
(gdb) Thread 1 "test-program" hit Breakpoint 1, thread_worker () ...
52 pthread_barrier_wait (&barrier);
```

1.  在另一个终端窗口中启动`strace`客户端。
2.  对`strace`执行以下操作:

    1.  告诉`strace`使用 TCP 端口 65432 与`gdbserver`通信。
    2.  告诉 strace 跟踪由`gdb`客户端启动的测试程序。
        
    3.  strace 窗口中会显示以下内容:

        ```
        ...

        brk(0x426000) = 0x426000
        ```

5.  **【GDB 窗口】**告诉`gdb`客户端使用线程二，设置断点，测试程序前进:

```
(gdb) thread 2
[Switching to thread 2 (Thread 7464.7490)]
#0 thread_worker () at test-program.c:59
(gdb) b 52
(gdb) continue
59 close (-1);
(gdb) next
61 chroot (".");
(gdb) next
```

6.  **【跟踪窗口】**注意到`strace`已经跟踪到了`close`和`chroot`系统调用:

```
[pid 7490] close(-1) = -1 EBADF (Bad file descriptor)
[pid 7490] chroot(".") = -1 EPERM (Operation not permitted)
```

7.  **【GDB 窗口】**推进测试程序:

```
(gdb) next
63 pipe (fd);
(gdb) next
65 write (fd[1], buf1, sizeof (buf1));
(gdb) next
67 read (fd[0], buf2, sizeof (buf2));
(gdb) next
```

8.  **【跟踪窗口】**注意`strace`已经跟踪到`pipe`、`write`和`read`调用:

```
[pid 7490] pipe([3, 4]) = 0
[pid 7490] write(4, "a\0", 2) = 2
[pid 7490] read(3, "a\0", 2) = 2
```

9.  **【GDB 窗口】**继续测试程序直至完成:

```
(gdb) continue
[Inferior 1 (process 7464) exited normally]
(gdb) quit
Remote connection closed
```

10.  **【跟踪窗口】**注意，附加的进程已经退出:

```
[pid 7490] madvise(0x7ffff7475000, 8368128, MADV_DONTNEED) = 0
[pid 7464] --- stopped by 255 ---
[pid 7464] +++ exited with 0 +++
strace: Process 7490 detached
```

## 当前工具状态

GDB 远程协议版本`strace`和相应的`gdbserver`目前正在审核中。最初版本的`gdbserver`和`strace`工具可以在 [gdbserver_copr](https://copr.fedorainfracloud.org/coprs/scox/gdbserver/) 和 [strace_copr](https://copr.fedorainfracloud.org/coprs/scox/strace/) 进行实验。

## 摘要

扩展`strace`以额外支持`gdbserver`提供了运行`strace`的额外手段。例如，跟踪正在远程机器上运行的程序。此外，这个扩展的`strace`和相应的扩展的`gdbserver`一起，在`gdb`客户端调试同一个程序时，能够跟踪程序中的系统调用。

*Last updated: June 29, 2020*