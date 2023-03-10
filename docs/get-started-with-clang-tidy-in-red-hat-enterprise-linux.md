# Red Hat Enterprise Linux 中的 clang-tidy 入门

> 原文：<https://developers.redhat.com/blog/2021/04/06/get-started-with-clang-tidy-in-red-hat-enterprise-linux>

[Clang-tidy](https://clang.llvm.org/extra/clang-tidy/#clang-tidy) 是一个独立的 linter 工具，用于检查 [C 和 C++](/topics/c) 源代码文件。它提供了一组额外的编译器警告——称为*检查*——超出了 C 或 C++编译器通常包含的内容。Clang-tidy 附带了一个大的内置检查集和一个用于编写您自己的检查的框架。

Clang-tidy 使用与 [Clang C 语言编译器](https://clang.llvm.org/)相同的前端库。但是，因为它只接受源文件作为输入，所以无论使用什么编译器，您都可以对任何 C 或 C++代码库使用`clang-tidy`。

本文简要介绍了 clang-tidy 的代码分析，包括如何在一个简单的基于 C 的程序中检查违反规则的情况，以及如何将 clang-tidy 集成到您的构建系统中。

## 在 Red Hat Enterprise Linux 中使用 clang-tidy

在[Red Hat Enterprise Linux](/products/rhel/overview)(RHEL)中，`clang-tidy`被包含在 [LLVM 工具集](/blog/2017/11/01/getting-started-llvm-toolset/)中:

```
# RHEL7
$ yum install llvm-toolset-10.0-clang-tools-extra

# RHEL8
$ yum install clang-tools-extra

```

开始使用`clang-tidy`的最好方式是查看包含的检查列表，看看哪些可能对您的代码库有用。整洁的项目页面包括各种可用检查的摘要。您可以通过运行以下命令来查看可用的单个检查的列表:

```
$ clang-tidy -checks=* -list-checks

```

今天，我们将关注对 [SEI CERT 安全编码标准](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard)的检查，这些标准在`clang-tidy`中由前缀`cert-`表示。

**注**:SEI CERT 安全编码标准由软件工程协会(SEI)的计算机应急响应小组(CERT)维护。

## 用铿锵整齐的声音检查错误

以下示例程序违反了 CERT 安全编码标准规则中的两个规则，即 [ENV33-C](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152177) 和 [ERR34-C](https://wiki.sei.cmu.edu/confluence/display/c/ERR34-C.+Detect+errors+when+converting+a+string+to+a+number) :

```
#include <stdlib.h>

int string_to_int(const char *num) {
  return atoi(num);
}

void ls() {
  system("ls");
}

```

让我们看看在这段代码上运行`clang-tidy`会发生什么:

```
$ clang-tidy -checks=cert-* -warnings-as-errors=* cert-err.c

2 warnings generated.
cert-err.c:4:10: error: 'atoi' used to convert a string to an integer value, but function will not report conversion errors; consider using 'strtol' instead [cert-err34-c,-warnings-as-errors]
  return atoi(num);
         ^
cert-err.c:8:3: error: calling 'system' uses a command processor [cert-env33-c,-warnings-as-errors]
  system("ls");
  ^

```

`-checks=cert-*`选项告诉`clang-tidy`启用所有 CERT 安全编码标准检查，`-warnings-as-errors=*`选项告诉它将所有警告视为错误。`-warnings-as-errors`采用通配符参数，因此您可以选择将哪些警告升级为错误。例如，如果您想要启用所有检查，但仅在证书检查上生成错误，您可以这样做:

```
$ clang-tidy -checks=* -warnings-as-errors=cert-* cert-err.c

```

## 将 clang-tidy 集成到您的构建系统中

除了在源文件上手动运行`clang-tidy`之外，您还可以将该工具集成到您的构建系统中。构建集成使得自动化检查更加容易，并将它们包含在一个[持续集成(CI)系统](/topics/ci-cd)中。将`clang-tidy`集成到您的构建中的一个简单方法是使用`make`或类似的构建工具，将检查作为`check`目标的一部分来运行。

回头看看我们之前的例子，我们可以用一个`clang-tidy`集成构造一个简单的 makefile:

```
SOURCES=cert-err.c
OBJS=cert-err.o

all: $(OBJS)

%.o: %.c
        $(CC) -c -o $@ $< $(CPPFLAGS) $(CFLAGS)

check: $(SOURCES)
        clang-tidy $(CPPFLAGS) -checks=cert-* --warnings-as-errors=* $(SOURCES)

```

现在，我们可以使用`make check`运行我们的`clang-tidy`检查。

## 使用编译数据库

如果你正在使用一个 [CMake](https://cmake.org/) 基础构建系统，`clang-tidy`可以使用一个编译数据库。这样，您就不需要手动传递编译时使用的那个`CPPFLAGS`。要生成编译数据库，只需要在配置时将`-DCMAKE_EXPORT_COMPILE_COMMANDS=ON`选项传递给 CMake 即可。以下是使用编译数据库的 CMake 配置示例:

```
set(sources ${CMAKE_SOURCE_DIR}/cert-err.c)

add_library(cert-err ${sources})

add_custom_target(
    clang-tidy-check clang-tidy -p ${CMAKE_BINARY_DIR}/compile_commands.json -checks=cert* ${sources}
    DEPENDS ${sources})

add_custom_target(check DEPENDS clang-tidy-check)

```

当您使用 CMake 进行配置时，它将生成一个名为`compile_commands.json`的文件，`clang-tidy`使用该文件来确定要使用哪些编译器标志:

```
$ cmake . -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
$ make clang-tidy-check

```

## 结论

这是对`clang-tidy`的基本介绍，但是你可以用它做更多的事情。更多信息，请阅读上游[铿锵整齐的项目页面](https://clang.llvm.org/extra/clang-tidy/)。

*Last updated: October 7, 2022*