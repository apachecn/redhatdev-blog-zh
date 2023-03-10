# Annocheck:检查二进制文件的内容

> 原文：<https://developers.redhat.com/blog/2019/02/04/annocheck-examining-the-contents-of-binary-files>

GCC 的 [Annobin 插件在编译二进制文件时存储额外的信息。检查这些信息过去是由一组 shell 脚本执行的，但现在已经发生了变化，已经编写了一个新的程序—annocheck—来完成这项工作。该程序的优点是它比脚本更快、更灵活，并且它不依赖于其他实用程序来实际查看二进制文件内部。](https://developers.redhat.com/blog/2018/02/20/annobin-storing-information-binaries/)

这篇文章是关于 annocheck 程序的:如何使用它，如何工作，以及如何扩展它。该程序的主要目的是检查二进制文件是如何构建的，并检查它是否启用了所有适当的安全强化功能。但这不是它的唯一用途。它还有其他几种模式，可以对二进制文件进行不同种类的检查。

annocheck 的另一个特性是它被设计成易于扩展。它为剖析二进制文件提供了一个框架，并提供了一组实用程序来帮助进行这种检查。它还知道如何处理归档、rpm 和目录，将这些内容作为一系列普通文件呈现给每个工具。因此，工具只需要担心它们想要执行的特定任务。

## 使用 annocheck

Annocheck 作为 Fedora 随附的 [Annobin RPM 的一部分提供。它也可以直接从从](https://rpmfind.net/linux/rpm2html/search.php?query=annobin) [git 库](http://git%20clone%20git://sourceware.org/git/annobin.git)获得的源代码中构建。运行它的最简单方法是用要检查的文件名调用它。(注意:文件必须是 ELF 格式的文件。Annocheck 不处理其他二进制文件类型。)

```
annocheck <file>
```

该命令将在`<file>`运行默认工具——安全强化检查器。支持常用的命令行选项`--help`、`--version`和`--verbose`，以提供更多信息。Annocheck 还支持目录、归档和 rpm，因此以下所有功能也将起作用:

```
annocheck <directory>
```

```
annocheck lib<foo>.a
```

```
annocheck <rpm>
```

第一个版本会递归地对`<directory>`里面的所有 ELF 文件运行 annocheck。(其他类型的文件将被忽略。)第二个版本会对`lib<foo>.a`里面的所有 ELF 文件运行 annocheck。如果这是一个精简归档文件，将按照归档文件中的链接来查找真正的文件。

第三个版本会对`<rpm>`里面的所有 ELF 文件运行 annocheck。此版本还支持命令行选项，以提供与二进制 RPM 相关联的调试信息 RPM 的名称:

```
annocheck <rpm> --debug-rpm <debuginfo rpm>
```

提供调试信息 RPM 不是必需的，但是它非常有用，因为它通常包含使工具工作更容易的信息。

Annocheck 包含多个用于检查二进制文件的工具，尽管默认情况下仅启用安全强化检查器。其他工具可以通过特定的命令行选项来启用。目前，annocheck 支持的额外工具如下所述:

```
annocheck --enable-built-by
```

该工具报告用于构建二进制文件的编译器的名称。它会在二进制文件的不同位置查找，如果有`--all`命令行选项，它会报告找到的所有字符串，而不仅仅是第一个。

```
annocheck --enable-notes
```

这个工具显示 Annobin GCC 插件存储在二进制文件中的笔记。它类似于`readelf`程序的`--notes`选项，除了它根据地址范围对笔记进行排序，然后按顺序显示它们。

```
annocheck --section-size=<name>
```

此工具显示命名截面的大小。它类似于`readelf`程序的`--sections`选项，除了输出被限制在特定的部分，并在最后产生一个累积结果。因此，例如，它可以用来计算特定目录中所有二进制文件中所有`.text`部分的总大小。

每个工具都有自己的命令行选项来修改其行为。使用`--help`命令行选项查看这些选项的完整列表。可以同时启用多个工具，如果需要，可以通过以下选项禁用安全强化检查器:

```
annocheck --disable-hardened
```

## 安全强化检查器

这是 annocheck 框架中最大的，也可以说是最重要的工具。它对每个二进制文件运行一系列检查，以查看它是否是用适当的安全强化选项构建的。它通过 Annobin 插件生成的注释获得了许多关于这些选项的信息。

安全检查程序运行几项测试来确保程序链接正确:

*   程序在内存的可执行区域不能有堆栈。
*   程序中的任何段都不应该设置读、写和执行三个权限位。
*   不应该有针对可执行代码的重定位。
*   程序不能有任何保存在可写内存中的重定位。
*   用于在运行时定位共享库的运行路径信息必须只包括以`/usr` *开头的目录路径。*
*   动态可执行文件必须有一个动态段。
*   必须已经启用了`-pie`和`-z now`链接器选项。

安全检查器还对 Annobin 数据进行测试，以确保程序编译正确:

*   程序必须有注释，注释中不能有空白。
*   程序必须在启用了以下选项
    的情况下编译:
    *   `-D_FORTIFY_SOURCE=2`
    *   `-fpic`或`-fPIC`或`-fpie`或`-fPIE`
    *   `-fstack-protector-strong`或`-fstack-protector-all`
    *   `-O2`(或`-O3`或`-Og`)
*   使用异常处理的程序必须在启用了`-fexceptions`并指定了`-D_GLIBCXX_ASSERTIONS`的情况下编译。
*   如果编译器支持，并且适合特定的目标体系结构，还必须启用以下选项:
    *   `-fcf-protection=full`
    *   `-fstack-clash-protection`
    *   `-mcet`
    *   `-mstackrealign`

## 扩展 annocheck

annocheck 中的每个工具都由一个独立的源文件(或一组源文件)提供，该源文件被编译并链接到 annocheck 二进制文件中。只要在最终的 link 命令行中不包含这些工具，就可以省略它们。annocheck 框架没有任何工具的内置知识，而是通过每个工具为自己创建的全局结构与它们进行交互。
结构看起来像这样:

```
struct {
  const char * name;
  bool (* process_arg)
  void (* usage)
  void (* version)
  void (* start_scan)
  void (* end_scan)
  bool (* start_file)
  bool (* interesting_sec)
  bool (* check_sec)
  bool (* interesting_seg)
  bool (* check_seg)
  bool (* end_file)
}
```

该结构在`annocheck.h`头文件中定义。为了简单起见，这里省略了结构的全部细节。这些字段大部分是回调函数，除了第一个——`name`——它是工具的名称。当向用户报告特定于工具的信息时，会使用该名称。

回调函数是这个结构的主要部分。如果工具不需要该特定特征，则函数可以为空。`process_arg`、`usage`和`version`函数是内务处理工具的一部分，分别用于处理命令行参数、显示帮助信息和显示版本信息。

annocheck 启动时会调用`start_scan`函数(尽管是在命令行参数被处理之后)。然后，一旦所有的输入文件都处理完毕，就调用`end_scan`函数。然而，RPM 格式的一个怪癖意味着 annocheck 必须在 RPM 中递归调用自身。这在`start_scan`和`end_scan`函数中是允许的，它们有一个深度参数和一个文件名，可以用来在调用级别之间传递信息。

在定位到单个二进制文件之后，但在开始任何处理之前，调用`start_file`函数。它给工具一个机会来初始化任何每个文件的数据，并决定它是否有兴趣处理文件的内容。如果该工具对该文件不感兴趣，annocheck 将不会对它调用任何其他基于文件的函数。

一旦开始处理文件，就会为文件中的每个部分调用`interesting_sec`回调。这允许工具决定是否要更详细地检查该部分，如果是，将调用`check_sec`回调。检查以这种方式完成，这样，如果没有工具对某个特定部分感兴趣，它的内容就不会被加载到内存中，从而加快了执行速度。

一旦处理完这些部分——如果有的话——就使用`interesting_seg`和`check_seg`回调以类似的方式扫描这些部分。

一旦一个文件被完全扫描，调用`end_file`回调以允许工具执行任何最终处理并显示其结果。

除了回调结构，`annocheck.h`头还为 annocheck 本身导出的所有实用函数提供了原型。这包括探索调试信息和 ELF 注释、定位符号和打印消息的功能。

该头还定义了一个向 annocheck 框架注册工具的函数。这个函数的特殊之处在于，它应该从构造函数内部调用，而不是从普通代码中调用。这就是工具如何告诉 annocheck 它们的存在，并且它也允许在没有任何工具的特定知识的情况下构建 annocheck。构造函数应该如下所示:

```
  static bool disabled = false;

static __attribute__((constructor)) void
tool_register_checker (void) 
{
  if (! annocheck_add_checker (& tool_structure, major_version))
     disabled = true;
}
```

在这个函数中，`tool_structure`就是上面描述的回调结构。`major_version`变量由 annocheck 提供，它允许进行一致性检查，以确保 annocheck 和工具使用相同版本的回调结构。如果`annocheck_add_checker`函数返回 false，该工具将不会继续执行，因为注册过程中出现了问题。

## 未来的工作

annocheck 的开发是一个不断增加新功能的过程。目前，计划进行以下改进:

*   在 tar 文件中添加对二进制文件的支持，尤其是在压缩的 tar 文件中。可能还会增加对 zip 文件和其他类似归档系统的支持。
*   改进参数处理。目前，每个工具都定义了自己的命令行选项，这些选项可能会与其他工具冲突。相反，annocheck 框架应该执行一个策略，只为特定的工具提供特定的选项。
*   改进笔记工具，以显示笔记矩阵及其应用的内存区域。此外，添加对显示适用于特定区域的节和函数名称的支持。

*Last updated: February 3, 2019*