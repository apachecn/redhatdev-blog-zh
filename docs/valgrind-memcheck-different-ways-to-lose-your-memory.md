# Valgrind Memcheck:失去记忆的不同方式

> 原文：<https://developers.redhat.com/blog/2021/04/23/valgrind-memcheck-different-ways-to-lose-your-memory>

Valgrind 是一个工具框架，用于构建动态分析工具，检查 C 和 C++程序的错误。Memcheck 是默认工具 [Valgrind](https://valgrind.org/) 在你没有向它要求另一个工具时使用。您可以选择(使用`valgrind tool=*toolname*`)的其他有用工具有:

*   `cachegrind`和`callgrind`，进行缓存和调用图函数评测
*   `helgrind`和`drd`，进行线程错误和数据竞争检测
*   `massif`和`dhat`，进行动态堆使用分析

这些工具都值得单独写一篇文章，但是这里我们将集中讨论 Memcheck。

## 用 Valgrind Memcheck 检测内存泄漏

Memcheck 跟踪 C 或 C++程序中所有的内存读取、写入、分配和释放。该工具可以检测许多不同的内存错误。例如，它检测分配的内存块之前或之后的读取或写入。它警告在条件代码中使用(部分)未定义的值或将这样的值传递给系统调用。它还会通知您内存块的坏分配或双重分配。但是现在，我们将使用 Memcheck 讨论内存泄漏检测。

## 生成泄漏摘要

当您在没有任何附加参数的情况下在程序上运行 Valgrind 时，它会生成一个已检测到的不同类型泄漏的摘要。例如，`valgrind ./myprog`可能会生成以下摘要:

```
    LEAK SUMMARY:
      definitely lost: 48 bytes in 1 blocks
      indirectly lost: 24 bytes in 3 blocks
        possibly lost: 0 bytes in 0 blocks
      still reachable: 14 bytes in 1 blocks
           suppressed: 0 bytes in 0 blocks

```

Memcheck 将泄漏分为五类:明确丢失、间接丢失、可能丢失、仍可到达和被抑制。前四个类别表示在程序结束前没有被释放的不同种类的内存块。如果您对特定的块不感兴趣，您可以告诉 Valgrind 不要报告它们(您很快就会看到如何报告)。摘要还显示了丢失的字节数以及它们在多少个块中，这告诉您是丢失了许多小的分配，还是丢失了一些大的分配。

以下部分解释了每个类别。

### 肯定输了

第一类，肯定是丢失的，通常是最需要追踪的泄漏类型，因为没有办法使用或恢复这些内存。让我们看一个简单地调用几次`output_report`的小程序的例子。该函数每次打印一个小横幅和一个数字。正如我们将看到的，当程序结束时，我们为报告横幅保留的内存肯定会丢失(多次):

```
 #include <stdlib.h>
 #include <stdio.h>
 #include <string.h>

 char *
 create_banner ()
 {
   const char *user = getenv ("USER");
   size_t len = 1 + 2 * 4 + strlen (user) + 1;
   char *b = malloc (len);
   sprintf (b, "\t|** %s **|", user);
   return b;
 }

 void
 output_report (int nr)
 {
   char *banner = create_banner ();
   puts (banner);
   printf ("Number: %d\n", nr);
   printf ("\n");
 }

 int
 main ()
 {
   for (int i = 1; i <= 3; i++)
     output_report (i);

   return 0;
 }

```

用`gcc -Wall -g -o definitely definitely.c`编译这段代码，在 Valgrind 下运行，用`valgrind --leak-check=full ./definitely`询问细节。现在，在泄漏总结之前，Valgrind 将显示程序分配了最终丢失的内存的回溯:

```
 42 bytes in 3 blocks are definitely lost in loss record 1 of 1
    at 0x4C29F33: malloc (vg_replace_malloc.c:309)
    by 0x4011C7: create_banner (definitely.c:10)
    by 0x401200: output_report (definitely.c:18)
    by 0x40124C: main (definitely.c:28)

```

请注意，Memcheck 发现了三个泄漏，它将其报告为一个丢失记录，因为它们具有相同的回溯。默认情况下，它要求整个回溯是相同的，以便考虑足够相似的泄漏来一起报告。如果您想让 Memcheck 组合更多的泄漏，您可以使用`--leak-resolution=low`或`--leak-resolution=med`对只有两个或四个共同回溯条目的泄漏进行分组。如果 Memcheck 报告了大量的泄漏，而您怀疑这些泄漏可能是同一个问题，那么这是非常有用的。然后，您可以专注于丢失字节(或块)数量最多的记录。

### 仍然可以到达

在前面的例子中，很明显我们应该在使用后释放`banner`。我们可以在`output_report`函数的末尾添加`free (banner)`来实现。然后再在 Valgrind 下运行的时候，它会开心的说`All heap blocks were freed -- no leaks are possible`。

但是我们很聪明，看到代码为每个报告重用了相同的标题。因此，我们在代码中将`banner`定义为静态顶级变量，并将`create_banner`调用移动到`main`函数，这样`create_banner`只被调用一次:

```
char *banner;

void
output_report (int nr)
{
  puts (banner);
  printf ("Number: %d\n", nr);
  printf ("\n");
}

int
main
{
  banner = create_banner ();
  for (int i = 1; i <= 3; i++)
    output_report (i);

  return 0;
}

```

请注意，我们再次忘记调用`free`，这次是在`main`的末尾。现在，当在 Valgrind 下运行时，Memcheck 将报告`still reachable: 14 bytes in 1 blocks`和任何其他类别的零字节丢失。

但是输出没有提供丢失记录的详细信息，即使我们用`--leak-check=full`运行，也没有回溯到仍然可以到达的内存块。这是因为 Memcheck 认为错误不是很严重。内存仍然可以访问，所以程序可能还在使用它。理论上，你可以在程序结束时释放它，但是所有的内存在程序结束时都会被释放。

尽管理论上仍然可以达到的内存并不是一个真正的问题，但是您可能仍然想要研究它。您可能想看看是否可以更早地释放给定的块，这可能会降低长时间运行的程序的内存使用。还是因为你真的喜欢看那句话`All heap blocks were freed -- no leaks are possible`。要获得您需要的细节，请在 Valgrind 命令行中添加`--show-leak-kinds=reachable`或`--show-leak-kinds=all`(连同`--leak-check=full`)。现在，您还将获得回溯，显示程序中仍可到达的内存块的分配位置。

### 可能丢失

为了探索其他类别的泄漏，我们稍微修改了一下程序，添加了一些要报告的数字列表。每个报告将有一个不同的报告数字列表。完整的数据结构是在程序开始时分配的。对于每组数字，我们分配一组新的数字。为了简单起见(Memcheck 会指出这太简单了)，我们只保留一个指向当前要打印的 numbers 结构的指针。虽然我们创建了三组数字，但我们只输出两个报告:

```
#include <stdlib.h>
#include <stdio.h>

struct numbers
{
  int n;
  int *nums;
};

int n;
struct numbers *numbers;

void
create_numbers (struct numbers **nrs, int *n)
{
  *n = 3;
  *nrs = malloc ((sizeof (struct numbers) * 3));
  struct numbers *nm = *nrs;
  for (int i = 0; i < 3; i++)
    {
      nm->n = i + 1;
      nm->nums = malloc (sizeof (int) * (i + 1));
      for (int j = 0; j < i + 1; j++)
        nm->nums[j] = i + j;
      nm++;
    }
}

void
output_report ()
{ 
  puts ("numbers"); 
  for (int i = 0; i < numbers->n; i++)
    printf ("Number: %d\n", numbers->nums[i]);
  printf ("\n");
}

int
main ()
{ 
  create_numbers (&numbers, &n);
  for (int i = 0; i < 2; i++)
    {
      output_report ();
      numbers++;
    }
  return 0;
}

```

当我们用`gcc -Wall -g -o possibly possibly.c`编译这个程序，然后用`valgrind --leak-check=full ./possibly`在 Valgrind 下运行时，Valgrind 报告`possibly lost: 72 bytes in 4 blocks`。因为我们用`--leak-check=full`运行，它也报告回溯:

```
 24 bytes in 3 blocks are possibly lost in loss record 1 of 2
    at 0x4C29F33: malloc (vg_replace_malloc.c:309)
    by 0x4011C3: create_numbers (possibly.c:22)
    by 0x40128F: main (possibly.c:41)

 48 bytes in 1 blocks are possibly lost in loss record 2 of 2
    at 0x4C29F33: malloc (vg_replace_malloc.c:309)
    by 0x401185: create_numbers (possibly.c:17)
    by 0x40128F: main (possibly.c:41)

```

Memcheck 调用这个内存*可能丢失了*，因为它仍然可以看到如何访问内存块。`numbers`指针指向第三组数字。如果我们保留一些额外的信息，理论上我们可以倒计数到这个内存块的开始并访问剩余的信息，或者释放整个内存块和它所指向的其他内存。

但是 Memcheck 认为这很可能是一个错误。在我们的例子中，正如在大多数这样的情况下，Memcheck 是正确的。当遍历一个数据结构而不保留对该结构本身的引用时，我们永远无法重用或释放该结构。我们应该使用`numbers`指针作为基础，并使用一个(数组)索引来传递当前记录作为`output_report (&numbers[i])`。然后，Memcheck 会报告数据块仍然可以到达。(仍然存在内存泄漏，但并不严重，因为有一个指向内存的直接指针，它很容易被释放。)

### 间接损失

在前面的例子中，Memcheck 报告了一个可能丢失的块，因为`numbers`指针仍然指向一个已分配的块。我们可能想通过简单地在调用`output_report`后清除指针来解决这个问题，方法是通过做`numbers = NULL;`来指示没有当前的号码列表要报告。但是我们也失去了指向内存数据块的最后一个指针。我们应该先释放内存，但是我们现在不能这样做，因为我们不再有指向数据结构开始的指针:

```
int
main ()
{
  create_numbers (&numbers, &n);
  for (int i = 0; i < 2; i++)
    {
      output_report ();
      numbers++;
    }
  numbers = NULL;
  return 0;
}

```

现在，Memcheck 将报告内存肯定丢失。因为内存块包含指向其他内存块的指针，所以这些内存块被报告为间接丢失。如果我们用`--leak-check=full`运行，我们会看到主 numbers 内存块的回溯:

```
 72 (48 direct, 24 indirect) bytes in 1 blocks are definitely lost in loss record 2 of 2
    at 0x4C29F33: malloc (vg_replace_malloc.c:309)
    by 0x401185: create_numbers (possibly.c:17)
    by 0x40128F: main (possibly.c:41)

 LEAK SUMMARY:
    definitely lost: 48 bytes in 1 blocks
    indirectly lost: 24 bytes in 3 blocks
      possibly lost: 0 bytes in 0 blocks
    still reachable: 0 bytes in 0 blocks
         suppressed: 0 bytes in 0 blocks

```

请注意，间接丢失的数据块没有回溯。这是因为 Memcheck 相信当你修复了肯定丢失的块时，你可能会修复它。如果您确实释放了明确丢失的块，但没有释放间接指向的内存块，那么下次在 Valgrind 下运行部分修复的程序时，Memcheck 会将这些间接丢失的块报告为明确丢失(现在带有回溯)。因此，通过迭代地修复明确丢失的内存泄漏，您将最终修复所有间接丢失的内存泄漏。

如果您不能立即找到导致某些间接丢失块的明确丢失的块，查看创建间接丢失块的回溯可能会有所帮助。当使用`--leak-check=full`时，你可以通过在`valgrind`命令行中添加`--show-leak-kinds=reachable`或`--show-leak-kinds=all`来实现。

### 抑制

默认情况下，Memcheck 用`--leak-check=full`将肯定丢失和可能丢失的块计为错误。它还将显示这些块的分配位置。默认情况下，它不会将间接丢失的块或仍可到达的丢失块视为错误。并且它不会显示那些仍然可到达或间接丢失的块被分配到哪里的回溯，除非使用`--show-leak-kinds=all`明确要求这样做。

当你解决了明确丢失的问题时，间接丢失的块将会消失(或变成明确丢失的块)。没有明确丢失的块，就不会有间接丢失的块。对于可到达的块，为了降低程序的内存使用，查看是否可以提前释放它们可能仍然是有意义的。或者在程序结束时显式地释放它们，以确保所有的内存都被真正地计算和清理了。

但是不修复所有的内存泄漏可能是有原因的。它们可能出现在您正在使用的不容易被替换的库中。或者你可能相信一个可能丢失的块并不是一个真正的错误。如果在最初的肯定丢失的例子中，您决定不修复这个问题并保持内存泄漏，那么您可能希望生成一个抑制，这样 Memcheck 就不会再抱怨这个特定的块了。您可以通过运行生成示例抑制的`valgrind --leak-check=full --gen-suppressions=all ./definitely`轻松做到这一点:

```
{
   *insert_a_suppression_name_here*
   Memcheck:Leak
   match-leak-kinds: definite
   fun:malloc
   fun:create_banner
   fun:output_report
   fun:main
}

```

你可以把它放到一个文件中(比如说，`local.supp`)，用一些描述性的东西代替`*insert_a_suppression_name_here*`，比如`small leak in create_banner`。现在，当您运行`valgrind --suppressions=./local.supp --leak-check=full ./definitely`时，泄漏将被抑制:

```
 LEAK SUMMARY:
    definitely lost: 0 bytes in 0 blocks
    indirectly lost: 0 bytes in 0 blocks
      possibly lost: 0 bytes in 0 blocks
    still reachable: 0 bytes in 0 blocks
         suppressed: 42 bytes in 3 blocks

```

任何被抑制的块都不会再有输出。但是如果您想查看使用了哪些抑制，您可以将`--show-error-list=yes`(或`-s`)添加到`valgrind`命令行。该选项使 Valgrind 显示抑制名称、抑制文件、行号以及该抑制规则抑制了多少字节和块:

```
used_suppression:
  1 small leak in create_banner ./local.supp:2 suppressed: 42 bytes in 3 blocks

```

## 测试套件集成

当您解决了所有内存泄漏问题，或者当您抑制了那些您不关心的问题时，您可能希望将 Valgrind 集成到您的测试套件中，以便尽早捕捉任何新的内存泄漏。如果使用`--error-exitcode=<number>`，当检测到错误(内存泄漏)时，Valgrind 会将程序的退出代码更改为给定的数字。你也可以用`--quiet`(或者`-q`)让 Valgrind 静音，这样就不会干扰程序正常的`stdout`和`stderr`，除了错误输出，这样你就可以照常比较程序输出了。

请记住，默认情况下，Memcheck 只将肯定丢失和可能丢失的内存块视为错误。你可以通过使用`--errors-for-leak-kinds=*set*`来改变它。如果您只对明确丢失的块感兴趣，您可以使用`--errors-for-leak-kinds=definite`。当你的测试程序总是释放所有的内存块，包括仍然可以到达的内存块，你可以使用`--errors-for-leak-kinds=definite,possibly,reachable`或者`--errors-for-leak-kinds=all`。注意，`--errors-for-leak-kinds=*set*`与`--error-exitcode=*number*`一起工作，上面提到的`--show-leak-kinds=*set*`选项决定显示哪些回溯，它们是独立的。但是一般来说，你会希望它们是相同的，这样你就可以得到一个内存错误的回溯。

因此，运行测试的一个好方法是`valgrind -q --error-exitcode=99 --leak-check=full ./testprog`。如果有什么局部抑制，可以加上`--suppressions=local.supp`。如果你真的希望你所有的测试用例完全没有任何类型的内存泄漏，添加`--show-leak-kinds=all --errors-for-leak-kinds=all`。

*Last updated: April 22, 2021*