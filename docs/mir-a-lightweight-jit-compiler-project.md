# MIR:一个轻量级 JIT 编译器项目

> 原文：<https://developers.redhat.com/blog/2020/01/20/mir-a-lightweight-jit-compiler-project>

在过去的三年里，我一直在参与添加即时编译( [JIT](https://en.wikipedia.org/wiki/Just-in-time_compilation) )到 [CRuby](https://github.com/ruby/ruby) 。现在，CRuby 有了基于[方法的实时编译器(MJIT)](https://www.rubyguides.com/2018/11/ruby-mjit/) ，它提高了非输入/输出限制程序的性能。

实现 JIT 最流行的方法是使用 [LLVM](https://llvm.org) 或 [GCC](https://gcc.gnu.org/) JIT 接口，如 [ORC](https://llvm.org/docs/ORCv2.html) 或 [LibGCCJIT](https://gcc.gnu.org/onlinedocs/jit/) 。GCC 和 LLVM 开发人员花费了巨大的努力来可靠、有效地实现优化，并致力于许多目标。使用 LLVM 或 GCC 来实现 JIT，我们可以免费使用这些优化。在 Ruby 3.0 发布之前的短时间内，使用现有的编译器是为 CRuby 获得 JIT 的唯一方法，Ruby 3.0 的目标是将 CRuby 的性能提高三倍。

所以，CRuby MJIT 利用了 GCC 或 LLVM，但是这个 JIT 有什么独特之处呢？

MJIT 不使用现有的编译器 JIT 接口。而是用 C 作为接口语言，又不失编译速度。实际上，通过使用[预编译头文件](https://en.wikipedia.org/wiki/Precompiled_header)和内存文件系统，可以获得与现有 JIT 接口相同的编译速度。

选择 C 作为 JIT 接口语言极大地简化了 JIT 实现、维护和调试。这样做也使得 JIT 独立于特定的 C 编译器。预编译头是现代 C/C++编译器的一个非常标准的特性。

## 基于 GCC/LLVM 的 JIT 的缺点

关于基于 GCC/LLVM 的 JIT 的优点已经说得够多了。让我们谈谈缺点:

*   基于 GCC/LLVM 的 JIT 很大。
*   它们的编译速度可能很慢。
*   很难实现用不同编程语言编写的代码的组合优化。

关于最后一点，在 CRuby 的情况下，Ruby 和 C 必须一起优化，因为您可以使用不同 Ruby 方法的实现语言。让我们更详细地考虑一下这些缺点。

### GCC/LLVM 规模

首先，GCC 和 LLVM 相对于 CRuby 是大的。多大？如果我们问[slocount](https://dwheeler.com/sloccount)，GCC 和 LLVM 源大约是 CRuby 的三倍，如图 1 所示。CRuby 本身已经是一个大项目，它的源代码包含超过 150 万行代码:

[![](img/981f9d87bf9924359a48df9601b5cd42.png "sloc-wide")](/sites/default/files/blog/2019/12/sloc-wide.png)Figure 1: CRuby’s source code size compared to GCC's and LLVM's source code size.">

至于机器码，GCC 和 LLVM 二进制文件要大得多，要大 7 到 18 倍，如图 2 所示:

[![](img/0a602477f06e55c014f2915de51bc080.png "objsize-wide")](/sites/default/files/blog/2019/12/objsize-wide.png)Figure 2: CRuby’s machine code is significantly smaller than GCC's and LLVM's machine code.">

想象一下用 LLVM 为简单语言添加一个 JIT。您可以轻松地将其解释器二进制代码大小增加一百倍。使用当前 JIT 的 CRuby 比 JRuby 或 T2 Graal/truffruby 需要更少的内存。尽管如此，对于云、[物联网](https://en.wikipedia.org/wiki/Internet_of_things)或移动环境来说，大代码长度可能是一个严重的问题。

### GCC/LLVM 编译速度

第二，GCC/LLVM 编译速度慢。对于使用 GCC/LLVM 的方法编译来说，在现代 Intel CPU 上可能感觉 20 毫秒很短，但是对于功能较弱但广泛使用的 CPU 来说，这个值可能是半秒。例如，根据 [SPEC2000 176.gcc](https://www.spec.org/cpu2000/CINT2000/176.gcc/docs/176.gcc.html) 基准测试，[树莓 PI3 B3+](https://en.wikipedia.org/wiki/Raspberry_Pi) CPU 大约比英特尔 i7-9700K 慢 30 倍(得分 320 比 8520)。

在速度慢的机器上，我们更需要 JIT，但是对于这些机器来说，JIT 编译慢得令人无法忍受。即使在快速的机器上，基于 GCC/LLVM 的 JIT 在像 [MinGW](https://en.wikipedia.org/wiki/MinGW) 这样的环境中也会太慢。更快的 JIT 速度也可以通过积极的[自适应优化](https://en.wikipedia.org/wiki/Adaptive_optimization)和[函数内联](https://compileroptimizations.com/category/function_inlining.htm)来帮助实现理想的 JIT 性能。

更快的 JIT 编译会是什么样子？2017 年 LLVM 开发者大会上关于 [Java Falcon 编译器](https://youtu.be/Uqch1rjPls8)的主题演讲建议，对于基于 LLVM 的 JIT 编译器，每个方法大约 100 毫秒，对于更快的[第一层 JVM 编译器](https://docs.oracle.com/javase/7/docs/technotes/guides/vm/performance-enhancements-7.html#tieredcompilation)，大约 1 毫秒。在回答关于使用 LLVM 实现 Python JIT 的问题时，演讲者(Philip Reems)说，你首先需要一个第一层的编译器实现。对于用于 Ruby 的 MJIT，我们从相反的方向出发，首先实现第二层编译器。

那么，为什么 GCC/LLVM 编译速度慢呢？这里是 GCC-8 后端传递的完整列表，按照它们的执行顺序显示，如图 3 所示:

[![GCC-8 backend passes presented in their execution order.](img/2f1dedf60e48bc8c62b81cec2f92faa0.png "passes")](/sites/default/files/blog/2019/12/passes-1.png)Figure 3: GCC-8 backend passes presented in their execution order.">

有 321 个编译器通道，其中 239 个是唯一的。这个事实可能反映了 GCC 社区已经有 600 多个贡献者。这些过程中的一些是相同优化的重复运行(例如，[死代码消除](https://en.wikipedia.org/wiki/Dead_code_elimination))。许多编译器通道不止一次遍历[中间表示](https://en.wikipedia.org/wiki/Intermediate_representation) (IR)。例如，[寄存器分配器](https://en.wikipedia.org/wiki/Register_allocation)至少遍历它八次。运行所有这些通道需要很多时间。有人认为，通过关闭大多数通道，我们可以成比例地加快 GCC/LLVM 编译。不幸的是，事实并非如此。

在轻量级 JIT 编译器中使用 GCC 和 LLVM 的最大问题是初始化时间长。对于小代码，初始化会占用编译时间的大部分，小方法代码是 Ruby 和其他[动态](https://en.wikipedia.org/wiki/Dynamic_programming_language)高级语言中的其他程序的典型场景。

下图说明了 GCC-8/LLVM-8 在英特尔 i7-9700K 上在 [Fedora Core](https://start.fedoraproject.org) 29 下的长启动时间，如图 4 所示:

[![GCC-8/LLVM-8 takes a long time to start up on an Intel i7-9700K under Fedora Core 29.](img/245baa16b7c69b1bbcb5d472f7f0b471.png "startup")](/sites/default/files/blog/2019/12/startup.png)Figure 4: GCC-8/LLVM-8 startup time on an Intel i7-9700K under Fedora Core 29.">

所以，你不能关闭优化，按比例加速 GCC 和 Clang。

### GCC/LLVM 的函数内联

函数内联是获得更好的 JIT 性能的最重要的优化。方法调用是昂贵的，内联允许在更大的范围内进行优化。

当两者都用 Ruby 编写时，将一个方法内联到另一个方法中不是问题。我们可以在 VM 指令级别或者在生成机器代码时这样做。问题是将用 C 编写的方法内联到用 Ruby 编写的方法中，反之亦然。

这里有一个小例子:

```
  x = 2
  10.times {x *= 2}
```

由 CRuby [VM](https://en.wikipedia.org/wiki/Virtual_machine) 解释的 Ruby 代码调用 C 中实现的方法`times`，这个 C 代码反复调用另一个由 VM 解释的 Ruby 方法并实现一个乘法运算，如图 5 所示:

[![How the times method works with CRuby.](img/a023eafa82cbb6d52d4f41b62667c99f.png "times")](/sites/default/files/blog/2019/12/times.png)Figure 5: The `times` method through CRuby.">

我们如何将代码的所有三个部分集成到一个生成的函数中？让我们考虑一下 MJIT 的当前结构，如图 6 所示:

[![MJIT's structure.](img/6dc28627b652097b0c9623451eeebd5f.png "SMJIT-header")](/sites/default/files/blog/2019/12/SMJIT-header.png)Figure 6: MJIT's structure.">

实现标准 Ruby 方法并且可以内联的 C 函数应该在预编译头文件中。我们在环境头中插入的代码越多，MJIT 预编译头的生成就越慢，因此 MJIT 的启动就越慢。另一个后果是由于更大的预编译头，JIT 代码生成更慢。

我们也可以用 Ruby 重写 C 代码。有时这可能行得通，但在大多数情况下，这样做会导致生成代码的速度变慢。

## 轻量级 JIT 编译器

在分析了当前 MJIT 的缺点之后，我得出结论，轻量级 JIT 编译器可以解决这些问题。我认为轻量级 JIT 编译器应该是对现有 MJIT 编译器的补充，或者是在当前编译器不工作的情况下唯一的 JIT 编译器。

轻量级 JIT 编译器还有另一个原因。我相信这个编译器可以成为 MRuby JIT 的一个很好的解决方案，并有助于将 Ruby 的使用从主要的服务器市场扩展到移动和物联网市场。

所以，去年我开始在业余时间开发一个轻量级的 JIT 编译器。因为我不仅想为 Ruby 使用 JIT 编译器，所以我决定让它成为一个通用的 JIT 编译器和一个独立的项目。

## 米尔

JIT 的核心概念是一种定义良好的中间语言，称为中间内部表示，简称 MIR。你可以在 Steven Muchnik 的名著《高级编译器设计与实现》中找到这个名字。[Rust](https://www.rust-lang.org/)团队也将这个术语用于 Rust 中间语言。尽管如此，我还是决定用同一个名字，因为我喜欢它:MIR 在俄语中是“和平”和“世界”的意思。

MIR 是强类型的，足够灵活。不同形式的 MIR 能够代表 [CISC](https://en.wikipedia.org/wiki/Complex_instruction_set_computer) 和 [RISC](https://en.wikipedia.org/wiki/Reduced_instruction_set_computer) 处理器的机器码。虽然这个轻量级 JIT 编译器是一个独立的项目，但我打算先用 CRuby 或 MRuby JIT 来尝试一下。

为了更好地理解 MIR，让我们考虑厄拉多塞素数筛算法代码作为例子。下面是 sieve 的 C 代码:

```
#define Size 819000
int sieve (int iter) {
  int i, k, prime, count, n; char flags[Size];

  for (n = 0; n < iter; n++) {
    count = 0;
    for (i = 0; i < Size; i++)
      flags[i] = 1;
    for (i = 0; i < Size; i++)
      if (flags[i]) {
        prime = i + i + 3;
        for (k = i + prime; k < Size; k += prime)
          flags[k] = 0;
        count++;
      }
  }
  return count;
}
```

这是相同代码的 MIR 文本表示。MIR 中没有硬寄存器，只有类型化变量。[调用约定](https://en.wikipedia.org/wiki/Calling_convention)也被隐藏:

```
m_sieve:  module
         export sieve
sieve:    func i32, i32:iter
         local i64:flags, i64:count, i64:prime, i64:n, i64:i, i64:k, i64:temp
         alloca flags, 819000
         mov n, 0
loop:
         bge fin, n, iter
         mov count, 0;   mov i, 0
loop2:
         bgt fin2, i, 819000
         mov ui8:(flags, i), 1;   add i, i, 1
         jmp loop2
fin2:
         mov i, 0
loop3:
         bgt fin3, i, 819000
         beq cont3, ui8:(flags,i), 0
         add temp, i, i;   add prime, temp, 3;   add k, i, prime
loop4:
         bgt fin4, k, 819000
         mov ui8:(flags, k), 0;   add k, k, prime
         jmp loop4
fin4:
         add count, count, 1
cont3:
         add i, i, 1
         jmp loop3
fin3:
         add n, n, 1;  jmp loop
fin:
         ret count
         endfunc
         endmodule
```

`func`伪指令中的第一个操作数是函数返回类型(一个 MIR 函数可以返回多个值)，之后声明所有函数参数。局部变量通过*局部*伪指令声明为 64 位整数。

## 轻量级 JIT 编译器项目目标

我为 JIT 编译器设定了性能目标。与使用-O2 的 GCC 相比，这个编译器的编译速度要快 100 倍，启动速度要快 100 倍，代码大小要小 100 倍。至于生成的代码性能，我决定应该至少是 GCC -O2 性能的 70%。

实现也应该比 10K C 线简单，因为我想更广泛地采用这个工具。简单的代码更容易学习和维护。我也想避免这个项目的任何外部依赖性。在文章的最后，你可以看到我现在的实际成果。

### 如何实现这些绩效目标

优化编译器既庞大又复杂，因为它们试图改进任何代码，包括罕见的边缘情况。因为它们要做很多事情，所以编译速度变得很重要。这些编译器使用最快的算法和数据结构来编译不同大小的程序，从小到大，即使算法和数据结构很复杂。

因此，为了实现我们的目标，我们需要使用一些最有价值的优化，只优化经常出现的情况，并使用简单性和性能最佳结合的算法。

那么最有价值的优化有哪些呢？最重要的优化是有效利用最常用的 CPU 资源:指令和寄存器。因此，最有价值的优化是良好的寄存器分配(RA)和[指令选择](https://en.wikipedia.org/wiki/Instruction_selection)。

最近，我做了一个实验，在 GCC 中只打开一个快速简单的 RA 和合并器。没有这样做的选项，我需要修改 GCC。(如果有人感兴趣，我可以提供一个补丁。)与 GCC-9.0 中使用-O2 进行的数百项优化相比，这两项优化在 Fedora Core 29 下的英特尔 i7-9700K 计算机上通过最可靠的编译器基准套件之一 [SpecCPU](https://www.spec.org/benchmarks.html#cpu) 实现了近 80%的性能:

| SPECInt2000 为。 | GCC -O2 | GCC -O0 +简单 RA +组合器 |
| --- | --- | --- |
| -fno-内嵌 | Five thousand four hundred and fifty-eight | 4342 (80%) |
| -鳍线 | Six thousand one hundred and forty-one | 4339 (71%) |

你可能会问，“这怎么可能？其他众多优化是做什么的？”在优化编译器方面工作了多年之后，我想借此机会说一下，提高成熟的优化编译器的性能是多么困难。

### 优化编译器性能的现实

在集成电路世界里，有[摩尔定律](https://en.wikipedia.org/wiki/Moore%27s_law)，它说电路上的晶体管数量每 18 个月翻一番。

在优化编译器领域，人们喜欢提到 [Proebsting 定律](http://proebsting.cs.arizona.edu/law.html)来说明提高生成代码的性能有多难。Proebsting 定律说，优化编译器开发人员每 18 年就能提高生成代码性能两倍。

其实这个“定律”太乐观了，我从来没看到有人测试过这个。(很难查，因为需要等 18 年。)所以，最近我在 SPEC CPU2000 上查了一下。SPEC CPU 是优化编译器开发人员使用的最可靠的基准测试套件之一。它包含一组来自现实世界的 CPU 密集型应用程序；例如，GCC 的特定版本就是基准之一。SPEC CPU 有很多版本:2000，2006，2017。我用的是 2000 版，因为它不需要几天就能运行。我在相同的英特尔 i7-4790K 处理器上使用了 17 年前的 GCC-3.1 编译器(第一个支持 AMD64 的 GCC 版本)和最新版本的 GCC-8，分别处于峰值性能模式:

|  | GCC 3.1 (-O3) | GCC 8 (-Ofast -flto -march=native) |
| --- | --- | --- |
| SPECInt2000 w/o 交换机 | Four thousand four hundred and ninety-eight | 5212 (+16%) |

实际的性能提升**只有 16%** ，并没有接近 100%的 Proebsting 定律状态。

看看优化编译器的进展，人们认为我们根本不应该把时间花在它们的开发上。Bernstein 博士是 CRuby 使用的 SipHash 算法的作者，他在 T2 的一次演讲中表达了这个观点。

还有一种观点认为，我们还是应该努力提高生成代码的性能。[据估计](https://www.theregister.co.uk/2013/08/16/it_electricity_use_worse_than_you_thought)，2013 年(在活跃的加密货币开采之前)，计算机消耗了全部发电量的 10%。2040 年[的计算机用电量很容易等同于 2016 年](https://www.sciencealert.com/computers-will-require-more-energy-than-the-world-generates-by-2040)的总发电量。

在[能源比例计算](https://en.wikipedia.org/wiki/Energy_proportional_computing)(IT 行业的一个目标)中，编译器性能提高 1%意味着从[每年 25，000 太瓦时的世界电力生产](https://en.wikipedia.org/wiki/Electricity_generation)中节省 25 太瓦时(TWh)。

25 太瓦时相当于六座胡佛大坝的平均年发电量。

## ![](img/5de80ce5a6fd972d69a8a6105cfb48a0.png)

虽然优化编译器的主题对我来说很有趣，但是让我们回到 MIR 项目的描述。

## 和平号项目的现状

图 7 显示了详细描述 MIR 项目当前状态的图表:

[![Current MIR project diagram](img/250b2be252dae7b5d9692371eb63f9bf.png "mir3")](/sites/default/files/blog/2019/12/mir3.png)Figure 7: The current state of the MIR project.">

目前，我可以通过 API 或从 MIR 文本或二进制表示创建 MIR。MIR 二进制表示比文本表示紧凑 10 倍，读取速度快 10 倍。

我可以解释 MIR 代码，并从 MIR 在内存中生成 AMD64 机器码。最近我摆脱了 MIR 解释器唯一使用的外部依赖，即 [Libffi](https://sourceware.org/libffi) 外来函数接口库。

我可以从 MIR 生成 C 代码，我正在开发一个 C-to-MIR 编译器，在我看来已经完成了 90%左右。

## 和平号项目未来可能的方向

图 8 显示了该项目的可能发展方向:

[![ossible future development directions for the MIR project.](img/a2dc7b66850eac3fb0e8cfde4cfcdcca.png "mirall")](/sites/default/files/blog/2019/12/mirall.png)Figure 8: Possible future development directions for the MIR project.">

从 [LLVM 内部表示](https://llvm.org/docs/LangRef.html)生成 MIR 允许使用 LLVM 实现的不同语言的代码；比如铁锈或者[水晶](https://crystal-lang.org)。

从 [Java 字节码](https://en.wikipedia.org/wiki/Java_bytecode)生成 MIR 对于用 [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine) 实现的语言也是一样的。

从 MIR 生成 [WebAssembly](https://webassembly.org) 可以允许在 web 浏览器中使用 MIR 代码。实际上，MIR 的特性与 WebAssembly 的特性非常接近。唯一大的区别是 WebAssembly 是基于栈的内部表示，而 MIR 是基于寄存器的。

如果所有的方向都实现了，会有很多有趣的可能性。

## MIR 发生器

目前，最有趣的组件，至少对我来说，是产生优化机器代码的 MIR 生成器。图 9 显示了它的工作原理:

[![](img/d60e307ddfe7de0d2aed3bd454100cb4.png "mir-gen")](/sites/default/files/blog/2019/12/mir-gen.png)Figure 9: The process the MIR generator follows.">

下面是更详细的过程。首先，我们尽可能简化 MIR，这意味着只使用寄存器间接内存寻址。这也意味着立即指令操作数只在移动指令中使用。在此过程中，我们还通过局部值编号删除未使用的指令。

然后，我们内联由 MIR 指令`inline`和`call`表示的函数调用，之后，构建[控制流图](https://en.wikipedia.org/wiki/Control-flow_graph)(CFG)——换句话说，就是[基本块](https://en.wikipedia.org/wiki/Basic_block)和它们之间的边。接下来，我们通过所谓的[全局公共子表达式消除](https://en.wikipedia.org/wiki/Common_subexpression_elimination) (GCSE)，转换代码以在全局范围内重用已经计算的值。这一步对内联代码是一个重要的优化。

下一遍主要通过死代码消除优化移除在前一遍中发现冗余的指令。然后，我们做[稀疏条件常数传播](https://en.wikipedia.org/wiki/Sparse_conditional_constant_propagation) (SCCP)。当存在常量调用参数时，这一步对于内联代码也是一个重要的优化。我们还发现可以有常量表达式值。

常量值可以在任何执行过程中关闭 CFG 路径。关闭这些路径也可以使其他值保持不变。SCCP 是一种组合优化，这意味着其结果优于执行常数传播和移除未使用的 CFG 路径这两种单独的优化。

之后，我们运行机器相关的代码。例如，大多数 x86 指令还要求指令结果操作数与输入操作数之一相同。因此，运行机器相关代码来产生双操作数指令。这段代码还为调用参数传递和返回生成额外的 MIR 指令。

下一步是在 MIR 代码 CFG 中找到[自然循环](https://web.cs.wpi.edu/~kal/PLT/PLT8.6.4.html)。该信息将用于后续的寄存器分配。然后，我们为 MIR 变量计算[活信息](https://en.wikipedia.org/wiki/Live_variable_analysis)。在编译器的世界里，这种技术被称为逆向[数据流问题](https://en.wikipedia.org/wiki/Data-flow_analysis)。

一旦这项工作完成，我们就可以计算 MIR 变量所在的程序点。这个信息在一个快速寄存器分配器中使用，该分配器将目标硬寄存器或堆栈槽分配给镜像变量，并删除各种复制指令。在寄存器分配之后，我们重写 MIR 代码，将变量更改为分配的硬寄存器或堆栈槽。

然后，我们尝试将数据相关的 MIR 指令对组合成其形式可以表示机器指令的指令对。这是一个指令选择任务。这一过程还进行[复制传播](https://en.wikipedia.org/wiki/Copy_propagation)优化。

在指令组合之后，通常有许多指令的输出永远不会被使用。我们删除此类说明。最后，我们在内存中生成机器代码。每个 MIR 指令由一个机器指令编码。对于 AMD64，编码可能会很复杂。

## MIR 发生器特性

如今，大多数编译器优化都是针对[静态单赋值形式](https://en.wikipedia.org/wiki/Static_single_assignment_form)实现的。这是 IR 的一种特殊形式，函数 IR 中的每个变量只有一个赋值。它是很久以前由 IBM 的研究人员发明的。

使用 SSA 简化了优化实现，但是构建和销毁这种表单的成本很高。将 SSA 用于 MIR 生成器的短通管道对我来说没有什么意义，所以我不使用它。

另外，我不生成与位置无关的代码。我看不出现在有什么理由这样做。生成这样的代码更加复杂，并且会降低性能。对于 AMD Geode 处理器(用于[每个孩子一台笔记本电脑计划](https://en.wikipedia.org/wiki/One_Laptop_per_Child)的一款笔记本电脑中)，性能下降了 7%。对于现代处理器，性能下降要小得多，但仍然存在。

## 将 C 编译成 MIR 的可能方法

要开始在 CRuby 中使用 MIR，我需要一个 C-to-MIR 编译器。有几种方法可以实现这个特性。我可以实现一个 LLVM IR-to-M IR 编译器或者编写一个以 MIR 为目标的 GCC 端口，但是这样做会对一个特定的外部项目产生很大的依赖性。这也不是一件容易的事情。有一次，我在一个团队中工作，这个团队专门将 GCC 移植到新的目标上，标准的估计是创建一个简单的“hello，world”程序至少需要六个月的时间。

另一方面，人们已经相当快地编写了小型 C 编译器。这里我可以提一下 [Tiny C 编译器](https://en.wikipedia.org/wiki/Tiny_C_Compiler)，以及最近的 [8cc](https://github.com/rui314/8cc) 和 [9cc](https://github.com/rui314/9cc) 编译器项目。

所以，我决定先写自己的 C-to-MIR 编译器。这个编译器应该实现没有可选特性的标准 C11，比如变量数组、复数和原子数据。

主要目标是简单，而不是速度。同样，这样做可以让其他人更容易地学习代码，并减少维护代码所需的工作量。

### C-to-MIR 编译器

简单性通常是通过将任务分成小的、可管理的子任务来实现的。甚至有一个极端的 [nanopass 编译器设计](https://www.cs.indiana.edu/~dyb/pubs/nano-jfp.pdf)用于研究教育中的编译器主题。

我的 C 编译器实现方法是在大约相同大小的四个通道上进行经典除法，如图 10 所示:

[![Flow chart for the C-to-MIR compiler.](img/5ee66f5a58b6eeb928e26d92a645090a.png "mir2c")](/sites/default/files/blog/2019/12/mir2c.png)Figure 10: The C-to-MIR compiler approach.">

我不像 YACC 那样使用任何工具进行编译。还有，我不修改 ANSI 标准语法，虽然它[模棱两可](https://en.wikipedia.org/wiki/Ambiguous_grammar)。我使用解析表达式语法( [PEG](https://en.wikipedia.org/wiki/Parsing_expression_grammar) )手动解析器。它是一个带有适度回溯的解析器，简单小巧，但比[确定性解析器](https://en.wikipedia.org/wiki/Deterministic_parsing)稍慢。

MIR-to-C 编译器大部分已经实现。它通过了来自不同 C 测试套件的大约 1000 次测试。最近，我实现了一个重要的里程碑:成功的引导。新的`c2m`编译自己的源代码并生成一个 MIR 二进制文件。这个 MIR 二进制文件的执行再次处理`c2m`source 并生成另一个 MIR 二进制文件，两个 MIR 二进制文件是相同的(后面`c2m`调用的选项`-el`是指通过懒惰机器代码生成执行生成的 MIR 代码):

```
   cc -O3 -fno-tree-sra -std=gnu11 -Dx86_64 -I. mir-gen.c c2mir.c c2mir-driver.c mir.c -ldl -o c2m
  ./c2m -Dx86_64 -I. mir-gen.c c2mir.c c2mir-driver.c mir.c -o 1.bmir
  ./c2m 1.bmir -el -Dx86_64 -I. mir-gen.c c2mir.c c2mir-driver.c mir.c -o 2.bmir
```

尽管如此，仍需要付出大量努力来完成缺失的强制性 C11 标准功能(例如，宽字符)，实现完全的呼叫 ABI 兼容性，为交换机生成更有效的代码，以及实现 CRuby JIT 实现所必需的 GCC C 扩展。

### LLVM 红外到红外编译器

我还在开发一个 LLVM IR-to-MIR 编译器。目前，我正专注于从标准 C 代码翻译由 Clang 产生的 LLVM IR。当我们想要生成更优化的 MIR 代码并且不需要快速 C 编译器时(例如，当为编程语言构建基于 MIR 的 JIT 时)，这样做将是有用的。

现在唯一缺少的部分是复合值的翻译，在极少数情况下，当在函数调用中传递小结构或从函数调用中返回它们时，会产生 Clang。将来，这个翻译器可以扩展到支持用 LLVM 实现的其他编程语言生成的代码，比如 Rust。

我希望 [LLVM IR 将在不久的将来](https://llvm.org/devmtg/2019-10/)稳定下来，并且不需要大量的维护。

## 将 MIR 用于 CRuby MJIT 的可能方法

那么，我打算如何将和平号项目用于克鲁比呢？图 11 显示了当前的 MJIT 是如何工作的:

[![Diagram showing how the current MJIT works.](img/2bd7d7871ea0e88dd13b67bd55a61fe7.png "SMJIT0")](/sites/default/files/blog/2019/12/SMJIT0.png)Figure 11: How the current MJIT works.">

### MIR 编译器作为 CRuby 中的第一层 JIT 编译器

图 12 显示了在将基于 MIR 的 JIT 编译器实现为第一层编译器之后，未来的 MJIT 将会是什么样子:

[![How the future MJIT would look after implementing a MIR-based JIT compiler as a tier one compiler.](img/fe738e3f4d35688a0ec39670de0e1cb4.png "SMJIT1")](/sites/default/files/blog/2019/12/SMJIT1.png)Figure 12: How the future MJIT would look after implementing a MIR-based JIT compiler as a tier-one compiler.">

蓝色部分显示了 MJIT 的新数据流。在构建 CRuby 时，我们可以为用 c 编写的标准 Ruby 方法生成 MIR 代码。这一部分可以很快完成。

MJIT 可以通过 MIR API 为 Ruby 方法创建 MIR 代码。该镜像代码可以内嵌已经存在的镜像代码功能。对应的机器代码可以由 MIR 生成器生成。这是一种快速的方法，但是我们也可以通过将 MIR 翻译成 C，然后使用 GCC 或 LLVM 来生成机器码。这是一种慢得多的方法，但是它允许我们使用 GCC/LLVM 优化的全部功能，并且它允许高效地实现 C 代码到 Ruby 代码的内联，反之亦然。

### MIR 编译器作为 CRuby 中的单个 JIT 编译器

对于某些环境，MIR JIT 编译器可以作为单个 JIT 编译器使用。在这种情况下，带有 MJIT 的 MIR 将如图 13 所示:

[![Diagram showing how the MIR compiler would work as a single JIT compiler in CRuby.](img/7e9fa15793dd4bc9c33f430dfb0ea6d2.png "SMJIT2")](/sites/default/files/blog/2019/12/SMJIT2.png)Figure 13: How the MIR compiler would work as a single JIT compiler in CRuby.">

### 在 CRuby 中将 MIR 编译器和 C-to-MIR 编译器作为单个 JIT 编译器

我们可以先生成 C 代码，然后用 C-to-MIR 翻译器把它翻译成 MIR，而不是直接为 JIT 编译器生成 MIR。在这种情况下，MJIT 将如图 14 所示:

[![Diagram showing how the MIR compiler combined with the C-to-MIR compiler would work in CRuby.](img/0b4c75c654bf466a1e0658ea34053e1f.png "SMJIT3")](/sites/default/files/blog/2019/12/SMJIT3.png)Figure 14: How the MIR compiler combined with the C-to-MIR compiler would work in CRuby.">

我和几个人讨论了轻量级 JIT 编译器项目，其中两个人独立地想要为 JIT 生成 C 代码。将基于 MIR 的 JIT 编译器用于他们自己的 JIT 实现会使他们的生活更容易。

C-to-MIR JIT 编译器非常小，启动速度很快，可以用作从内存中读取生成的 C 代码的库。虽然 C-to-MIR 转换器的设计速度并不快，但它仍然比使用-O2 的 GCC 快 15 倍左右。所有这些都使得这种方法可行。

## 当前绩效结果

最后，这里是 MIR 生成器和解释器与 GCC-8.2.1 相比的当前性能结果。我在运行 Fedora Core 29 的 Intel i7-9700K 机器上使用了 sieve 基准测试。sieve C 代码没有包含指令，只有大约 30 行预处理代码:

|  | MIR-gen | MIR-interp | gcc -O2 | 海湾合作委员会-O0 |
| --- | --- | --- | --- | --- |
| 编译[1] | **1.0** (75us) | 0.16(12 美国) | 178(13.35 毫秒) | 171(12.8 毫秒) |
| 执行[2] | **1.0** (3.1s) | 5.9(18.3 秒) | **0.94** (2.9s) | 2.05(6.34 秒) |
| 代码大小[3] | 1.0 (175KB) | 0.65(114 千字节) | **144** (25.2MB) | 144(25.2 兆字节) |
| 启动[4] | **1.0**(1.3 美制) | 1.0(1.3 美国) | 9310(12.1 毫秒) | 9850(12.8 毫秒) |
| 锁定[5] | **1.0** (16K) | 0.56 (9K) | 93 (1480K) | 93 (148 万) |

MIR-generator 的编译速度比使用-O2 的 GCC 快大约 180 倍。生成筛子的代码需要 80 微秒。而 MIR 生成器生成的 sieve 代码只慢了 6%。

MIR 发生器的物体尺寸远小于`cc1`的物体尺寸。MIR 生成器启动时间很快，适合用作 tier1 JIT 编译器。

以下是每个表格行的注释:

*   [1]:筛选代码编译的壁时间(没有任何包含文件，使用 GCC 的内存文件系统)。
*   【2】:10 次跑步的最佳上墙时间。
*   [3]:GCC 的`cc1`的剥离尺寸，MIR 的 MIR 核和解释器或生成器的剥离尺寸。
*   [4]:空 C 文件的目标代码生成时间，或通过 API 生成空 MIR 模块的时间。
*   [5]:仅基于 AMD64 C 编译器所需的文件以及创建和运行 MIR 代码所需的最少文件数。

## 当前米尔 SLOC 分布

图 15 显示了 MIR 项目当前版本的源代码行分布。MIR-to-C 编译器大约有 12000 行 C 代码。和平号核心大约有九千行:

[![A source line distribution for the current version of the MIR project.](img/a16141c6592b7f11ad1d5f9d0b79d126.png "size-chart")](/sites/default/files/blog/2019/12/size-chart-1.png)Figure 15: A source line distribution for the current version of the MIR project.">

和平号发电机不到五千线。

生成器使用的机器相关代码大约有 2000 行，因此您可以估计将 MIR 生成器移植到另一个目标所需的工作量。例如，我预计将 MIR 移植到 [Aarch64](https://en.wikipedia.org/wiki/ARM_architecture#AArch64) 将花费我一到两个月的时间。

## MIR 项目竞争对手

我研究 JITs 很久了。在开始 MIR 项目之前，我考虑过修改现有的代码。下面是 MIR 与我考虑的简单编译器项目的比较:

| 项目 | SLOC | 许可证 | IR type | 主要优化 | 输出 |
| --- | --- | --- | --- | --- | --- |
| 米尔 | 16K C | 用它 | 非特别服务协定 | 内嵌，GCSE，SCCP，RA，CP，DCE，LA | 二进制码 |
| 多纳特·利吉特 | 80K 摄氏度 | LGPL | 非特别服务协定 | 只有 RA 和原始 CP | 二进制码 |
| 。网络龙 JIT | 360K C++ | 用它 | 社会保障总署(Social Security Administration) | 和平号减去 SCCP 加 LICM，距离 | 二进制码 |
| QBE | 10K 角 | 用它 | 社会保障总署(Social Security Administration) | 镜像 1 加别名减内联 | 装配工 |
| LIBFirm | 14 万摄氏度 | LGPL2 | 社会保障总署(Social Security Administration) | RA，内联，DCE，LICM，CP，LA，其他 | 装配工 |
| 起重机升降机 | 70K 铁锈 | 街头流氓 | 社会保障总署(Social Security Administration) | DCE，LICM，罗德岛，GVN，加拿大，洛杉矶 | 二进制码 |

以下是表中使用的缩写:

*   CP: [复制传播](https://en.wikipedia.org/wiki/Copy_propagation)
*   DCE: [死码消除](https://en.wikipedia.org/wiki/Dead_code_elimination)
*   GCSE: [全局公共子表达式消除](https://en.wikipedia.org/wiki/Common_subexpression_elimination)
*   GVN: [全局数值编号](https://en.wikipedia.org/wiki/Value_numbering)
*   LA:在 CFG 中查找循环
*   LICM: [循环不变代码运动](https://en.wikipedia.org/wiki/Loop-invariant_code_motion)
*   RA: [寄存器分配](https://en.wikipedia.org/wiki/Register_allocation)
*   范围:[范围分析](https://en.wikipedia.org/wiki/Value_range_analysis)
*   SCCP: [稀疏条件常数传播](https://en.wikipedia.org/wiki/Sparse_conditional_constant_propagation)

其他包括:

*   [部分冗余消除](https://en.wikipedia.org/wiki/Partial_redundancy_elimination)
*   [循环展开](https://en.wikipedia.org/wiki/Loop_unrolling)
*   标量替换
*   [尾部递归消除](https://stackoverflow.com/questions/1240539/what-is-tail-recursion-elimination)
*   表达式重新关联
*   功能克隆
*   [强度降低](https://en.wikipedia.org/wiki/Strength_reduction)
*   [循环优化](https://en.wikipedia.org/wiki/Loop_optimization)
*   装载存储运动
*   [跳线程](https://en.wikipedia.org/wiki/Jump_threading)

在仔细研究了这些项目之后，我决定启动 MIR 项目。现有项目的最大缺点是它们的规模，事实上我很难用它们来实现我的目标，而且我无法控制这些项目。此外，MIR 项目较小的源代码大小使得其他人更容易学习代码，并减少了维护代码所需的工作量。

## 和平号项目计划

和平号项目相当雄心勃勃。我决定以开放的方式开发它，因为这允许我在项目的早期阶段从许多人那里获得有价值的反馈。你可以在 [GitHub](https://github.com/vnmakarov/mir) 上关注项目的进展。

尽管 MIR 仍处于开发的早期阶段，但我计划很快在 CRuby/MRuby JIT 实现中使用它。这将是改进 MIR 编译器实现并找到缺失特性的有用方法。

如果将基于 MIR 的编译器用于 Ruby JIT 是成功的，我将致力于 MIR 项目代码的第一个发布，我希望这将对其他编程语言的 JIT 实现有用。

*Last updated: December 10, 2020*