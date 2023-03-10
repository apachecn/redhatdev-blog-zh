# 如何在程序中存储大量数据

> 原文：<https://developers.redhat.com/blog/2019/07/05/how-to-store-large-amounts-of-data-in-a-program>

大多数程序需要数据才能运行。有时这些数据是在程序运行时提供给程序的，有时这些数据是内置在程序中的。在本文中，我将解释如何在程序中存储大量数据，以便在程序运行时这些数据就在那里。

存储数据最明显的方法是将其包含在程序的源代码中。例如，在 C 语言中:

```
int a = 1;
```

这种方法适用于少量数据，但是随着要存储的数据量的增加，它很快变得很麻烦。此外，如果要以这种方式存储数据，通常需要创建一个工具，将数据转换成所用编程语言可接受的形式。

下一个选择是在运行时加载数据。这是可行的，但也有问题。例如，它假设存在可用于存储数据文件的文件系统。这也意味着程序不再是一个单独的实体，而是必须与这些数据文件一起发布。此外，需要编写额外的代码来处理文件丢失或损坏的情况。

因此，本文介绍了一种将大型数据文件包含到可执行程序体中的方法。本文是在考虑基于 ELF 的 GNU/Linux 系统的情况下撰写的。其他操作系统可能有其他方法来解决这个问题。特别值得注意的是，Windows 支持程序的“资源”概念[1]，它提供对各种类型的嵌入式数据的只读访问。

## INCBIN 指令

方法是利用一个汇编源文件，或者甚至是内联汇编程序，以及名为`.incbin` [2]的特殊汇编伪操作。该指令允许在程序中的指定位置包含任意文件。例如:

```
.incbin "foo.jpg"
```

实际上，最好确保数据位于正确的部分，并且正确对齐。此外，可能需要符号来提供对高级源代码中数据的访问:

```
.data
.align 4
.global start_of_foo
start_of_foo:
.incbin "foo.jpg"
.global end_of_foo
end_of_foo:
```

然后可以在 C 源文件中访问它，如下所示:

```
extern char start_of_foo;
extern char end_of_foo;
char * p;

for (p = & start_of_foo; p < & end_of_foo; p++)
  ...
```

注意，在上面的代码片段中，地址操作符(&)的使用和指针类型(char *)的缺失是正确的。这是因为汇编程序创建的符号和编译器生成的符号之间的差异[3]。当汇编程序创建一个符号时，它真正做的只是提供一个对应于给定地址的标签。然而，当编译器创建一个符号时，它在程序的数据中创建一个空格，将一个值安装到该空格中，然后使用该符号作为对该值的间接引用。

C 语言确实允许将符号视为标签；但是，它们必须声明为未调整大小的数组:

```
extern char start_of_foo[];
extern char end_of_foo[];
char * p;

for (p = start_of_foo; p < end_of_foo; p++)
...
```

汇编代码将`foo.jpg`的内容放入程序的数据段，这意味着它可以被写入和读取。如果数据需要是只读的，那么它应该放在`.rodata`部分，就像这样:

```
.section .rodata
[...]
.incbin "foo.jpg"
[...]
```

事实上，可能需要将数据单独放在一个节中，这样就可以很容易地在生成的可执行文件中找到它。的。section 指令允许创建新的节，因此可以使用以下内容:

```
.section foo-image, "a" @progbits
```

`"a"`表示应该在程序的运行时内存映像中为该段分配空间。默认情况下，这个数据是只读的，所以如果它需要被写入，您可以添加`w`标志(即`"aw"`)。`@progbits`表示该部分只包含数据，没有其他内容。

使用这种方法要考虑的另一件事是，它改变了当前节，如果汇编程序被内联到更高级别的源代码中，这可能会导致问题。在这种情况下，`.pushsection`和`.popsection`伪操作可用于安全地改变截面，如下所示:

```
__asm__("\n\
    .pushsection .foo-image, \"a\", @progbits\n\
    .align 4\n\
    .global start_of_foo\n\
start_of_foo:\n\
    .incbin \"foo.jpg\"\n\
    .global end_of_foo\n\
end_of_foo:\n\
    .popsection\n");

```

将数据放入它自己的部分还有一个额外的好处。只要段名是有效的 C 标识符(意思是`foo_image`可以，但是`foo-image`不行)，那么链接器会自动为它创建开始和结束符号。因此，没有必要在汇编代码中声明它们。因此，下面的程序将打印出一个名为`foo.jpg`的文件的大小和内容，foo.jpg 被嵌入到可执行文件中:

```
int
main (void)
{
  extern const char __start_foo_image[];
  extern const char __stop_foo_image[];
  const char * p;

  __asm__("\n\
.pushsection foo_image, \"a\", @progbits\n\
.incbin \"foo.jpg\"\n\
.popsection\n");

  printf ("image size: %#lx\n", __stop_foo_image - __start_foo_image);

  for (p = __start_foo_image; p < __stop_foo_image; p++)
    printf ("%d ", *p);

  printf ("\n");
  return 0;
}
```

## 修改程序内数据

将数据存储在可执行文件中的一个问题是很难修改数据。重新编译始终是一个选项，但还有另一个选项。`objcopy`程序允许改变程序中各部分的内容。但是，请注意，它不允许编辑一个节中的单个字节，只允许批量替换一个节的内容。因此，这种方法只有在数据已经被放入它自己的部分时才有效。

命令[4]如下所示:

```
objcopy --update-section sectionname=filename <file>
```

因此，根据上面的例子，这个命令:

```
objcopy --update-section foo_image="bar.jpg" a.out
```

将用 bar.jpg 图像替换 a.out 中的 foo.jpg 图像。

然而，这种方法确实有一个重大缺陷；替换不会更改由汇编程序或链接程序生成的符号，编译后的代码仍将使用旧值。因此，如果新文件与旧文件的大小不同，那么停止/结束符号将是不正确的。开始符号仍然是 OK，因为它的值是相对于 foo_image 部分的开始的，它总是零。因此，这个故事的寓意是，除非数据是自描述的，否则不要用大小相等的块之外的任何东西来替换它。

## 结论

使用一点汇编程序技巧，在程序中存储大型数据集是可能的。将数据放入它自己的部分可以更容易地检查和修改(如果需要的话)。当然，这种方法确实会使程序变得更大，但是根据具体情况，它可能仍然比将数据存储在程序之外要好。

### 参考

[1][https://en . Wikipedia . org/wiki/Resource _(Windows)](https://en.wikipedia.org/wiki/Resource_(Windows))
【2】[https://sourceware . org/binutils/docs-2.32/as/Incbin . html # Incbin](https://sourceware.org/binutils/docs-2.32/as/Incbin.html#Incbin)
【3】[https://sourceware . org/binutils/docs-2.32/LD/Source-Code-Reference . html # Source-Code-Reference](https://sourceware.org/binutils/docs-2.32/ld/Source-Code-Reference.html#Source-Code-Reference)
【4】[https](https://sourceware.org/binutils/docs-2.32/binutils/objcopy.html#objcopy)

*Last updated: July 3, 2019*