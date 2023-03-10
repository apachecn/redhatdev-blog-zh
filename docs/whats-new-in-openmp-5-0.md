# OpenMP 5.0 的新功能

> 原文：<https://developers.redhat.com/blog/2019/03/19/whats-new-in-openmp-5-0>

新版本的 [OpenMP](https://www.openmp.org) 标准 5.0 于 2018 年 11 月发布，为用户带来了几个新的构造。OpenMP 是一种 API，由编译器指令和库例程组成，用于 C、C++和 [Fortran](https://opensource.com/article/17/11/happy-60th-birthday-fortran) 程序中的高级并行。即将到来的 [GCC](https://developers.redhat.com/blog/2019/03/08/usability-improvements-in-gcc-9/) 版本增加了对这个最新版本标准的某些部分的支持。

本文重点介绍了 OpenMP 标准中的一些最新特性、变化和“陷阱”。

## 任务再教育

该标准的新版本允许在`taskloop`结构上使用`reduction`子句，并增加了新的子句:`taskgroup`结构使用`task_reduction`，而`task`和`taskloop`结构使用`in_reduction`。以前，变量只能在“并行”区域的线程之间或“团队”区域的团队之间或 SIMD 通道之间减少。现在，在构造上，归约变量被私有化了，可以创建它们的许多副本。

对于任务缩减，实现可以从各种方法中进行选择。例如，它可以创建一个变量私有副本的数组，每个线程一个元素，并在`taskgroup`开始时初始化私有副本，或者它可以在任务第一次在`in_reduction`中引用私有副本时创建私有副本。它甚至可以在一个任务第一次访问私有副本时，以更慢的速度创建它们，或者它可以选择其他方式。在`taskgroup`构造完成时，变量需要减少。

只有在没有指定`nogroup`子句的情况下，才允许在`taskloop`结构上使用`reduction`子句；因此，如果在任务周围有一个隐式的`taskgroup`，那么该子句既作为隐式`taskgroup`的`task_reduction`子句，又作为该构造创建的单个显式任务的`in_reduction`。当在`parallel`或`workshare`构造上指定`reduction`子句时，可以有一个`task`修饰符，这样的约简就可以用在执行`parallel`或`workshare`构造时遇到的`task`和`taskloop`构造上的`in_reduction`子句中。

这里有一个例子:

```
int foo () {
  int r = 0;
  #pragma omp taskloop reduction (+:r)
  for (int i = 0; i < 128; i++)
    r += work (i);
  return r;
}
```

在为`taskloop`构造创建的任务内部(每个任务都可以处理一次或多次迭代)，对`r`变量的引用可能是指在第一次访问变量之前初始化的该变量的私有副本(在本例中，在任务内部初始化为 0)，通常使用用户定义的归约初始化器。当`taskloop`构造完成时，应该使用归约组合器(在本例中是`omp_out += omp_in`)从任何私有副本中归约原始的`r`。当所有任务完成时，实现可以选择是否进行锁定、使用原子操作或串行减少。

任务缩减甚至可以在需要时对私有化的全局变量起作用，如下例所示。`taskgroup`构造为`r`变量建立了任务缩减，任何具有相应的`in_reduction`子句的任务都将参与该缩减。(注意，`in_reduction`和相应的`task_reduction`或`reduction`子句的参数必须相同。)函数`foo`中的`taskloop`结构显示，那里的`reduction`子句行为类似，但是在由`taskloop`创建的任务中直接引用的`r`也参与了缩减。

```
int r;
void bar (int i) {
  #pragma omp task in_reduction (+:r)
  r += work (i, 0);
  #pragma omp task in_reduction (+:r)
  r += work (i, 1);
}
int foo () {
  #pragma omp taskgroup task_reduction (+:r)
  bar (0);
  #pragma omp taskloop reduction (+:r)
  for (int i = 1; i < 4; ++i)
    { bar (i); r += i; }
}
```

## ！=条件和 C++范围循环

旧版本的标准要求循环条件仅使用`<`、`>`、`<=`或`>=`比较。OpenMP 的新版本允许将`!=`用作循环条件，如下例所示，尽管在这种情况下，增量表达式必须将迭代器变量递增或递减 1。然后，在编译时，编译器可以确定它是递增还是递减。尤其是用 C++，随机访问迭代器的用户习惯写`!=`而不是`<`或者`>`；当编译器可以在编译时判断出循环是递增还是递减时，它可以将循环转换为`<`或`>`比较，因此这种变化主要是语法上的。

```
#pragma omp for
for (auto iv = something.begin (); iv != something.end (); ++iv)
/* ... */;
```

OpenMP 5.0 标准增加了对许多 C++11、C++14 和 C++17 特性的支持，C++范围`for`就是其中之一。所以你可以用这个:

```
#pragma omp parallel for
for (auto x : vec)
/* ... */;
```

并且编译器会在`parallel`的线程间拆分工作。

## 主办团队构建

在 OpenMP 4.5 中，`teams`构造过去只允许直接嵌套在`target`构造内用于卸载。OpenMP 5.0 标准允许`teams`也用于主机并行化，特别是对于不同 NUMA 节点之间的通信可能非常昂贵的 NUMA 系统。不同的团队不参与正常的同步，这是在“平行”区域内完成的。您可以使用`distribute`结构在不同的 NUMA 节点之间分配工作，并且在每个 NUMA 节点内，使用`parallel`结构和可能的`simd`结构进行并行化。

## 依赖子句中的迭代器

在`depend`子句中，人工迭代器可用于在运行时创建数量可变的`depend`子句。在下面的代码中:

```
#pragma omp task depend(iterator (i=0:64:16, long j=v1:v2), in: arr[i][j])
;
```

如果`v1`变量的值是 2，而`v2`变量的值是 4，那么上面的代码在运行时是这样处理的:

```
#pragma omp task depend(in: arr[0][2], arr[0][3], arr[16][2], arr[16][3]) \
                 depend(in: arr[32][2], arr[32][3], arr[48][2], arr[48][3])
```

其中`i`人工变量具有`int`类型，`j`具有`long`类型，并且仅在`depend`子句的范围内；`i`用步骤 16 从 0(含)到 64(不含)迭代，`j`用步骤 1 从`v1`(含)到`v2`(不含)迭代。

## 原子结构变化

在 OpenMP 4.5 中，`atomic`构造要么是宽松的(当没有指定额外的子句时)，要么是顺序一致的(与`seq_cst`子句一致)。OpenMP 5.0 允许您显式指定各种其他内存排序行为(`relaxed`、`acquire`、`release`、`acq_rel`子句)，并允许您在没有通过新的`requires`指令显式指定时更改默认值。此外，您可以指定一个`hint`子句(尽管目前 GCC 只是解析它并在以后忽略它)。在`flush`结构中，您还可以指定一个 memory order 子句。

```
#pragma omp requires atomic_default_mem_order(seq_cst)
// All atomic constructs will be in this translation unit
// sequentially consistent unless specified otherwise.

#pragma omp atomic update // This will be now seq_cst
i += 1;
#pragma omp atomic capture relaxed // This will be relaxed
v = j *= 2;
```

## 各种新的组合结构

作为语法糖，现在支持几个新的组合结构，特别是在使用 OpenMP 任务时节省输入。其中包括:

```
#pragma omp parallel master
#pragma omp parallel master taskloop
#pragma omp parallel master taskloop simd
#pragma omp master taskloop
#pragma omp master taskloop simd
```

## omp _ 暂停 _ 资源

OpenMP 5.0 新增了两个 API 调用，`omp_pause_resource`和`omp_pause_resource_all`，用户可以通过它们请求库释放资源(线程、卸载设备数据结构等)。).例如，当 OpenMP 指令以前已经使用过并且也将在子进程中使用时，这可以允许在没有立即执行的情况下使用`fork`。

