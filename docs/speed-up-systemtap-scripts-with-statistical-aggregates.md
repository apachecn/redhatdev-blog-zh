# 使用统计聚合加速 SystemTap 脚本

> 原文：<https://developers.redhat.com/blog/2019/04/10/speed-up-systemtap-scripts-with-statistical-aggregates>

SystemTap 可以用来回答的一个常见问题是系统中特定事件发生的频率。一些事件，比如系统调用，可能会频繁发生，我们的目标是尽可能提高 SystemTap 脚本的效率。

使用统计聚合代替常规整数是提高 SystemTap 脚本性能的一种方法。统计聚合以每个 CPU 为基础记录数据，以减少处理器之间所需的协调量，从而以更少的开销记录信息。在本文中，我将展示一个如何减少 SystemTap 脚本开销的例子。

下面是一个版本的`syscalls_by_proc.stp`示例脚本，它记录了每个可执行文件进行系统调用的次数。当脚本退出时，它打印出系统上每个可执行文件进行系统调用的次数，按系统调用次数从多到少排序。

第 1 行定义了用于存储信息的全局数组。第 3 行是使用`++`实际记录每个系统调用的探针。从第 5 行开始的探测端打印出存储在全局 syscalls 关联数组中的数据。

```
global syscalls

probe kernel.trace("sys_enter") { syscalls[execname()] ++ }

probe end {
  printf ("%-10s %-s\n", "#SysCalls", "Process Name")
  foreach (proc in syscalls-)
    printf("%-10d %-s\n", syscalls[proc], proc)
}
```

下面的代码示例显示了上一个脚本的运行。`-v`选项使 SystemTap 为五次传递中的每一次传递提供计时信息。`-t`为脚本中的每个探测器提供计时信息，并提供一种方法来比较不同实现的效率。一旦 SystemTap 检测准备就绪，`-c "make -j8"`就启动 make 命令，并在命令完成后停止检测。输出的最后是“探测命中报告”，它提供了各种探测的开销信息。

`kernel.trace("raw_syscalls:sys_enter")`的中线显示它被触发了超过 2400 万次。同一行的中间表示平均花费了 4224 个时钟周期来完成该操作。在“刷新报告”部分的全局关联数组`(__global_syscall)`上也发生了超过 300 万个锁争用。

```
$ stap -v -t ~/present/2019blog/fast/syscalls_by_proc.stp -c "make -j8"
Pass 1: parsed user script and 504 library scripts using 290680virt/85620res/3552shr/82780data kb, in 590usr/30sys/621real ms.
Pass 2: analyzed script: 2 probes, 1 function, 0 embeds, 1 global using 296352virt/92244res/4484shr/88452data kb, in 130usr/190sys/331real ms.
Pass 3: using cached /home/wcohen/.systemtap/cache/b8/stap_b8412b9e49934149b789a69c3e2a2b4e_1469.c
Pass 4: using cached /home/wcohen/.systemtap/cache/b8/stap_b8412b9e49934149b789a69c3e2a2b4e_1469.ko
Pass 5: starting run.
  DESCEND  objtool
  CALL    scripts/atomic/check-atomics.sh
  CALL    scripts/checksyscalls.sh
  CHK     include/generated/compile.h
  TEST    posttest
  Building modules, stage 2.
  MODPOST 3484 modules
arch/x86/tools/insn_decoder_test: success: Decoded and checked 5057175 instructions
  TEST    posttest
arch/x86/tools/insn_sanity: Success: decoded and checked 1000000 random instructions with 0 errors (seed:0x18e55059)
Kernel: arch/x86/boot/bzImage is ready  (#9)
#SysCalls  Process Name
22614118   make
1675055    sh
138647     awk
112797     modpost
93456      cat
69563      objdump
68771      insn_decoder_te
49887      grep
14656      cc1
13544      gcc
11714      rm
10174      as
...
----- probe hit report: 
kernel.trace("raw_syscalls:sys_enter"), (/home/wcohen/present/2019blog/fast/syscalls_by_proc.stp:17:1), hits: 24894191, cycles: 310min/4224avg/477851max, variance: 5282267, from: kernel.trace("sys_enter"), index: 0
end, (/home/wcohen/present/2019blog/fast/syscalls_by_proc.stp:21:1), hits: 1, cycles: 109027min/109027avg/109027max, variance: 0, from: end, index: 1
----- refresh report:
'__global_syscalls' lock contention occurred 3230491 times
Pass 5: run completed in 174200usr/100470sys/69766real ms.

```

