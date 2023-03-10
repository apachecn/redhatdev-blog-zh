# Clang 中的概要引导优化:处理修改的源

> 原文：<https://developers.redhat.com/blog/2020/07/06/profile-guided-optimization-in-clang-dealing-with-modified-sources>

Profile-guided optimization (PGO)是一种改进编译过程的通用编译器技术。在 PGO(有时读作“pogo”)中，管理员使用二进制文件的第一个版本，通过检测或采样来收集*概要文件*，然后使用该信息来指导编译过程。

概要引导优化可以帮助开发人员做出更好的决策，例如，关于内联或块排序。在某些情况下，它还会导致使用过时的配置文件信息来指导编译。出于我将解释的原因，这个特性可以使大型项目受益。这也给编译器实现带来了检测和处理不一致的负担。

本文关注的是 [Clang 编译器](https://developers.redhat.com/blog/category/clang-llvm/)是如何实现 PGO 的，特别是它是如何检测二进制文件的。我们将看看当 Clang 在编译阶段检测源代码以收集执行过程中的概要信息时会发生什么。然后，我将介绍一个真实世界的 bug，展示当前 PGO 方法的缺陷。

**注**:要了解更多关于 Clang 的 PGO，请参见 [*Clang 编译器用户手册*](https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization) 。

## 在 Clang 中检测代码

在 Clang 中，`-fprofile-instr-generate`标志指示编译器在源指令级别检测代码，`-fprofile-generate`标志指示编译器在 LLVM 中间表示(IR)级别检测代码。这两种方法共享一种设计理念，只是在粒度上有所不同。我们的主题是`-fprofile-instr-generate`，它与源代码交互的方式在分析和重新编译之间变化。

考虑以下场景:

1.  用`-fprofile-instr-generate`编译一个代码示例(`C0`)。
2.  运行它以收集配置文件信息(`P0`)。
3.  编辑`C0`样本并将其转换为新版本`C1`。
4.  使用原始的`P0`配置文件信息编译`C1`。

## Clang 如何处理代码修改

使用有些过时的配置文件信息的场景可能看起来很奇怪，因为我们通常*编译、配置、*和*重新编译*。然而，剖析步骤可能相当耗时。在某些情况下，大型项目很容易根据源快照提供可下载的概要信息。然后，管理员可以使用快照来重新编译代码，而不必每次都收集新的概要文件。( [dotnet 运行时](https://github.com/dotnet/runtime)采用了这种方法。)

此外，对于具有高提交率的项目，为每个提交提供概要信息是不可行的。因此，对代码的细微更改可能不会记录在用于重新编译的概要文件中。那么，Clang 会如何应对呢？

“比较整个文件的校验和”这种琐碎的回答并不令人满意，因为一个微小的改变就会使整个编译单元无效。但是实际的机制依赖于相同的想法:在*函数*的基础上，基于树结构，在*抽象语法树* (AST)上计算校验和。这样，更改函数不会使为其他函数收集的配置文件信息无效。当然，这种方法有局限性。移除调用点会改变函数被调用的次数，从而改变其*热度*。但至少它防止了概要信息指向不再存在的代码，反之亦然。

目前，如果使用这种过时的配置文件信息，Clang 编译器会忽略它并打印一条警告:

```
> echo 'int main() { return 0; }' > a.c && clang -fprofile-instr-generate a.c && LLVM_PROFILE_FILE=a.profraw ./a.out
> llvm-profdata merge -output=a.profdata a.profraw
> printf '#include \nint main() { if(1) puts("hello"); return 0; }' > a.c && clang -fprofile-instr-use=a.profdata a.c
warning: profile data may be out of date: of 1 function, 1 has mismatched data
      that will be ignored [-Wprofile-instr-out-of-date]
1 warning generated.

```

## 当不可能的事情发生时

最近，我的任务是调试一个 Clang 分段错误(segfault)，这是在 [Red Hat Bugzilla Bug 1827282](https://bugzilla.redhat.com/show_bug.cgi?id=1827282) 中提出的一个问题。经过调试，我得到了两个具有相同校验和的函数:

```
extern int bar;

// first version
void foo() {
    if (bar) { }
    if (bar) { }
    if (bar) { if (bar) { } }
}

// second version
void foo() {
    if (bar) { }
    if (bar) { }
    if (bar) { if (bar) { if (bar) { } } }
}

```

这是一个奇怪的结果，因为 [Clang 中使用的校验和算法依赖于 MD5](https://llvm.org/doxygen/classllvm_1_1MD5.html) ，所以发生冲突的几率应该*非常低*。不可思议的事情发生了吗？

事实证明并没有。冲突是由于哈希最终确定的方式中的一个小错误，我们用一个补丁( [D79961](https://reviews.llvm.org/D79961) )修复了它。基本上，在计算散列时，需要填充一个缓冲区(`uint64_t`)。一旦它满了，它就被转换成一个字节数组并发送给哈希例程。在最后的步骤中，`uint64_t`被直接发送到例程，并隐式地转换成一个`uint8_t`，从而潜在地忽略了 AST 后面的节点。我们通过添加一个新的测试用例来解决这个问题，这个测试用例简单地测试一个小的函数变化是否反映在哈希值中。

这个补丁起作用了，但是它改变了大多数现有函数的散列——也就是说，改变了那些在最后一个缓冲区中有多个元素的函数的散列。这是一个重要的副作用，因为更改散列会使大多数现有的缓存概要信息失效。幸运的是，该补丁不会影响典型的“编译、分析、重新编译”场景，但是对于大型构建系统来说，这可能是一个问题，因为大型构建系统会预先计算配置文件数据，以便客户端在构建过程中下载。

## 结论

Clang 和 GCC 都支持使用过时的概要信息来指导编译过程。如果函数体发生变化，过时的信息将被忽略。此功能对于收集配置文件信息成本高昂的大型项目非常有用。这给编译器检测和处理不一致增加了额外的负担，也增加了编译器错误的可能性。

*Last updated: June 30, 2020*