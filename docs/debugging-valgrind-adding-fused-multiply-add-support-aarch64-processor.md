# 使用 Valgrind 的- trace-flags 选项

> 原文：<https://developers.redhat.com/articles/2021/06/15/debugging-valgrind-adding-fused-multiply-add-support-aarch64-processor>

[Valgrind](https://valgrind.org/) 是一个很棒的工具，不仅可以查找程序中与内存管理相关的错误，还可以进行内存消耗分析、性能分析、多线程相关问题等等。在这篇文章中，我介绍了 Valgrind 未记录的`--trace-flags`选项，并解释了我们如何在与 Arm 的 [AArch64 处理器](https://developer.arm.com/documentation/102374/0101)相关的一个方面提高 Valgrind 的准确性。

## 舍入误差的例子

Valgrind 是应用程序和操作系统之间的抽象层。它反汇编应用程序的代码，并根据使用的 Valgrind 工具向其中添加工具。为了执行和分析内存或寄存器操作，Valgrind 解析指令并将其翻译成一种称为 VEX 的中间表示(IR)。例如，Valgrind 的前端将应用程序汇编语言代码中的 ADD 指令翻译成`Iop_Add` IR。Valgrind 工具(memcheck、helgrind 等)对 IR 进行检测，然后 Valgrind 后端重新汇编代码。

虽然 Valgrind 的翻译过程非常复杂，但是效果很好。但有时它确实会犯错误。例如，在 Valgrind 下运行的以下简单的 [C 语言](/topics/c)代码显示了某些浮点运算的舍入不精确(本错误报告中记录了[):](https://bugs.kde.org/show_bug.cgi?id=426014)

```
int main()

double x = 1004.3;
double y = 2.0;
double r = pow(x, y);

printf("r = %.10f\n", r); return 0;
```

在 AArch64 上正确编译后，这段代码应该打印出`r`的值为 100。49660 . 68668686661 然而，当在 Valgrind 下运行时，它打印了 10000000001 原因是 Valgrind 缺乏对 AArch64 融合乘加(FMADD)指令的正确支持，该指令在`pow`函数中使用，从而导致舍入问题。

## 融合乘法加法

两个操作数之间的乘积相加很常见，足以在许多处理器上获得一条特殊的指令。FMADD 代表[浮点融合乘加](https://developer.arm.com/documentation/dui0801/f/A64-Floating-point-Instructions/FMADD)。基本操作是:

```
D(destination) = A(accumulator) + N * M
```

该指令有 32 位(浮点)和 64 位(双精度)两种变体。

舍入问题的出现是因为一次做+ N * M 与先做(N * M)然后再加 A 会产生稍微不同的结果。

不同的处理器使用不同的指令来识别这种操作的流行。与 AArch64 一样，PowerPC ppc64 和 IBM s390x 也有标量 FMADD。但是大多数其他架构只有类似于 FMADD 的向量指令。例如，英特尔 x86 提供了一个 VFMADD 指令，它类似于 AArch64 的 FMADD，但它是一个向量(单指令/多数据，或 SIMD)指令。向量寄存器是存储几个数字的大型寄存器，允许同时对它们进行操作。例如，英特尔的 AVX-512 处理器使用 512 位寄存器。标量寄存器要小得多，通常为 32 或 64 位长，包含一个标量值。

因为 Valgrind 需要支持 ppc64 和 s390x 的标量融合乘加指令，它已经为它定义了一个称为`Iop_MAddF32`的 IR。这个 VEX IR 操作代表一个 32 位浮点融合乘加指令。但是 Valgrind 的 arm64 前端和后端还没有实现。我工作的一个团队创建了一个[补丁](https://bugsfiles.kde.org/attachment.cgi?id=134093)，为`Iop_MAdd/SubF32/64`增加了 Arm64 VEX 前端和后端支持。在添加这个补丁之前，FMADD 在 VEX 中被实现为两个 IRs，`Iop_Add`和`Iop_Mul`，来表示一个实际的指令。这导致了舍入误差。

补丁的前端部分用一个`Iop_Madd` IR 代替了`Iop_Add`和`Iop_Mul`的使用，这让 Valgrind 避免了舍入误差。然后后端再次将 IR 转换成实际的指令。

为了测试我们的 Valgrind 补丁，我们编写了汇编语言代码，以确保 Valgrind 在应该的时候生成`Iop_MAdd` IR 指令。如果一个架构支持标量 FMA 指令，编译器将有希望把类似于`x = a + (b *c)`的东西变成一个高效的 FMADD 指令，而不是一个乘法然后一个加法指令。但是直接使用内联汇编更容易:

```
asm("fmadd %s0, %s1, %s2, %s3\n;" : "=w"(dst) : "w"(x), "w"(y), "w"(z));
```

这里，`s`是 32 位 SIMD/浮点寄存器的名称，在本例中用作浮点(FP)寄存器。

下面是一个可以用命令`gcc -g -o tst test.c`编译的最小测试:

```
int
main(int argc, char **argv)
{
float x = 55;
float y = 0.69314718055994529;
float z = 38.123094930796988;
float dst;
//32bit variant
asm("fmadd %s0, %s1, %s2, %s3\n;" : "=w"(dst) : "w"(x), "w"(y), "w"(z));
printf("%f = %f + %f * %f\n", dst, z, x, y);

return 0;
}
```

## - trace-flags 选项

为了精确计算 Valgrind 做什么，它的`--trace-flags`选项非常有用。这个选项可以帮助您发现 Valgrind 代码中有问题的地方，对于那些想知道 Valgrind 在应用程序中到底在处理什么的专家用户来说也很有用。

Valgrind 手册页中没有记载`--trace-flags `选项，它也不会与`valgrind --help`一起显示。然而，你可以通过运行`valgrind --help-debug`在每个标志中看到它提供的选项。表 1 显示了这些标志及其效果。

Table 1: Flags in Valgrind's --trace-flags option.

| 旗 | 影响 |
| Ten million | 显示到 IR 的转换 |
| 01000000 | 初始选择后显示 |
| 00100000 | 检测后显示 |
| 00010000 | 第二次选择后显示 |
| 0000 1000 | 树构建后显示 |
| 00000100 | 显示选择 insns |
| 00000010 | 在注册分配后显示 |
| 00000001 | 显示最终装配 |
| 00000000(所有位清零) | 仅显示概要文件 |

**注意**:要从`--trace-flags`获得全部细节，还需要指定`--trace-notbelow`或`--trace-notabove`。

有了这些值，您可以看到 Valgrind 和相关检测工具执行的所有转换。但是这里我们只对第一个“拆卸”和最后的“组装”步骤感兴趣。我们接下来将探讨这些。

### trace-flags 如何工作

在这里，我将描述如何一步一步地使用`--trace-flags`选项，以我们关注的 FMADD 活动的跟踪为例。

Valgrind 的第一步，也就是转换成 IR，是独立于工具的。但对于下一步，显示最终装配，它有助于没有任何工具做仪器，使最终装配更清晰。在我们的例子中，我们并不太关心工具和它所做的优化。因此，我添加了`--tool=none`选项，这样就没有工具(默认为 memcheck)添加自己的指令。生成的命令是:

```
$ ./vg-in-place -q --tool=none --trace-flags=10000000 --trace-notbelow=999999 ./tst 2>&1 | less
```

该命令产生了许多我们不感兴趣的代码块。与我们相关的模块是`./tst`模块中的主要功能。为了找到相关的块，我们重新运行之前的命令，用之前运行调用`main`时显示的 SB(超级块)号替换`--trace-notbelow=999999`中的任意值:

```
SB 1237 (evchecks 6200) [tid 1] 0x400634 main /root/valgrind/tst+0x400634
```

`main`的 SB 号是 1237。我们用这个数字来跳过`main`之前的所有超级块。因此，我们的新命令是:

```
$ ./vg-in-place -q --tool=none --trace-flags=10000000 --trace-notbelow=1237 ./tst 2>&1 | less
```

我们希望在块的输出中寻找与我们相关的`fmadd`。补丁情况之前的`fmadd`相关块曾经是:

```
(arm64) 0x400670: fmadd s0, s0, s1, s2

------ IMark(0x400670, 4, 0) ------
t18 = Shr32(GET:I32(888),0x16:I8)
t19 = Or32(And32(Shl32(t18,0x1:I8),0x2:I32),And32(Shr32(t18,0x1:I8),0x1:I32))
t17 = AddF32(t19,GET:F32(352),MulF32(t19,GET:F32(320),GET:F32(336)))
PUT(320) = V128{0x0000}
PUT(320) = t17
PUT(272) = 0x400674:I64
```

在 FMADD 支持下，输出更改为:

```
(arm64) 0x400670: fmadd s0, s0, s1, s2
------ IMark(0x400670, 4, 0) ------
t18 = Shr32(GET:I32(888),0x16:I8)
t19 = Or32(And32(Shl32(t18,0x1:I8),0x2:I32),And32(Shr32(t18,0x1:I8),0x1:I32))
t17 = MAddF32(t19,GET:F32(320),GET:F32(336),GET:F32(352))
PUT(320) = V128{0x0000}
PUT(320) = t17
PUT(272) = 0x400674:I64 
```

比较贴片应用前后的实际添加说明。之前，添加的内容是:

```
t17 = AddF32(t19,GET:F32(352),MulF32(t19,GET:F32(320),GET:F32(336)))
```

应用补丁后，对应的代码是:

```
t17 = MAddF32(t19,GET:F32(320),GET:F32(336),GET:F32(352))
```

该轨迹向我们显示，根据需要，使用了`MAddF32`而不是`AddF32`和`MulF32`。

### 带有- trace-flags 选项的汇编代码

如前所述，跟踪和配置文件控制对于查看最终的汇编语言代码非常有用。对于这个任务，我们将使用值`10000111`中的四个标志:

```
$ ./vg-in-place --tool=none --trace-flags=10000111 --trace-notbelow=1237 -q ./tst 2>&1 | less
```

查看`MAddF32,`的输出，我们可以看到以下汇编代码:

```
-- t79 =
MAddF32(Or32(And32(Shl32(t72,0x1:I8),0x2:I32),And32(Shr32(t72,0x1:I8),0x1:I32)),GET:F32(320),GET:F32(336),GET:F32(352))
ldr %vD128(S-reg), 320(x21)
ldr %vD129(S-reg), 336(x21)
ldr %vD130(S-reg), 352(x21)

...

msr fpcr, %vR139
ffmadd %vD131(S-reg), %vD128(S-reg), %vD129(S-reg), %vD130(S-reg)
mov(d) %vD79, %vD131
```

`R`指通用寄存器，`D`指 SIMD/浮点寄存器。这个代码片段的倒数第二行显示寄存器`D128`到`131`被加载并用于`fmadd`。我们可以看看之前在 VEX IR 中看到的`MAddF32`指令:

```
t17 = MAddF32(t19,GET:F32(320),GET:F32(336),GET:F32(352)) 
```

并将其与生成的汇编代码进行比较:

```
ldr %vD128(S-reg), 320(x21)
ldr %vD129(S-reg), 336(x21)
ldr %vD130(S-reg), 352(x21)
ffmadd %vD131(S-reg), %vD128(S-reg), %vD129(S-reg), %vD130(S-reg) 
```

例如，比较告诉我们，加载到`D128` SIMD 寄存器的第一个参数`GET:F32(320)`成为了`fmadd`中的第二个操作数。这在我们的调试过程中非常有帮助，因为它揭示了操作数或它们的顺序是错误的。这里的例子展示了`--trace-flags`选项的信息量和细粒度。我们可以查看实际发出的指令，而不必关心寄存器分配，或者我们可以稍后查看实际生成的汇编代码。

## 结论

我希望这篇文章能帮助你更好地理解 Valgrind 是如何工作的，开发者是如何改进它的，以及你如何使用`--trace-flags`来精确地发现你的程序和 Valgrind 在底层做什么。

*Last updated: August 15, 2022*