## 数据共享变化

在 GCC 9 中，有一个在 OpenMP 5.0 中实际上并不新鲜的变化。OpenMP 4.0 引入了一个可能是错误的变化，即`const`限定变量不再被预先确定为共享变量。这有一个预期的效果，即您可以在一个`shared`子句中指定这些变量(这可能是该变化的动机)，但它也有一个不良的效果，即当使用`default(none)`子句时，如果在构造内部使用了`const`限定变量，则必须在一些数据共享子句中显式指定它们，而以前它们不必如此。在较老的 GCC 版本中，我希望逆转这种变化，但是大家一致认为这不会改变。遇到这种情况的用户有多种选择；详见 [OpenMP 数据共享](https://gcc.gnu.org/gcc-9/porting_to.html#ompdatasharing)。

## 尝试一下

OpenMP 5.0 标准的新版本[已经发布](https://www.openmp.org/wp-content/uploads/OpenMP-API-Specification-5.0.pdf)，并且包含了比上述更多的新特性。另外，请参见 OpenMP 5.0 [参考指南](https://www.openmp.org/wp-content/uploads/OpenMPRef-5.0-111802-web.pdf)。即将发布的 GCC 版本( [GCC 9](https://developers.redhat.com/blog/2019/03/08/usability-improvements-in-gcc-9/) )将支持上述特性和其他各种特性(目前仅针对 C 和 C++)，但许多其他 OpenMP 5.0 新特性将仅在以后的 GCC 版本中实现。请参见 [OpenMP 5.0 对 GCC 9 的支持](https://gcc.gnu.org/ml/gcc-patches/2018-11/msg00628.html)了解 GCC 9 中具体实现了什么，以及在以后的 GCC 版本中将会实现什么。

## 面向 C/C++开发人员的更多文章

*   [GCC 9 中的可用性改进](https://developers.redhat.com/blog/2019/03/08/usability-improvements-in-gcc-9)
*   [了解 GCC 警告](https://developers.redhat.com/blog/2019/03/13/understanding-gcc-warnings/)
*   [如何在红帽企业版 Linux 7 上安装 GCC 8 和 Clang/LLVM 6](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/#more-568577)
*   [GCC 的推荐编译器和链接器标志](https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/)
*   [GCC 8 中的可用性改进](https://developers.redhat.com/blog/2018/03/15/gcc-8-usability-improvements/)
*   [Clang/LLVM 入门](https://developers.redhat.com/blog/2017/11/01/getting-started-llvm-toolset/)
*   [用 GCC 8 检测字符串截断](https://developers.redhat.com/blog/2018/05/24/detecting-string-truncation-with-gcc-8/)
*   [通过 GCC 7 进行的隐式下降检测](https://developers.redhat.com/blog/2017/03/10/wimplicit-fallthrough-in-gcc-7/)
*   [使用 GCC 7 进行内存错误检测](https://developers.redhat.com/blog/2017/02/22/memory-error-detection-using-gcc/)
*   [用 GCC 插件诊断函数指针安全缺陷](https://developers.redhat.com/blog/2017/03/17/diagnosing-function-pointer-security-flaws-with-a-gcc-plugin/)
*   [更好地使用 C11 原子——第一部分](https://developers.redhat.com/blog/2016/01/14/toward-a-better-use-of-c11-atomics-part-1/)
*   [如何在红帽企业版 Linux 上安装 GCC 8](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/)

*Last updated: March 18, 2019*