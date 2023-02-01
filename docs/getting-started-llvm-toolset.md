# llvm 工具集入门

> 原文：<https://developers.redhat.com/blog/2017/11/01/getting-started-llvm-toolset>

llvm-toolset 是一个新的软件集合，它将 llvm 项目发布的许多工具打包在一起，包括:LLVM 工具和库、clang、clang-tools-extra 和 lldb。

## 安装 llvm 工具集

***更新安装说明参见[如何在红帽企业版 Linux 上安装 Clang/LLVM 6](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/)。***

Clang/LLVM 5.x 打包成`llvm-toolset-7`，可以在 RHEL 7 的`rhel-7-server-devtools-rpms`回购中获得。(如果你还没有 RHEL 7，红帽在这里提供免费的 [RHEL 订阅](https://developers.redhat.com/products/rhel/download/?intcmp=7016000000124eKAAQ)供开发使用。)可以使用 subscription manager 来启用 repo，然后像往常一样使用 yum 来安装它。

```
# subscription-manager repos --enable rhel-7-server-devtools-rpms
# yum install llvm-toolset-7
```

llvm-toolset-7 是一个元包，它将引入 llvm、clang、clang-tools-extra 和 lldb 包。

您可以通过检查各种工具的版本来验证软件包安装是否正确:

```
$ scl enable llvm-toolset-7 'clang -v'
Target: x86_64-unknown-linux-gnu
Thread model: posix
InstalledDir: /opt/rh/llvm-toolset-7/root/usr/bin
Found candidate GCC installation: /opt/rh/devtoolset-7/root/usr/lib/gcc/x86_64-redhat-linux/7
Found candidate GCC installation: /usr/lib/gcc/x86_64-redhat-linux/4.8.5
Selected GCC installation: /opt/rh/devtoolset-7/root/usr/lib/gcc/x86_64-redhat-linux/7
Candidate multilib: .;@m64
Candidate multilib: 32;@m32
Selected multilib: .;@m64
```

注意，clang 将使用随 devtoolset-7 发布的标准库和链接器。当您安装 llvm-toolset-7 时，这些依赖项会自动安装。

```
$ scl enable llvm-toolset-7 'lldb -v'
lldb version 4.0.1
```

对每个命令运行 scl 可能很麻烦，但是启用适当的路径来启动 shell 是很容易的:

```
$ scl enable llvm-toolset-7 bash
```

本文的其余部分假设了这样一个已启用的 shell。

## 尝试一下

您可以使用一个简单的 hello world 程序来试验 llvm-toolset 附带的一些工具。首先创建一个名为 hello.c 的简单 c 文件:

```
#include <stdio.h>
int main()
{
int ret;
printf("hello world!");
return ret;
}
```

clang-format 是一个重新格式化源代码的工具。它支持许多内置的编码风格，您可以定义自己的风格。使用 clang-format 更改编码风格(clang-format 默认为 LLVM 编码风格):

```
$ clang-format -i hello.c
```

```
#include <stdio.h>
int main() {
  int ret;
  printf("hello world!");
  return ret;
}
```

clang-tidy 是一个可以用来捕捉常见编程错误的工具。它支持许多不同种类的检查，并与 clang 静态分析器集成在一起。使用 clang-tidy 检查程序员错误:

```
$ clang-tidy -checks=all hello.c --

1 warning generated.
hello.c:5:3: warning: Undefined or garbage value returned to caller [clang-analyzer-core.uninitialized.UndefReturn]
return ret;
^
hello.c:3:3: note: 'ret' declared without an initial value
int ret;
^
hello.c:5:3: note: Undefined or garbage value returned to caller
return ret;
^
```

使用 clang 编译您的代码:

```
$ clang -g -o hello hello.c
```

使用 lldb 调试程序:

```
$ lldb ./hello
(lldb) target create "./hello"
Current executable set to './hello' (x86_64).
(lldb) b main
Breakpoint 1: where = hello`main + 25 at hello.c:4, address = 0x0000000000400529
(lldb) run
* thread #1, name = 'hello', stop reason = breakpoint 1.1
frame #0: hello`main at hello.c:4
  1 #include
  2 int main() {
  3 int ret;
> 4 printf("hello world!");
  5 return ret;
  6 }
```

## 进一步阅读

有关更多信息，请访问以下资源:

[llvm-toolset-7 在线文档](https://access.redhat.com/documentation/en-US/Red_Hat_Developer_Toolset/7/html/User_Guide/index.html)
[LLVM 官方在线文档](http://releases.llvm.org/4.0.1/docs/index.html)
[官方 clang 在线文档](http://releases.llvm.org/4.0.1/tools/clang/index.html)

*Last updated: March 28, 2019*