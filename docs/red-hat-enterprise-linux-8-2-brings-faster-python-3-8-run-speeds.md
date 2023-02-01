# 红帽企业版 Linux 8.2 带来更快的 Python 3.8 运行速度

> 原文：<https://developers.redhat.com/blog/2020/06/25/red-hat-enterprise-linux-8-2-brings-faster-python-3-8-run-speeds>

[红帽企业版 Linux (RHEL) 8](https://developers.redhat.com/rhel8/) 自带的 [Python](https://developers.redhat.com/blog/category/python/) 解释器是 3.6 版本，2016 年发布。虽然 Red Hat 承诺在 Red Hat Enterprise Linux 8 的整个生命周期中支持 Python 3.6 解释器，但对于某些用例来说，它已经有点过时了。

对于需要新的 Python 特性的开发人员，以及那些能够忍受不可避免的兼容性破坏的开发人员来说，Red Hat Enterprise Linux 8.2 也包含了 Python 3.8。除了提供新的特性，将 Python 3.8 与 RHEL 8.2 打包在一起允许我们比在坚如磐石的模块中更快地释放[性能](https://developers.redhat.com/blog/category/performance/)和打包改进。

本文主要关注`python38`包中的一个具体的性能改进。正如我们将要解释的，Python 3.8 是用 GNU 编译器集合(GCC)的`-fno-semantic-interposition`标志构建的。启用此标志会禁用语义插入，这可以将运行速度提高 30%。

**注意**:`python38`包加入了 RHEL 8.2 中发布的其他 Python 解释器，包括`python2`和`python3`包(我们在之前的文章中描述过，RHEL 8 中的 [Python)。您可以将 Python 3.8 与其他 Python 解释器一起安装，这样它就不会干扰现有的 Python 堆栈。](https://developers.redhat.com/blog/2018/11/14/python-in-rhel-8-3)

## 我以前在哪里见过这个？

写这篇文章感觉像是在抢别人的功劳。所以，让我们澄清一下:我们正在讨论的性能改进是其他人的成就。作为 RHEL 打包员，我们的角色类似于画廊馆长，而不是画家:我们的工作不是创建功能，而是从上游的 Python 项目中找出最好的，并在经过 Fedora 的审查、集成和测试后，将它们组合成让开发人员满意的体验。

请注意，我们在团队中确实有“画家”的角色。但是就像新鲜的油漆不属于展厅一样，原创作品首先进入更广泛的社区，只有在经过充分测试后才会出现在 RHEL(也就是说，有些无聊和明显)。

导致我们在本文中描述的变化的讨论包括由 Red Hat 的 Python 维护者提出的一个最初的天真建议、一个评论、由 C 专家 Jan Kratochvil 提出的一个更好的想法，以及对该想法的提炼。所有这一切都在 Fedora 开发邮件列表上公开进行，红帽公司和更广泛的社区都提供了意见。

## 在 Python 3.8 中禁用语义插入

正如我们提到的，我们的 RHEL 8.2 `python38`包中最显著的性能改进来自于启用 GCC 的`-fno-semantic-interposition`标志的构建。它将运行速度提高了 30%，而语义几乎没有变化。

这怎么可能呢？这里面有几个层次，让我们来解释一下。

### Python 的 C API

Python 的所有功能都暴露在它广泛的 C API 中。Python 的成功很大一部分来自于 C API，这使得*扩展*和*嵌入* Python 成为可能。*扩展*是用 C 之类的语言编写的模块，可以为 Python 程序提供功能。一个经典的例子是 [NumPy](https://numpy.org) ，这是一个用 C 和 Fortran 等语言编写的库，用于操作 Python 对象。*嵌入*意味着在更大的应用程序中使用 Python。像 [Blender](https://docs.blender.org/api/current/info_overview.html) 或 GIMP 这样的应用嵌入了 Python 来支持脚本。

Python(或者更准确地说， [CPython](https://github.com/python/cpython) ，Python 语言的参考实现)在内部使用 C API:每一次属性访问都要经过对 [`PyObject_GetAttr`函数](https://github.com/python/cpython/blob/v3.8.3/Objects/object.c#L837)的调用，每一次添加都是对 [`PyNumber_Add`](https://github.com/python/cpython/blob/v3.8.3/Objects/abstract.c#L956) 的调用，以此类推。

### Python 的动态库

Python 可以以两种模式构建:*静态*，其中所有代码都存在于 Python 可执行文件中，或者*共享*，其中 Python 可执行文件链接到其名为`libpython`的动态库。在 Red Hat Enterprise Linux 中，Python 是以共享模式构建的，因为嵌入 Python 的应用程序，如 Blender，使用的是`libpython`的 Python C API。

`python3.8`命令是嵌入的一个极简示例:它只调用`Py_BytesMain()`函数:

```
int
main(int argc, char **argv)
{
    return Py_BytesMain(argc, argv);
}

```

所有的代码都存在于`libpython`中。例如，在 RHEL 8.2 上，`/usr/bin/python3.8`的大小大约是 8 KiB，而`/usr/lib64/libpython3.8.so.1.0`库的大小大约是 3.6 兆字节。

### 语义插入

当执行一个程序时，动态加载器允许你覆盖将在程序中使用的动态库的任何符号(比如一个函数)。您可以通过设置环境变量`LD_PRELOAD`来实现覆盖。这种技术被称为 *ELF 符号插入*，在 GCC 中默认启用。

**注意**:在 Clang 中，语义插入是默认禁用的。

这个特性通常用于跟踪内存分配(通过覆盖 libc `malloc`和`free`函数)或者改变单个应用的时钟(通过覆盖 libc `time`函数)。语义插入是使用过程链接表(PLT)实现的。任何可以用`LD_PRELOAD`覆盖的函数在被调用之前都会在一个表中查找。

Python 从其他`libpython`函数中调用`libpython`函数。为了尊重语义插入，所有这些调用都必须在 PLT 中查找。虽然这个活动确实引入了一些开销，但是与被调用函数所花费的时间相比，速度的降低可以忽略不计。

**注意** : Python 使用 [`tracemalloc`模块](https://docs.python.org/dev/library/tracemalloc.html)跟踪内存分配。

### LTO 和函数内联

近年来，GCC 增强了链接时间优化(LTO ),以产生更高效的代码。一种常见的优化是*内联*函数调用，这意味着用函数代码的副本替换函数调用。一旦函数调用被内联，编译器可以在优化方面走得更远。

然而，不可能内联在 PLT 中查找的函数。如果函数可以用`LD_PRELOAD`完全换出，编译器就不能根据函数的功能进行假设和优化。

GCC 5.3 引入了`-fno-semantic-interposition`标志，它禁用语义插入。有了这个标志，`libpython`中调用其他`libpython`函数的函数就不必再通过 PLT 间接调用了。因此，它们可以通过 LTO 进行内联和优化。

所以，这就是我们所做的。我们在 Python 3.8 中启用了`-fno-semantic-interposition`标志。

## `-fno-semantic-interposition`的弊端

在启用了`-fno-semantic-interposition`的情况下构建 Python 的主要缺点是，我们不能再使用`LD_PRELOAD`覆盖`libpython`函数。不过影响仅限于`libpython`。例如，仍然有可能覆盖`libc`中的`malloc/free`来跟踪内存分配。

然而，这仍然是一个不兼容的问题:我们不知道开发者在 RHEL 8 上使用`LD_PRELOAD`和 Python 的方式是否会破坏`-fno-semantic-interposition`。这就是为什么我们只在新的 Python 3.8 中启用了这一变化，而 Python 3.6——默认的`python3`——继续像以前一样工作。

## 性能比较

为了在实践中看到`-fno-semantic-interposition`的优化，我们来看看`_Py_CheckFunctionResult()`函数。Python 使用这个函数来检查 C 函数是返回结果(不是`NULL`)还是引发异常。

下面是简化的 C 代码:

```
PyObject*
PyErr_Occurred(void)
{
    PyThreadState *tstate = _PyRuntime.gilstate.tstate_current;
    return tstate->curexc_type;
}

PyObject*
_Py_CheckFunctionResult(PyObject *callable, PyObject *result,
                        const char *where)
{
    int err_occurred = (PyErr_Occurred() != NULL);
    ...
}

```

#### 启用语义插入的汇编代码

我们先来看看红帽企业版 Linux 7 中的 Python 3.6，还没有用`-fno-semantic-interposition`构建。以下是汇编代码的摘录(由的`disassemble`命令读取):

```
Dump of assembler code for function _Py_CheckFunctionResult:
(...)
callq  0x7ffff7913d50 <PyErr_Occurred@plt>
(...)

```

如您所见，`_Py_CheckFunctionResult()`调用`PyErr_Occurred()`，并且该调用必须通过 PLT 间接方式。

#### 禁用语义插入的汇编代码

现在让我们来看一段禁用语义插入后的相同汇编代码:

```
Dump of assembler code for function _Py_CheckFunctionResult:
(...)
mov 0x40f7fe(%rip),%rcx # rcx = &_PyRuntime
mov 0x558(%rcx),%rsi    # rsi = tstate = _PyRuntime.gilstate.tstate_current
(...)
mov 0x58(%rsi),%rdi     # rdi = tstate->curexc_type
(...)

```

在这种情况下，GCC 内联了`PyErr_Occurred()`函数调用。结果，`_Py_CheckFunctionResult()`直接从`_PyRuntime`获取`tstate`，然后直接读取其成员`tstate->curexc_type`。没有函数调用和 PLT 间接调用，这导致了更快的性能。

**注意**:在更复杂的情况下，GCC 编译器可以根据调用它的上下文自由地对内联函数进行更多的优化。

## 你自己试试吧！

在本文中，我们将重点放在性能方面的一个具体改进上，将新特性留给上游文档[Python 3.7 中的新特性](https://docs.python.org/3.8/whatsnew/3.7.html)和[Python 3.8 中的新特性](https://docs.python.org/3.8/whatsnew/3.8.html)。如果您对 Python 3.8 中新的编译器性能可能性感兴趣，请从 Red Hat Enterprise Linux 8 资源库中获取`python38`包并尝试一下。我们希望你会喜欢跑步速度的提高，以及你自己会发现的许多其他新功能。

*Last updated: October 18, 2021*