通过使用统计集合，可以显著降低脚本的开销。用一个`<<< 1`代替了`++`来记录每个系统调用，记录现在在下面的代码中用一个`@sum(syscalls[proc])`打印出来。

```
global syscalls

probe kernel.trace("sys_enter") { syscalls[execname()] &lt;&lt;&lt; 1 }

probe end {
  printf ("%-10s %-s\n", "#SysCalls", "Process Name")
  foreach (proc in syscalls-)
    printf("%-10d %-s\n", @sum(syscalls[proc]), proc)
}

```

下面是使用统计聚合代替较慢的`++`的脚本的等效运行。查看清单末尾的“探测命中报告”,您会看到,`raw_syscalls:sys_enter`跟踪点的运行次数与其他脚本相似，大约为 2500 万次。但是，请注意，处理程序所需的平均时间是 409 个时钟周期，远远低于在带有`++`的版本上观察到的 4224 个周期。对于使用统计聚合的运行，在“刷新报告”一节中列出的 syscalls 关联数组也没有列出锁争用。

```
$ stap -v -t ~/present/2019blog/faster/syscalls_by_proc.stp -c "make -j8"
Pass 1: parsed user script and 504 library scripts using 290680virt/85624res/3552shr/82780data kb, in 580usr/30sys/616real ms.
Pass 2: analyzed script: 2 probes, 1 function, 0 embeds, 1 global using 296348virt/92256res/4492shr/88448data kb, in 140usr/190sys/332real ms.
Pass 3: using cached /home/wcohen/.systemtap/cache/de/stap_de5d5dc935de72aac43603205a17cad4_1482.c
Pass 4: using cached /home/wcohen/.systemtap/cache/de/stap_de5d5dc935de72aac43603205a17cad4_1482.ko
Pass 5: starting run.
  DESCEND  objtool
  CALL    scripts/atomic/check-atomics.sh
  CALL    scripts/checksyscalls.sh
  CHK     include/generated/compile.h
  TEST    posttest
  Building modules, stage 2.
  MODPOST 3484 modules
arch/x86/tools/insn_decoder_test: success: Decoded and checked 5057175 instructions
  TEST    posttest
arch/x86/tools/insn_sanity: Success: decoded and checked 1000000 random instructions with 0 errors (seed:0xbefec711)
Kernel: arch/x86/boot/bzImage is ready  (#9)
#SysCalls  Process Name
22616433   make
1675055    sh
138647     awk
112797     modpost
93456      cat
69563      objdump
68771      insn_decoder_te
49891      grep
14706      cc1
13544      gcc
11714      rm
10174      as
...
----- probe hit report:
kernel.trace("raw_syscalls:sys_enter"), (/home/wcohen/present/2019blog/faster/syscalls_by_proc.stp:17:1), hits: 24895995, cycles: 216min/409avg/395696max, variance: 3425671, from: kernel.trace("sys_enter"), index: 0
end, (/home/wcohen/present/2019blog/faster/syscalls_by_proc.stp:19:1), hits: 1, cycles: 847759min/847759avg/847759max, variance: 0, from: end, index: 1
----- refresh report:
Pass 5: run completed in 172780usr/66010sys/64215real ms.

```

使用统计集合的缺点是，当使用`@sum()`时，需要从机器中的所有处理器获取数据。这比仅仅获取存储在关联数组中的单个整数值更昂贵。然而，对于这个例子，减少系统调用探针的开销远远超过了用于打印结果的`@sum()`的开销。

在编写 SystemTap 脚本时，您可以使用`-t`选项来更好地理解脚本的开销，并在可行时考虑使用统计集合。如本例所示，在 SystemTap 脚本中，插装脚本的开销可以显著降低。