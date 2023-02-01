# 隐式函数声明:flex 对“reallocarray”的使用

> 原文：<https://developers.redhat.com/blog/2019/04/22/implicit-function-declarations-flexs-use-of-reallocarray>

几个月前，我接手了 [Fedora](https://getfedora.org/) 中的 *flex* 包的维护工作，并决定通过 [Fedora Rawhide](https://fedoraproject.org/wiki/Releases/Rawhide) 中的*包来踢轮胎。我下载并散列了当时最新的 tarball， *flex-2.6.4* ，调整了 *spec* 文件，并启动了一个本地构建。不幸的是，它在构建时以一个`SIGSEGV`失败了:*

```
./stage1flex -o stage1scan.c ./scan.l
make[2]: *** [Makefile:1695: stage1scan.c] Segmentation fault (core dumped)

```

用 *gdb* 进行了一些调试，让我得出结论，分段错误是 flex 初始化期间从`reallocarray`函数返回的内存块被写入的结果。在本文中，我将进一步描述这个问题，并解释为解决这个问题所做的改变。

以下是我的 *gdb* 会话的简化片段:

```
(gdb) bt
#0 check_mul_overflow_size_t (right=1, left=2048, left@entry=0)
#1 __GI___libc_reallocarray (optr=0x0, nmemb=2048, elem_size=1)
#2 allocate_array at misc.c:147
#3 flexinit at main.c:974
#4 flex_main at main.c:168
#5 __libc_start_main
(gdb) fin
Run till exit from #0 check_mul_overflow_size_t
__GI___libc_reallocarray
33              return realloc (optr, bytes);
(gdb) fin
Run till exit from #0 __GI___libc_reallocarray
in allocate_array
147             mem = reallocarray(NULL, (size_t) size, element_size);
Value returned is $1 = (void *) 0x5555557c6420
(gdb) fin
Run till exit from #0 allocate_array
in flexinit
974             action_array = allocate_character_array (action_size);
Value returned is $2 = (void *) 0x557c6420
(gdb) n
975             defs1_offset = prolog_offset = action_offset = action_index = 0;
(gdb) n
976             action_array[0] = '\0';
(gdb) n
Program received signal SIGSEGV, Segmentation fault.

```

在 segfault 发生之前，我没有注意到这里有任何异常，但也许你已经注意到了。我看到的所有*是返回的指针在第`974`行为*非空*，但是在第`976`行写入它导致了 segfault。它开始看起来像一个`malloc`错误。*

我一时兴起，在 Fedora 构建系统之外构建了相同的 tarball。这一次，典型的`./configure && make`命令行在构建时没有 segfault。所以很明显区别在于 *rpmbuild* 使用的构建选项。一些试验和错误引导我找到了原因:`-pie`，产生位置独立可执行文件的链接器标志。带有`-pie`的建筑造成了分割断层。

有了这个“复制者”和我在 Red Hat 的同事的建议，我开始对 flex 源代码做了一个*git-等分*。 *HEAD* 在上游 *master* 分支上干净利落地构建，即使有`-pie`也是如此，所以问题只是找到修复构建的提交。有问题的提交是对针对 flex upstream 报告的以下问题的修复:

[# 241:“C99 中函数 reallocarray 的隐式声明无效”](https://github.com/westes/flex/issues/241)

所以，flex 源代码没有声明`_GNU_SOURCE`，导致编译器看不到 *reallocarray* 函数的声明。在这种情况下，编译器用默认返回类型(`int`)创建一个隐式函数声明，并相应地生成代码。在 64 位英特尔机器上， *int* 类型只有 32 位宽，而指针是 64 位宽。回过头来看看 gdb 会话，我清楚地看到指针被截断了:

```
147             mem = reallocarray(NULL, (size_t) size, element_size);
Value returned is $1 = (void *) 0x5555557c6420
(gdb) fin
Run till exit from #0  allocate_array
in flexinit
974             action_array = allocate_character_array (action_size);
Value returned is $2 = (void *) 0x557c6420

```

这种情况只发生在位置独立的可执行文件中，因为堆被映射到地址空间的一部分，那里的指针大于`INT_MAX`，暴露了上面的 flex bug。GCC 实际上通过`-Wimplicit-function-declaration`选项警告了隐式函数声明的存在。似乎最近有一个[提议在 Fedora 版本中启用这个警告](https://fedoraproject.org/wiki/Changes/Fedora26CFlags)，但是最终被搁置了。如果启用，警告仍然会导致 flex 构建失败——但是会更早，而且是在问题很明显的时候。

在这一点上，让构建成功编译是一件简单的事情，即反向移植定义了 *_GNU_SOURCE* 并向编译器公开了 *reallocarray* 原型的相应 flex 补丁。

但我们并没有就此止步。我的一个同事，Florian Weimer——glibc 的定期撰稿人——认为如果 glibc 通过更通用的`_DEFAULT_SOURCE`功能测试宏来公开 *reallocarray* ,这一切都可以避免。这个改变现在已经[提交](https://sourceware.org/git/?p=glibc.git;a=commit;h=2bda273aa3)到 glibc upstream，并且从 *glibc-2.29* 开始可用。

通过这一改变，我们希望在 Fedora 和 glibc 用户社区的其他组件中避免类似的情况。glibc 现在提供了 *reallocarray* 函数原型，除非用户明确要求更严格地符合给定的标准。

*Last updated: April 17, 2019*