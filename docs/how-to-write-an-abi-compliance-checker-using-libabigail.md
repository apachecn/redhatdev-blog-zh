# 如何使用 Libabigail 编写 ABI 合规检查器

> 原文：<https://developers.redhat.com/blog/2020/04/02/how-to-write-an-abi-compliance-checker-using-libabigail>

我以前写过关于[确保由本地共享库](https://developers.redhat.com/blog/2014/10/23/comparing-abis-for-compatibility-with-libabigail-part-1)公开的应用程序二进制接口(ABI)的向前兼容性的挑战。本文介绍等式的另一面:如何为上游项目验证 ABI 向后兼容性。

如果您已经阅读了我的上一篇文章，那么您已经了解了 [Libabigail](https://sourceware.org/libabigail) ，这是一个静态代码分析和工具库，用于构造、操作、序列化和反序列化与 ABI 相关的工件。

在本文中，我将向您展示如何构建一个基于 Python 的检查器，它使用 Libabigail 来验证 ABIs 在一个[共享库](https://en.wikipedia.org/wiki/Library_%28computing%29#Shared_libraries)中的向后兼容性。在这种情况下，我们将把重点放在基于 Linux 的操作系统上运行的可执行和可链接格式(ELF)二进制格式的共享库的 ABI 上。

**注意**:本教程假设您已经在开发环境中安装并设置了 Libabigail 及其相关的命令行工具`abidw`和`abidiff`。参见 [Libabigail](https://sourceware.org/libabigail) 文档，获取并安装 Libabigail 的指南。

## 确保向后兼容性

如果我们声明共享库的新版本的 ABI 是*向后兼容的*，我们向用户保证库的新版本中的 ABI 变化不会影响与旧版本链接的应用程序。这意味着应用程序的功能不会以任何方式改变或中断，即使对于那些没有重新编译应用程序就更新到库的新版本的用户也是如此。

为了自信地做出这样的声明，我们需要一种方法来比较新的库版本和旧的库版本的 ABI。知道了 ABI 的变化是什么，我们就能够确定任何变化是否有可能破坏向后兼容性。

## 示例项目:libslicksoft.so

为了这篇文章，让我们假设我是一个名为 SlickSoftware 的自由软件项目的发布经理。我已经说服你(我的黑客伙伴)我们库的 ABI，`libslicksoft.so`，应该向后兼容旧版本，至少现在是这样。为了确保向后兼容性，我们将编写一个 ABI 检查程序，可以在开发周期的任何时候运行。检查器将帮助我们确保当前版本的 ABI 与先前版本的 ABI，基线 ABI 保持兼容。一旦我们编写了检查器，我们也将能够在未来的项目中使用它。

下面是`slick-software/lib`目录的布局，其中包含 SlickSoftware 的源代码:

```
+ slick-software/
|
+ lib/
|    |
|    + file1.c
|    |
|    + Makefile
|
+ include/
|        |
|        + public-header.h
|
+ abi-ref/

```

让我们从设置示例项目开始。

### 步骤 1:创建共享库

为了创建一个共享库，我们访问`slick-software/lib`目录并输入`make`。我们将称这个新的共享库为`slick-software/lib/libslicksoft.so`。

### 步骤 2:创建参考 ABI 的表示

我们的下一步是为我们的共享库`slick-software/lib/libslicksoft.so`创建一个 ABI 的表示。一旦我们完成了这些，我们将把它保存在`slick-software/abi-ref/`目录中，这个目录目前是空的。

ABI 代表处将作为*ABI*的参考。我们将把`libslicksoft.so`的所有后续版本的 ABI 与它进行比较。理论上，我们可以只保存一个`libslicksoft.so`的副本，并使用二进制文件本身进行 ABI 比较。我们选择不这样做，因为像许多开发人员一样，我们不喜欢在版本控制软件中存储二进制文件。幸运的是，Libabigail 允许我们保存 ABI 的文本表示。

#### 创建 ABI 代表

要生成 ELF 二进制 ABI 的文本表示，我们所要做的就是打开您最喜欢的命令行解释器并输入以下内容:

```
$ abidw slick-software/lib/libslicksoft.so > slick-software/abi-ref/libslicksoft.so.abi

```

#### 自动化创建过程

我们可以通过在`slick-software/lib/Makefile`的末尾添加一个规则来自动化这个过程。将来，每当我们想要生成 ABI `libslicksoft.so.abi`文件的文本表示时，我们只需键入`make abi-ref`。

下面是那个`Makefile`的内容:

```
$cat slick-software/lib/Makefile SRCS:=file1.c
HEADER_FILE:=../include/public-header.h
SHARED_LIB:=libslicksoft.so
SHARED_LIB_SONAME=libslicksoft
ABI_REF_DIR=../abi-ref
ABI_REF=$(ABI_REF_DIR)/$(SHARED_LIB).abi
CFLAGS:=-Wall -g -I../include
LDFLAGS:=-shared -Wl,-soname=$(SHARED_LIB_SONAME)
ABIDW:= /usr/bin/abidw
ABIDIFF= /usr/bin/abidiff

OBJS:=$(subst .c,.o,$(SRCS))

all: $(SHARED_LIB)

%.o:%.c $(HEADER_FILE)
        $(CC) -c $(CFLAGS) -o $@ $<

$(SHARED_LIB): $(OBJS)
        $(CC) $(LDFLAGS) -o $@ $<

clean:
        rm -f *.o $(SHARED_LIB) $(ABI_REF)

abi-ref: $(SHARED_LIB)
        $(ABIDW) $< > $(ABI_REF)

```

### 第三步:比较 ABI 的变化

现在我们有了一个参考 ABI，我们只需要比较新版本的`libslicksoft.so`并分析变化。我们可以使用 Libabigail 的 [abidiff](https://sourceware.org/libabigail/manual/abidiff.html) 程序来比较两个库版本。下面是调用`abidiff`的命令:

```
abidiff baseline.abi path/to/new-binary

```

该命令行将`new-binary`的 ABI 与`baseline.abi`进行比较。它生成一个关于潜在 ABI 更改的报告，然后返回一个状态代码，告诉我们检测到的不同种类的 ABI 更改。通过分析用位图表示的状态代码，我们将能够判断 ABI 的任何更改是否有可能破坏向后兼容性。

## 基于 Python 的 ABI 差异检查器

我们的下一个任务是编写一个调用`abidiff`来执行 ABI 检查的程序。我们将其命名为`check-abi`，并将其放在新的`slick-software/tools`目录中。

有人告诉我 Python 很酷，所以我想用这个新的 checker 来尝试一下。我远不是一个 Python 专家，但是，嘿，有什么会出错呢？

### 步骤 1:规范 ABI 检查器

首先，让我们浏览一下我们想要编写的基于 Python 的 ABI 检查器。我们将这样运行它:

```
$ check-abi baseline.abi slicksoft.so

```

检查器应该很简单。如果没有 ABI 问题，它将以零(0)状态代码退出。如果发现向后兼容问题，它将返回一个非零状态代码和一条有用的消息。

### 步骤 2:导入依赖关系

我们正在用 [Python 3](https://www.python.org/downloads/) 编写脚本形式的`check-abi`程序。我们要做的第一件事是导入该程序所需的包:

```
#!/usr/bin/env python3

import argparse
import subprocess
import sys

```

### 步骤 3:定义一个解析器

接下来，我们需要一个解析命令行参数的函数。让我们先定义它，现在不要过多考虑它的内容:

```
def parse_command_line():
    """Parse the command line arguments.

       check-abi expects the path to the new binary and a path to the
       baseline ABI to compare against.  It can also optionaly take
       the path to the abidiff program to use.
    """
# ...

```

### 步骤 4:编写主函数

在这种情况下，我已经写好了主函数，让我们来看看:

```
def main():
    # Get the configuration of this program from the command line
    # arguments. The configuration ends up being a variable named
    # config, which has three properties:
    #
    #   config.abidiff: this is the path to the abidiff program
    #
    #   config.baseline_abi: this is the path to the baseline
    #                        ABI. It's the reference ABI that was
    #                        previously stored and that we need to
    #                        compare the ABI of the new binary
    #                        against.
    #
    #   config.new_abi: this is the path to the new binary which ABI
    #                   is to be compared against the baseline
    #                   referred to by config.baseline_abi.
    #
    config = parse_command_line()

    # Execute the abidiff program to compare the new ABI against the
    # baseline.
    completed_process = subprocess.run([config.abidiff,
                                        "--no-added-syms",
                                        config.baseline_abi,
                                        config.new_abi],
                                       universal_newlines = True,
                                       stdout = subprocess.PIPE,
                                       stderr = subprocess.STDOUT)

    if completed_process.returncode != 0:
        # Let's define the values of the bits of the "return code"
        # returned by abidiff.  Depending on which bit is set, we know
        # what happened in terms of ABI verification.  These bits are
        # documented at
        # https://sourceware.org/libabigail/manual/abidiff.html#return-values.
        ABIDIFF_ERROR_BIT = 1
        ABI_CHANGE_BIT = 4
        ABI_INCOMPATIBLE_CHANGE_BIT = 8

        if completed_process.returncode & ABIDIFF_ERROR_BIT:
            print("An unexpected error happened while running abidiff:n")
            return 0
        elif completed_process.returncode & ABI_INCOMPATIBLE_CHANGE_BIT:
            # If this bit is set, it means we detected an ABI change
            # that breaks backwards ABI compatibility, for sure.
            print("An incompatible ABI change was detected:n")
        elif completed_process.returncode & ABI_CHANGE_BIT:
            # If this bit is set, (and ABI_INCOMPATIBLE_CHANGE_BIT is
            # not set) then it means there was an ABI change that
            # COULD potentially break ABI backward compatibility.  To
            # be sure if this change is problematic or not, a human
            # review is necessary
            print("An ABI change that needs human review was detected:n")

        print("%s" % completed_process.stdout)
        return completed_process.returncode

    return 0;

```

#### 关于代码的注释

代码被大量注释，以使未来的程序员更容易理解。这里有两个重要的亮点。首先，注意`check-abi`是如何用`--no-added-syms`选项调用`abidiff`的。该选项告诉`abidiff`添加的函数、全局变量和公共定义的 ELF 符号(*又名*添加的 ABI 工件)应该**而不是**被报告。这让我们将注意力集中在已经被改变或移除的 ABI 工件上。

其次，注意我们如何设置检查器来分析由`abidiff`生成的返回代码。您可以在从这里开始的`if`语句中看到这个细节:

```
if completed_process.returncode != 0:

```

如果该返回代码的第一位被置位(位值 1)，那么这意味着`abidiff`在执行时遇到了管道错误。在这种情况下，`check-abi`会打印一条错误消息，但不会报告 ABI 问题。

如果设置了返回代码的第四位(位值 8)，那么这意味着 ABI 的改变破坏了与旧库版本的向后兼容性。在这种情况下，`check-abi`将打印一条有意义的消息和一份详细的变更报告。回想一下，在这种情况下，检查器会产生一个非零的返回代码。

如果只设置了返回代码的第三位(位值 4)，而上面提到的第四位没有设置，那么这意味着`abidiff`检测到了可能*潜在地*破坏向后兼容性的 ABI 变化。在这种情况下，对变更进行人工审查是必要的。检查者将打印一条有意义的消息和一份详细的报告，供他人检查。

**注**:如果你有兴趣，可以在这里找到`abidiff` [生成的返回码的完整细节。](https://sourceware.org/libabigail/manual/abidiff.html#return-values)

## check-abi 程序的源代码

下面是`check-abi`程序的完整源代码:

```
#!/usr/bin/env python3

import argparse
import subprocess
import sys

def parse_command_line():
    """Parse the command line arguments.

       check-abi expects the path to the new binary and a path to the
       baseline ABI to compare against.  It can also optionaly take
       the path to the abidiff program to use.
    """

    parser = argparse.ArgumentParser(description="Compare the ABI of a binary "
                                                 "against a baseline")
    parser.add_argument("baseline_abi",
                        help = "the path to a baseline ABI to compare against")
    parser.add_argument("new_abi",
                        help = "the path to the ABI to compare "
                               "against the baseline")
    parser.add_argument("-a",
                        "--abidiff",
                        required = False,
                        default="/home/dodji/git/libabigail/master/build/tools/abidiff")

    return parser.parse_args()

def main():
    # Get the configuration of this program from the command line
    # arguments. The configuration ends up being a variable named
    # config, which has three properties:
    #
    #   config.abidiff: this is the path to the abidiff program
    #
    #   config.baseline_abi: this is the path to the baseline
    #                        ABI. It's the reference ABI that was
    #                        previously stored and that we need to
    #                        compare the ABI of the new binary
    #                        against.
    #
    #   config.new_abi: this is the path to the new binary which ABI
    #                   is to be compared against the baseline
    #                   referred to by config.baseline_abi.
    #
    config = parse_command_line()

    # Execute the abidiff program to compare the new ABI against the
    # baseline.
    completed_process = subprocess.run([config.abidiff,
                                        "--no-added-syms",
                                        config.baseline_abi,
                                        config.new_abi],
                                       universal_newlines = True,
                                       stdout = subprocess.PIPE,
                                       stderr = subprocess.STDOUT)

    if completed_process.returncode != 0:
        # Let's define the values of the bits of the "return code"
        # returned by abidiff.  Depending on which bit is set, we know
        # what happened in terms of ABI verification.  These bits are
        # documented at
        # https://sourceware.org/libabigail/manual/abidiff.html#return-values.
        ABIDIFF_ERROR_BIT = 1
        ABI_CHANGE_BIT = 4
        ABI_INCOMPATIBLE_CHANGE_BIT = 8

        if completed_process.returncode & ABIDIFF_ERROR_BIT:
            print("An unexpected error happened while running abidiff:n")
            return 0
        elif completed_process.returncode & ABI_INCOMPATIBLE_CHANGE_BIT:
            # If this bit is set, it means we detected an ABI change
            # that breaks backwards ABI compatibility, for sure.
            print("An incompatible ABI change was detected:n")
        elif completed_process.returncode & ABI_CHANGE_BIT:
            # If this bit is set, (and ABI_INCOMPATIBLE_CHANGE_BIT is
            # not set) then it means there was an ABI change that
            # COULD potentially break ABI backward compatibility.  To
            # be sure if this change is problematic or not, a human
            # review is necessary
            print("An ABI change that needs human review was detected:n")

        print("%s" % completed_process.stdout)
        return completed_process.returncode

    return 0;

if __name__ == "__main__":
    sys.exit(main())

```

## 使用 Makefile 中的 check-abi

我们已经完成了基本的 checker，但是我们可以添加一两个特性。例如，如果我们可以从`slick-software/lib`目录中调用我们闪亮的新`check-abi`程序不是很好吗？然后，我们可以在任何需要进行 ABI 验证的时候输入一个简单的`make`命令。

我们可以通过在`slick-software/lib/Makefile`的末尾添加一条规则来设置这个特性:

```
abi-check: $(SHARED_LIB)
        $(CHECK_ABI) $(ABI_REF) $(SHARED_LIB) || echo "ABI compatibility issue detected!"

```

当然，我们还需要在 Makefile 的开头定义变量`CHECK_ABI`:

```
CHECK_ABI=../tools/check-abi

```

以下是包含这些更改的完整 Makefile:

```
SRCS:=file1.c
HEADER_FILE:=../include/public-header.h
SHARED_LIB:=libslicksoft.so
SHARED_LIB_SONAME=libslicksoft
ABI_REF_DIR=../abi-ref
ABI_REF=$(ABI_REF_DIR)/$(SHARED_LIB).abi
CFLAGS:=-Wall -g -I../include
LDFLAGS:=-shared -Wl,-soname=$(SHARED_LIB_SONAME)
ABIDW:=/usr/bin/abidw
ABIDIFF=/usr/bin/abidiff
CHECK_ABI=../tools/check-abi

OBJS:=$(subst .c,.o,$(SRCS))

all: $(SHARED_LIB)

%.o:%.c $(HEADER_FILE)
        $(CC) -c $(CFLAGS) -o $@ $<

$(SHARED_LIB): $(OBJS)
        $(CC) $(LDFLAGS) -o $@ $<

clean:
        rm -f *.o $(SHARED_LIB) $(ABI_REF)

abi-ref: $(SHARED_LIB)
        $(ABIDW) $< > $(ABI_REF)

abi-check: $(SHARED_LIB)
        $(CHECK_ABI) $(ABI_REF) $(SHARED_LIB) || echo "ABI compatibility issue detected!"

```

## 运行检查器

我们差不多完成了，但是让我们用一个简单的 ABI 检查来测试我们的新检查器的向后兼容性。首先，我将对`slick-software`库做一些修改，这样我就可以检查差异了。

接下来，我访问`slick-software/lib`目录并运行`make abi-check`。这是我得到的回报:

```
$ make abi-check
../tools/check-abi ../abi-ref/libslicksoft.so.abi libslicksoft.so || echo "ABI compatibility issue detected!"
An incompatible ABI change was detected:

Functions changes summary: 1 Removed, 0 Changed, 0 Added function
Variables changes summary: 0 Removed, 0 Changed, 0 Added variable

1 Removed function:

  'function void function_1()'    {function_1}

ABI compatibility issue detected!
$

```

ABI 检查器报告了一个兼容性问题，删除了一个函数。我想我应该把`function_1()`放回去，以免打破 ABI。

## 结论

在本文中，我向您展示了如何为上游项目中的共享库编写一个基本的 ABI 验证器。为了使这个项目简单，我省略了您可能想要自己添加到 checker 的其他特性。例如，Libabigail 有处理误报的机制，这在现实世界的项目中很常见。此外，我们还在不断改进该工具，以提高其分析质量。如果关于 Libabigail 的任何事情不尽如人意，请在[Libabigail 邮件列表](https://sourceware.org/libabigail/)上告诉我们。

祝你黑客生涯愉快，愿你所有的 ABI 不兼容问题都能被发现。

*Last updated: June 29, 2020*