# OpenMP 5.0 和 5.1 中的新特性

> 原文：<https://developers.redhat.com/blog/2021/05/03/new-features-in-openmp-5-0-and-5-1>

[OpenMP](https://www.openmp.org) 是由编译器指令和库例程组成的 API，用于 [C 和 C++](/topics/c) 以及 [Fortran](https://opensource.com/article/17/11/happy-60th-birthday-fortran) 中的高级并行。[OpenMP 5.1](https://www.openmp.org/spec-html/5.1/openmp.html)版本于 2020 年 11 月发布，[5.0](https://www.openmp.org/spec-html/5.0/openmp.html)版本于 2018 年 11 月发布。本文讨论了在 GCC 11 中实现的 OpenMP 5.0 的新特性，以及 OpenMP 5.1 的一些新特性。

## OpenMP 5.0 特性

让我们从 OpenMP 5.0 标准版中添加的功能开始。

### 支持非矩形折叠循环

在 OpenMP 5.0 之前，所有 OpenMP 循环构造(工作共享循环、`simd`、`distribute`、`taskloop`以及基于这些的组合或复合构造)都被要求为矩形。这意味着循环嵌套中所有相关循环的所有下限、上限和增量表达式都需要相对于最外层循环保持不变。OpenMP 5.0 仍然要求所有的增量表达式都是循环不变的，但是允许某些情况下内部循环的下限和上限表达式可以基于单个外部循环迭代器。

然而，这个新特性有一些限制:一个内循环迭代器必须最多使用一个外循环迭代器，表达式需要解析为 *a * outer + b* ，其中 *a* 和 *b* 是循环不变的表达式。如果内部循环和引用的外部循环具有不同的增量，则存在进一步的限制，以支持在循环之前折叠循环嵌套的迭代次数的简单计算。此外，非矩形循环可能没有指定的`schedule`或`dist_schedule`子句。这允许实现选择它喜欢的任何迭代分布。

下面的三角形循环就是一个例子:

```
#pragma omp for collapse(2)
for (int i = 0; i < 100; i++)
  for (int j = 0; j < i; j++)
    arr[i][j] = compute (i, j);
```

但是非矩形回路也可以复杂得多:

```
#pragma omp distribute parallel for simd collapse(4)
for (int i = 0; i < 20; i++)
  for (int j = a; j >= g + i * h; j -= n)
    for (int k = 0; k < i; k++)
      for (int l = o * j; l < p; l += q)
        arr[i][j][k][l] = compute (i, j, k, l);
```

最简单的实现是通过计算循环嵌套的矩形外壳，并且在组合的循环体内不做任何事情来进行原始循环不会运行的迭代。例如，对于本节中的第一个循环，实现将是:

```
#pragma omp for collapse(2)
for (int i = 0; i < 100; i++)
  for (int j = 0; j < 100; j++)
    if (j < i)
      arr[i][j] = compute (i, j);
```

不幸的是，这样的实现会导致严重的工作不平衡，有些线程根本不做实际工作。因此，除了非组合的非矩形`simd`构造，GCC 11 在循环之前计算精确的迭代次数。在只有一个循环依赖于外循环迭代器的循环嵌套的情况下，它使用 Faulhaber 的公式，并根据外迭代器的一些值可能导致内循环没有迭代的事实进行了调整。这样，只要循环体在每次迭代中执行大致相同的工作量，工作就会平均分配。

### 条件 lastprivate

在 OpenMP 中，`lastprivate`子句可用于检索在循环的最后一次迭代中分配的私有变量的值。带有条件修饰符的`lastprivate`子句是一种奇特的归约，它从执行最大逻辑迭代次数的线程(或团队、SIMD 巷或任务)中选择值。例如:

```
#pragma omp parallel for lastprivate(conditional:v)
for (int i = 0; i < 1024; i++)
  if (cond (i))
    v = compute (i);
result (v);
```

为了让这个结构工作，私有变量只能通过直接存储来修改，而不应该通过指针或在其他函数内部修改。这使得实现可以很容易地找到这些存储，并调整存储以记住存储它的逻辑迭代。这个特性已经在 GCC 10 中实现了。

### 包含和排除扫描支持

OpenMP 5.0 增加了对实现并行前缀总和(也称为累积总和或包含和排除扫描)的支持。这种支持允许 C++17 `std::inclusive_scan`和`std::exclusive_scan`使用 OpenMP 并行化。语法建立在带有特殊修饰符的`reduction`子句的基础上，新的指令将循环体分成两半。例如:

```
#pragma omp parallel for reduction (inscan, +:r)
for (int i = 0; i < 1024; i++)
  {
    r += a[i];
    #pragma omp scan inclusive(r)
    b[i] = r;
  }
```

然后，实现可以将循环分成两半，不仅为每个线程创建一个私有变量，而且为整个构造创建一个完整的数组。在对所有迭代的一半用户代码进行评估之后——这在包含扫描和排除扫描之间是不同的——可以对私有数组执行前缀和的高效并行计算，最后，另一半用户代码可以由所有线程进行评估。该语法允许代码正常工作，即使 OpenMP 编译指示被忽略。这个特性在 GCC 10 中实现。

### 声明变体支持和元指令

在 OpenMP 5.0 中，一些直接调用可以根据调用它们的 OpenMP 上下文重定向到专门的替代实现。可以根据调用站点在词汇上嵌套在哪个 OpenMP 构造中来实现这种专门化。然后，OpenMP 实现可以根据实现供应商、CPU 架构和编译代码的 ISA 标志等选择正确的替代方案。这里有一个例子:

```
void foo_parallel_for (void);
void foo_avx512 (void);
void foo_ptx (void);
#pragma omp declare variant (foo_parallel_for) \
match (construct={parallel,for},device={kind("any")})
#pragma omp declare variant (foo_avx512) \
match (device={isa(avx512bw,avx512vl,"avx512f")})
#pragma omp declare variant (foo_ptx) match (device={arch("nvptx")})
void foo (void);
```

如果从工作共享循环的词法体中直接调用`foo`，该工作共享循环在词法上嵌套在并行结构中(包括组合的`parallel for`)，该调用将被对`foo_parallel_for`的调用所取代。如果从为前面提到的 AVX512 ISAs 编译的代码中调用`foo`，将改为调用`foo_avx512`。最后，如果从运行在英伟达 PTX 上的代码调用`foo`，编译器将调用`foo_ptx`。

一个复杂的评分系统，包括用户评分，决定在多个变体匹配的情况下使用哪个变体。这个结构在 GCC 10 中得到部分支持，在 GCC 11 中得到完全支持。OpenMP 5.0 规范还允许使用类似语法的元指令，其中可以使用几个不同 OpenMP 指令中的一个，这取决于使用它的 OpenMP 上下文。

### 循环结构

在 OpenMP 4.5 中，各种循环结构规定了实现应该如何划分工作。一个程序员指定了工作是否应该在团队联盟中的团队之间，或者在平行区域的线程之间，或者在一个`simd`构造中的 SIMD 车道之间，等等。OpenMP 5.0 提供了一种新的循环结构，这种结构不太规范，并且在如何实际实现分工方面给实现留下了更多的自由。这里有一个例子:

```
#pragma omp loop bind(thread) collapse(2)
for (int i = 0; i < 1024; i++)
  for (int j = 0; j < 1024; j++)
    a[i][j] = work (i, j);
```

`bind`子句是孤立构造所必需的，它指定遇到它的线程将参与构造。如果 pragma 在词汇上嵌套在 OpenMP 构造中，使得绑定显而易见，则可以省略 bind 子句。允许实现使用额外的线程来执行迭代。GCC 10 中实现了循环结构。

对哪些 OpenMP 指令可以出现在循环体中有限制，并且不能在循环体中使用 OpenMP API 调用。施加这些限制是为了让用户程序无法观察和依赖指令实际上是如何实现的。OpenMP 5.1 中增加了对工作调度的限制，这将在下面讨论。

## OpenMP 5.1 特性

在 OpenMP 5.1 中，C++程序可以使用 C++11 属性来指定 OpenMP 指令，除了以前使用的 pragmas。下面是两个使用属性的示例:

```
[[omp::directive (parallel for, schedule(static))]]
for (int i = 0; i < 1024; i++)
  a[i] = work (I);

[[omp::sequence (directive (parallel, num_threads(16)), \
                 directive (for, schedule(static, 32)))]]
for (int i = 0; i < 1024; i++)
  a[i] = work (i);
```

OpenMP 5.1 增加了一个 scope 指令，所有遇到它的线程都将执行构造体。私有和减让条款可以适用于它。例如:

```
#pragma omp scope private (i) reduction(+:r)
{
  i = foo ();
  r += i;
}
```

除非指令中有`nowait`子句，否则在区域的末尾会有一个隐含的障碍。

OpenMP 5.1 新增了`assume`、`interop`、`dispatch`、`error`和`nothing`指令。还添加了循环转换指令。master 已被弃用，并被新的 masked 构造所取代。有许多新的 API 调用，包括:

*   `omp_target_is_accessible`
*   `omp_get_mapped_ptr`
*   `omp_calloc`
*   `omp_aligned_alloc`
*   `omp_realloc`
*   `omp_set_num_teams`
*   `omp_set_teams_thread_limit`
*   `omp_get_max_teams`
*   `omp_get_teams_thread_limit`

[OpenMP API 特性历史附录](https://www.openmp.org/spec-html/5.1/openmpap2.html#x349-524000B)涵盖了所有变更，包括不推荐使用的特性。

## 尝试一下

OpenMP 5.0 和 OpenMP 5.1 的规格可在 openmp.org/specifications/[获得，包括 PDF 和 HTML 布局。GCC 的最新版本(](https://www.openmp.org/specifications/) [GCC 11](https://gcc.gnu.org/gcc-11/changes.html) )支持本文和其他各种特性(这次不仅仅是 C 和 C++，还有许多 Fortran 的特性)。但是 OpenMP 的其他几个新特性只能在以后的 GCC 版本中实现。

*Last updated: April 29, 2021*