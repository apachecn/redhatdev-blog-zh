# GCC 11 中的静态分析更新

> 原文：<https://developers.redhat.com/blog/2021/01/28/static-analysis-updates-in-gcc-11>

[![The GNU logo.](img/b487100b7730792898d4d142c7200ce7.png)](https://developers.redhat.com/blog/wp-content/uploads/2014/09/gnu-logo.png) 
我在红帽上工作 [GNU 编译器集合](https://gcc.gnu.org/) (GCC)。在 GCC 10 中，我添加了新的`-fanalyzer`选项，一个[静态分析通道](https://developers.redhat.com/blog/2020/03/26/static-analysis-in-gcc-10/)，用于在编译时而不是运行时识别各种问题。最初的实现是针对早期采用者的，他们发现了一些错误，包括一个安全漏洞: [CVE-2020-1967](https://www.theregister.com/2020/04/23/gcc_openssl_vulnerability/) 。发现这个问题的 Bernd Edlinger 不得不费力地通过许多伴随真正问题的假阳性。其他用户也设法让分析器在他们的代码上崩溃。

我已经重写了分析器，以便在下一个主要版本中解决这些问题。在本文中，我描述了我正在采取的步骤，以减少误报的数量，并使这个静态分析工具更加健壮。

## 跟踪程序状态

我一直试图修复`-fanalyzer`中的 bug，因为它们是通过 GCC 的 Bugzilla 实例报告的。GCC 10 中分析器的状态跟踪组件有许多崩溃错误。我修复的 bug 越多，出现的 bug 就越多，而发现的速度并没有明显下降。这让我想到我需要重写组件。

在最初的`-fanalyzer`实现中，我在跟踪程序状态方面至少犯了两个大错误。这就是我如何追踪象征性的价值和区域。GCC 10 实现试图为这些符号实体分配唯一的 ID，并对它们进行规范化，以便可以比较不同的状态(不同状态之间的等价实体应该具有相同的 ID)。不幸的是，总有一个规范化的问题。

在新的实现中，我将这些实体变成了单件。因此，一个唯一的对象现在表示在分析入口的函数调用中特定参数的(符号)初始值。对单例的更改去掉了大量繁琐的规范化代码，代之以使用简单的指针。实现更简单，更快，我已经能够修复所有的 crasher 错误。(我不太确定我在最初的方法中看到了什么好处，但事后想来，我想是 20/20。)

第二个大的变化是象征性的价值和地区代表了什么。之前，我表示了到符号值的映射，其中键是内存区域的符号访问路径。在新的实现中，我将状态表示为内存中位偏移簇的映射。这些有时是具体的(例如，在特定的位偏移处)，有时是符号的(例如，索引是符号的数组偏移)。这种方法在处理联合、指针别名等方面做得更好。此外，当我切换到新的实现时，许多复杂的错误“自我修复”,这让我确信我是在正确的轨道上。

## 内存泄漏检测和不确定性

我必须为新的实现完全重写内存泄漏检测。也就是说，旧的实现有许多误报，而新的实现似乎更不容易出现。

我遇到的另一个问题是非确定性的，在这种情况下，分析器的确切行为会随着调用的不同而不同。在不同的地方，实现将遍历值，并且由于哈希算法，迭代的顺序将隐含地依赖于精确的指针值。由于地址空间布局随机化，指针值可能不同，从而导致不同的结果。我现在已经在代码中修复了这样的逻辑，以确保分析器的行为在每次运行中都是可重复的。

## 四个新警告

`-fanalyzer`的 GCC 10 实现增加了 15 个警告:

*   与内存管理相关的警告:
    *   `-Wanalyzer-double-free`
    *   `-Wanalyzer-use-after-free`
    *   `-Wanalyzer-free-of-non-heap`
    *   `-Wanalyzer-malloc-leak`
*   与缺少错误检查或误用空指针相关的警告:
    *   `-Wanalyzer-possible-null-argument`
    *   `-Wanalyzer-possible-null-dereference`
    *   `-Wanalyzer-null-argument`
    *   `-Wanalyzer-null-dereference`
*   与`stdio`流相关的警告:
    *   `-Wanalyzer-double-fclose`
    *   `-Wanalyzer-file-leak`
*   与从堆栈帧返回后使用相关的警告:
    *   `-Wanalyzer-stale-setjmp-buffer`
    *   `-Wanalyzer-use-of-pointer-in-stale-stack-frame`
*   不安全呼叫警告:
    *   `-Wanalyzer-unsafe-call-within-signal-handler`
*   概念验证警告:
    *   `-Wanalyzer-tainted-array-index`
    *   `-Wanalyzer-exposure-through-output-file`

对于 GCC 11，我添加了四个新警告:

*   `-Wanalyzer-write-to-const`
*   `-Wanalyzer-write-to-string-literal`
*   `-Wanalyzer-shift-count-negative`
*   `-Wanalyzer-shift-count-overflow`

其中的每一个都对应于 C 和 C++前端中实现的预先存在的警告，但是带有一个“`-Wanalyzer`”前缀，而不是“`-W`”例如，`-Wanalyzer-write-to-const`对应于`-Wwrite-to-const`。值得注意的是，这两种实现略有不同:现有的警告只是遍历特定表达式的语法树，而分析器变体进行基于过程间路径的分析，寻找试图写入`const`全局变量的代码路径。

在讨论了是否针对此类警告重用现有的命令行选项之后，我选择创建新的选项来明确警告的实现方式是不同的。以`-Wanalyzer`为前缀的警告会发现更多的问题，但是它们在编译时会更加昂贵。(尽管你已经通过选择`-fanalyzer`付出了代价。)

## 进行中:用于标记 API 的属性

GCC 很早就有`__attribute__((malloc))`将 API 入口点标记为内存分配器。在以前的 GCC 版本中，这纯粹是对优化器的指针别名逻辑的暗示。该属性让优化器“知道”从函数返回的指针指向不同于其他被优化指针的内存。然后，优化器可以在通过返回的指针进行写操作之后，消除从没有被破坏的位置进行的读操作。

在 GCC 11 中，这个属性现在可以带一个额外的参数来标记应该对结果调用哪个释放器函数。我正致力于将`-fanalyzer`一般化，以警告带有该属性的 API 的不匹配、泄漏和双重释放。然而，到目前为止，如果没有许多附加属性，结果是否有用还不清楚。例如，我尝试使用以下属性来检测 Linux 驱动程序中的泄漏(CVE-2019-19078):

```
  extern struct urb *usb_alloc_urb(int iso_packets, gfp_t mem_flags);
  extern void usb_free_urb(struct urb *urb);

```

我添加了属性来将`fns`标记为分配/解除分配对，这里有一个`urb`在错误处理路径上的泄漏。不幸的是，其他各种函数都使用`struct urb *`，分析器保守地认为传递给它们的`urb`可能被释放，也可能不被释放。因此，它停止跟踪它们的状态，并且只在我禁用了大部分中间代码时才报告问题。除了在最简单的情况下，这个特性还需要额外的工作才能发挥作用。

## 进行中:HTML 输出

分析器发出的控制流路径可能非常冗长，所以我一直在试验其他形式的输出。我有一个 HTML 输出的实现，其中路径信息被写到一个单独的 HTML 文件中。这里有几个例子:

*   [双免 bug](https://dmalcolm.fedorapeople.org/gcc/2020-11-05/html-examples/test.c.path-1.html)
*   [信号处理器问题](https://dmalcolm.fedorapeople.org/gcc/2020-11-05/html-examples/signal-1.c.path-1.html)
*   [内存泄漏](https://dmalcolm.fedorapeople.org/gcc/2020-11-05/html-examples/setjmp-7.c.path-1.html)(由于`longjmp`过去了一个`free`)

HTML 路径输出显示堆栈帧和事件的运行，使用投影给出 3D 效果。这个想法是突出显示一堆帧，就好像它是一堆重叠的卡片。我还添加了 JavaScript 来使用`j`和`k`在控制流事件中前进和后退。

不幸的是，HTML 输出没有捕获警告本身，只捕获了路径。解决这个问题需要对 GCC 的诊断子系统进行深入的修改，这是我在开发周期的这一点上非常担心的。所以，我不确定我找到了启用 HTML 格式的最佳方式；似乎更好的方法是以某种方式将所有的诊断捕获为构建工件，而不仅仅是那些具有关联路径的诊断的路径。

## GCC 11 和 fanalyzer 的下一步是什么

我们正处于 GCC 11 开发的错误修复阶段，目标是在 2021 年春天发布。分析器仍然需要相当多的错误修正，我们正在努力扩大它的规模。我计划在新年的第一天把重点放在这上面。(顺便说一下，这些问题可能是相关的:bug 有时会导致循环处理出错。然后，分析器将试图有效地*展开循环*，这将导致达到安全极限和缓慢、不完整的分析。)

我还在 GCC 11 中只为 C 开发`-fanalyzer`。我增加了对 C++的`new`和`delete`的部分支持，但是缺少的特性太多了，还不值得在真正的 C++代码上使用。我计划在 GCC 11 中让分析器对 C 代码具有健壮性和可伸缩性，并将 C++支持推迟到 GCC 12。

GCC 11 将在 [Fedora 34](https://fedoraproject.org/wiki/Changes/GNUToolchain) 中，也应该在 2021 年春天发布。对于简单的代码示例，您可以在[godbolt.org](https://godbolt.org/)在线体验新的 GCC。选择您的 GCC“主干”,并将`-fanalyzer`添加到编译器选项中。玩得开心！

*Last updated: February 8, 2021*