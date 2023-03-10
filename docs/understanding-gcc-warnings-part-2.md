# 理解 GCC 警告，第 2 部分

> 原文：<https://developers.redhat.com/blog/2019/03/13/understanding-gcc-warnings-part-2>

在[第 1 部分](https://developers.redhat.com/blog/2019/03/13/understanding-gcc-warnings/)中，我阐明了 GCC 实现选择各种类型的前端警告时所涉及的权衡，比如预处理警告、词法警告、类型安全警告和其他警告。

尽管前端警告很有用，但是那些基于程序中控制流或数据流的警告却有相当大的局限性。为了克服它们，基于流量的警告越来越多地在 GCC 所谓的“中端”实施。中端警告是本文的重点。

## 中端警告

出于我们的目的，中端从*Gimplifier*(GCC 的一部分，将 [GENERIC](https://gcc.gnu.org/onlinedocs/gccint/GENERIC.html) 转换为 [GIMPLE](https://gcc.gnu.org/onlinedocs/gccint/GIMPLE.html) )开始，包括所有后续阶段，直到 GIMPLE 扩展到 RTL(低级寄存器传输语言)。相当多的 GCC 警告符合这种描述。代表性抽样包括

*   `-Warray-bounds`，检测越界数组索引
*   `-Wformat-overflow`(GCC 7 中的新特性)，它在调用`sprintf`和相关函数时寻找缓冲区溢出
*   `-Wnonnull`，它检测将空指针传递给不需要它的函数
*   `-Wstringop-overflow`(GCC 7 中的新特性)，它有助于发现字符串和原始内存函数调用中的缓冲区溢出
*   `-Wuninitialized`，寻找未初始化变量的用法

随着 GCC 试图检测越来越难的 bug，中端警告的数量一直在增长。

GCC 中的中端被设计为一系列超过一百个或多或少独立的模块，称为*通道*。每一次遍历都以预定的顺序遍历源代码的中间表示(最初是泛型，然后是 GIMPLE ),并将语句序列转换成其他语句序列，在此过程中执行各种简化。在这个过程中，一些传递暴露了程序操作的数据的有趣属性，例如指针引用的对象的大小或整数变量在条件语句的不同分支中可以采用的值的范围，循环迭代次数的上限，等等。

虽然少数遍的目的是专门寻找 bug，但几乎所有遍的主要目的都是为了提高代码效率。因为这些优化通道仅在其优化开启时被选择，如果使用`-O0`禁用优化，警告的数量和质量将会降低。如果在过程中的某个地方，pass 发现了一个必然无效的构造，它可能会发出警告，但是除了少数例外，这样做是次要的。其结果是，在中端实施质量警告具有挑战性。

## 中端警告的局限性

在中端实现的所有警告的不可避免的挑战是所有基于流的警告所固有的:由于可计算性的限制，它们容易出现假阴性和假阳性。但是除此之外，GCC 中端警告也是不完美的，因为它们依赖于主要为优化而设计的基础设施和表示，并且具有与静态分析器不同的优先级。他们的首要目标是为遵循语言规则的程序发出高效的目标代码。GCC 优化者历来所做的一个基本假设是，他们使用的代码必须是有效的，这个假设在 C 和 C++以及其他语言的标准中都有体现。否则，所有的赌注都是关闭的，他们的行为是未定义的。基于这种观点的设计很难检测甚至是简单的错误并对它们进行诊断。同样的设计还会导致不经意地发出正确构造的警告。我们来看例子。

### 由于对程序有效性的假设而导致的假阴性

作为一个简单的例子，考虑下面这个非常无效的程序:

```
const int primes[10] = { 1, 3, 5, 7, 11, 13, 17, 19 };
int f (void)
{
  return primes[17];   // index out-of-bounds 
}
```

人们会认为编译它会触发一个读取`primes`数组的`-Warray-bounds`警告，但是 GCC 没有发出这样的警告。这似乎是一个简单的 bug(它可以在 GCC Bugzilla: [78678](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=78678) 和 [86691](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=86691) 中找到)，但事实证明，它实际上是内部设计的一个“特性”，基于对中间端的输入是有效程序的假设。在计算数组解引用的结果的函数中甚至有一个注释，使得这一点显式化(GCC 使用术语*构造函数*来表示聚合初始化器):

```
  /* Memory not explicitly mentioned in constructor is 0 (or
     the reference is out of range). */
  return type ? build_zero_cst (type) : NULL_TREE;
}
```

代码认为`primes`数组中任何没有被显式初始化的元素的值都是零，不管它是在边界内还是在边界外。因此，不会诊断静态常数的无效索引。

### 早期折叠的问题是

为了发出尽可能高效的代码，GCC 积极地执行转换，这些转换简化了程序的内部表示，并使得对。一个简单的例子是这样的分配

```
n = strlen ("hello");
```

其中 GCC 可以很容易地计算调用的结果，并将其折叠成一个常数，将赋值转换成`n = 5`。一个更复杂的例子是一个常量字符串的偏移量:

```
extern int i;
n = strlen ("hello" + i);
```

这里，调用的结果不能折叠成一个常数，除非`i`的值是已知的；然而，因为字符串的长度是已知的，并且因为结果指针有效的唯一值`i`在 0 和 5 之间，所以结果也必须在 0 和 5 之间。因此，作业可以折叠成`n = 5 - i`。事实上，这和发生的事情很接近:GCC 用表达式替换调用

```
n = (size_t)i < 5 ? 5 - (size_t)i : 0;
```

条件并不是绝对必要的，因为一个有效的程序必须保证`i`不超出字符串的界限，但是它确保了即使对于无效的值，结果也不会完全没有意义(或者过大)。用简单的条件替换`strlen`调用使得计算速度更快，代码更紧凑，因此这是一个有用的优化，但这是有代价的。因为 GCC 很早就执行了这种转换，如果`i`的值被一些后续的优化确定为在 0 到 5 的有效范围之外，那么检测到`strlen`参数中的指针加法无效就太晚了。该调用及其参数已经被删除，并被整数减法所取代。结果是无效代码没有被诊断出来。这可以在下面例子的第一个 GIMPLE 转储中看到。注意，GIMPLE 中没有出现`strlen`调用:

```
size_t f (int i)
{
  i = 17;
  return strlen ("hello" + i);
}
```

通过编译带有`-fdump-tree-gimple=/dev/stdout`选项的函数获得的转储如下所示:

```
f (int i)
{
  long unsigned int D.1909;
  long unsigned int iftmp.0;

  i = 17;
  if (i <= 5) goto <D.1911>; else goto <D.1912>;
  <D.1911>:
  _2 = (sizetype) i;
  iftmp.0 = 5 - _2;
  goto <D.1913>;
  <D.1912>:
  iftmp.0 = 0;
  <D.1913>:
  D.1909 = iftmp.0;
  return D.1909;
}
```

除了计算常量字符串长度之外，GCC 早期还执行了许多其他转换，包括用其他调用(通常是`memcpy`)替换对库函数(如`strcpy`)的调用，甚至是更低级的原语，如聚合赋值(在 GIMPLE 中，包括数组在内的所有聚合都可以通过使用各种形式的 [MEM_REF 表达式](https://gcc.gnu.org/onlinedocs/gccint/Storage-References.html)进行赋值)。它们中的大多数(但不是全部)使转换后的代码更容易进一步转换，或者开辟其他有趣的优化机会。不幸的是，大多数错误还会导致详细信息的丢失，尽管这些信息对于优化来说很少重要，但对于后续的 bug 检测来说却是必不可少的。

### 缺少优化意味着缺少警告

所有的中端警告都在某种程度上依赖于优化，这些优化暴露了关于程序或数据的信息，而这些信息并不能从源代码中立即显现出来。GCC 是一个非常好的优化编译器，但是它有不可避免的限制，所以当某个警告所依赖的优化没有被实现时，警告就不会被发出。

让我们以`-Wnonnull`警告为例来说明这一点。因为下面的`strchr`函数的参数是一个全局常数，所以 GCC 能够很早就对它求值，并用它的结果或 null 替换`strcpy`的第二个参数。当`-Wnonnull`检查器看到`strcpy`调用时，第二个参数被替换为空，并触发警告:

```
const char a[] = "123";

void f (char *d)
{
  char *p = strchr (a, '9');
  strcpy (d, p);
}

warning: argument 2 null where non-null expected [**-Wnonnull**]
6 | strcpy (d, p);
  | **^~~~~~~~~~~~~**
```

通过使用`-fdump-tree-optimized=/dev/stdout`选项编译函数，可以看到优化的结果:

```
;; Function f (f, funcdef_no=0, decl_uid=1907, cgraph_uid=1, symbol_order=1)

f (char * d)
{
  <bb 2> [local count: 1073741824]:
  strcpy (d_2(D), 0B); [tail call]
  return;
}
```

但是由于常量数组被定义为局部变量，GCC 不再能够在编译时计算`strchr`调用并将其结果折叠到一个常量中。这是因为局部变量是动态初始化的，即使它们被声明为`const`。结果，没有额外智能的`-Wnonnull`警告不会触发。这仅仅是因为`strchr`优化不处理动态初始化的变量。

```
void g (char *d)
{
  const char b[] = "234";
  char *p = strchr (b, '9');
  strcpy (d, p);
}
```

实际上，在这种情况下，优化器的输出确认了`strchr`调用没有被折叠:

```
;; Function g (g, funcdef_no=1, decl_uid=1911, cgraph_uid=2, symbol_order=2)

g (char * d)
{
  char * p;
  const char b[4];

  <bb 2> [local count: 1073741824]:
  b = "234";
  p_3 = strchr (&b, 57);   // 57 == '9'
  strcpy (d_4(D), p_3);
  a ={v} {CLOBBER};
  return;
}
```

## 中端警告中的误报

当各种优化在程序的内部表示上传递时，它们执行各种转换，目的是简化它们或者为后面的传递提供简化机会。从假阳性警告的角度来看，虽然这些转换大多数是良性的，但有时它们会带来问题。问题的两个最大来源是通常所说的*早期折叠*和被称为*跳转线程*的优化。

### 跳线穿线

跳转线程传递试图通过合并现有的条件或者将语句移动甚至复制到条件块中来最小化代码中的条件分支。对于未经训练的人来说，这种传递似乎在程序中引入了源代码中不存在的执行路径。当常量或范围随后被传播到那些语句中时，条件块中的重复语句会带来麻烦。对跳转线程传递的完整解释超出了本文的范围。要了解更多背景知识，请阅读 Aldy Hernandez 的“[跳转线程优化简介](https://developers.redhat.com/blog/2019/03/13/intro-jump-threading-optimizations/)”。

作为跳转线程效果的一个例子，就拿下面这个简单的函数来说吧(减少自 GCC bug [88771](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=88771) )。在 ILP32 模式下用 GCC 8 编译它会导致下面两个可疑的警告。代码作者的自然反应是惊呼:“这些巨大的数字是从哪里来的？不是我写的！”

```
int f (char *d, const char *s, size_t i)
{
  size_t n = i + 1 ? i + 1 : i;

  strncpy (d, s, n);

  if (i + 1)
    return -1;
  return 0;
}

**warning:** ‘strncpy’ pointer overflow between offset 0 and size [4294967295, 2147483647] [**-Warray-bounds**]
   **strncpy (d, s, n);**
   **^~~~~~~~~~~~~~~~~**
**warning:** ‘strncpy’ specified size 4294967295 exceeds maximum object size 2147483647 [**-Wstringop-overflow=**]
```

通过使用`-fdump-tree-vrp`选项编译函数，可以从数值范围传播过程(VRP)的转储中了解触发警告的原因。对于非零的`n + 1`的测试已经导致通过，结合跳转线程优化，引入一个全新的基本块(`<bb 5>`)到带有明显无效的`strncpy`调用的函数中。在对转换后的代码的后续传递中运行的`-Warray-bounds`和`-Wstringop-overflow`警告会注意到第二个`strncpy`调用中不可思议的过大的大小，并假设它来自源代码，发出(有点令人困惑的)诊断。

```
f (char * d, const char * s, size_t i)
{
  int _2;
  size_t iftmp.0_4;

  <bb 2> [local count: 1073741825]:
  if (i_3(D) != 4294967295)    ;; 4294967295 == (size_t)-1
    goto <bb 3>; [66.00%]
  else
    goto <bb 5>; [34.00%]

  <bb 3> [local count: 708669604]:
  iftmp.0_4 = i_3(D) + 1;
  strncpy (d_6(D), s_7(D), iftmp.0_4);

  <bb 4> [local count: 1073741825]:
  # _2 = PHI <0(5), 1(3)>
  return _2;

  <bb 5> [local count: 365072224]:
  strncpy (d_6(D), s_7(D), 4294967295);
  goto <bb 4>; [100.00%]
}
```

跳转线程还会导致其他难以处理的警告，比如`-Wunintialized`和`-Wmaybe-uninitialized`。一个似乎特别容易受其影响的流行构造是 C++类模板`std::optional`。参见 GCC bug [80635](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=80635) 中的一个测试用例，以及对它所带来的挑战的详细讨论。

已经讨论了该问题的几种解决方案，并且针对一些警告采取了补救措施，但是没有一个是最佳的。下一节将讨论一种普遍提倡的方法。

### 与消毒剂的相互作用

中端警告的另一个意外来源可能是优化器与杀毒器(如地址杀毒器或未定义行为杀毒器)的交互。杀毒程序插入的工具有时会干扰优化程序跟踪控制或数据流的能力，或者限制优化程序确定诸如指针为空等属性的能力。因为警告和杀毒软件都服务于相同的目的，理想情况下，它们可以同时使用并具有相同的功效。不幸的是，尽管努力减少这些不良的相互作用，但似乎不可能完全避免。

## 中端警告挑战的解决方案

GCC 开发人员正在讨论许多解决方案，以提高中端警告的信噪比。实现额外的优化是使警告更有效的好方法。在许多情况下，这是一个双赢的主张，因为它也导致更有效的目标代码；然而，在某些情况下，特别是当实现优化的成本很高，而编码模式不够普遍时，投资回报就不那么明确了。

可能最困难的问题是避免由于早期折叠而导致的假阴性，因为在语句被折叠后保留原始细节需要对编译器中程序的表示进行架构上的改变。

避免由跳转线程引起的误报也是一个类似的问题，因为它们会导致 GCC 插入原始源代码中不存在的语句序列。如果跳转线程导致无效的语句，可以对其进行限制，但前提是在做出初始线程决策时可以确定其有效性。如果这些语句只是在附加的转换之后才被证明是无效的，那么要改变这个决定可能就太晚了。在这些情况下，可能需要采用另一种策略来避免警告。正在考虑的一种方法是用对`__builtin_trap()`或`__builtin_unreachable()`的调用来替换这样的语句，如果引入的路径没有在随后被消除，则可能结合在过程的很晚阶段发布诊断。适当策略的选择可能需要由命令行选项来控制，如视频“[C-GNU 工具大锅 2018](https://www.youtube.com/watch?v=inDduOFEyew) 中未定义行为的未来处理方向”中所讨论的

## 面向 C/C++开发人员的更多文章

*   [如何在红帽企业版 Linux 7 上安装 GCC 8 和 Clang/LLVM 6](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/)
*   [GCC 推荐的编译器和链接器标志](https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/)—用正确的标志改进警告和代码生成。
*   [GCC 8 中的可用性改进](https://developers.redhat.com/blog/2018/03/15/gcc-8-usability-improvements/)
*   [Clang/LLVM 入门](https://developers.redhat.com/blog/2017/11/01/getting-started-llvm-toolset/)
*   [用 GCC 8 检测字符串截断](https://developers.redhat.com/blog/2018/05/24/detecting-string-truncation-with-gcc-8/)
*   [GCC 7 的隐式失败检测](https://developers.redhat.com/blog/2017/03/10/wimplicit-fallthrough-in-gcc-7/)—检测 switch 块内缺失的 break 语句。通过`-Wimplicit-fallthrough`启用警告。如果您使用`-Wextra`，这也是将启用的警告之一。
*   [使用 GCC 7 进行内存错误检测](https://developers.redhat.com/blog/2017/02/22/memory-error-detection-using-gcc/)
*   [用 GCC 插件诊断函数指针安全缺陷](https://developers.redhat.com/blog/2017/03/17/diagnosing-function-pointer-security-flaws-with-a-gcc-plugin/)
*   [更好地使用 C11 原子——第一部分](https://developers.redhat.com/blog/2016/01/14/toward-a-better-use-of-c11-atomics-part-1/)