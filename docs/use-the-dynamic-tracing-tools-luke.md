# 使用动态追踪工具，卢克

> 原文：<https://developers.redhat.com/blog/2018/05/11/use-the-dynamic-tracing-tools-luke>

追踪运行开源软件的计算机系统上的问题的一个常见说法是“使用源代码，Luke。”查看源代码有助于理解代码是如何工作的，但是静态视图可能无法让您完整地了解代码中的事情是如何工作的(或被破坏的)。代码中的路径严重依赖于数据。如果不知道代码中关键位置的具体值，您很容易忽略正在发生的事情。动态检测工具，如 SystemTap，可以跟踪和检测软件，有助于更全面地了解代码实际在做什么

我想更好地理解 Ruby 解释器是如何工作的。这是一个在 Red Hat Enterprise Linux 7 上使用 SystemTap 研究 Ruby MRI 内部的机会。文章[什么是 SystemTap，如何使用？](https://access.redhat.com/solutions/5441)有更多关于安装 SystemTap 的信息。x86_64 RHEL 7 机器安装了`ruby-2.0.0648-33.el7_4.x86_64.rpm`，所以安装了匹配的`debuginfo` RPM，为 SystemTap 提供函数参数的信息，为我提供人类可读的源代码。通过以 root 用户身份运行以下命令来安装`debuginfo` RPM:

```
# debuginfo-install ruby -y
```

我注意到的第一件事是 Ruby 命令`/usr/bin/ruby`非常小，不到 8K 字节:

```
$ ls -l /usr/bin/ruby
-rwxr-xr-x. 1 root root 7184 Feb 19 07:12 /usr/bin/ruby
```

实际的解释器必须位于别处。我浏览了一下`/usr/bin/ruby`使用的共享库，发现`libruby.so.2.0`很可能是 Ruby 解释器的家:

```
$ ldd /usr/bin/ruby
	linux-vdso.so.1 =>  (0x00007ffc11076000)
	libruby.so.2.0 => /lib64/libruby.so.2.0 (0x00007f8a62a1a000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f8a627fe000)
	librt.so.1 => /lib64/librt.so.1 (0x00007f8a625f6000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007f8a623f2000)
	libcrypt.so.1 => /lib64/libcrypt.so.1 (0x00007f8a621bb000)
	libm.so.6 => /lib64/libm.so.6 (0x00007f8a61eb9000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f8a61aec000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f8a62e77000)
	libfreebl3.so => /lib64/libfreebl3.so (0x00007f8a618e9000) 
```

我们可以使用通配符`"*"`匹配所有函数名，列出 SystemTap 可以探测的共享库中的所有函数:

```
$ stap -l 'process("/lib64/libruby.so.2.0").function("*")'
```

在 Ruby MRI 解释器中，SystemTap 可以看到超过 5600 个函数。检测所有这些功能可能会给我们带来信息过载。此外，SystemTap 会检查它的插装给代码带来了多少开销，如果导致了太多的开销，它就会停止。让我们缩小范围，只看 Ruby 的任意精度算术是如何工作的。下面显示了与`bignum`相关的 125 个不同的功能:

```
$ stap -l 'process("/lib64/libruby.so.2.0").function("*")'|grep "bignum"
```

下面是 Ruby MRI 共享库`/usr/lib64/libruby.so.2.0.0`中函数`rb_cstr_to_inum`的上面`stap -l`命令的单行输出。这一行在`@`符号后的函数名后面有附加信息:这个函数的源文件和行号。

```
process("/usr/lib64/libruby.so.2.0.0").function("rb_cstr_to_inum@/usr/src/debug/ruby-2.0.0-p648/bignum.c:579")
```

如前所述，Ruby `debuginfo`提供了`bignum.c`源文件。在编辑器中加载该文件，转到第 579 行，显示了该函数的开始:

```
VALUE
rb_cstr_to_inum(const char *str, int base, int badcheck)
{
const char *s = str;
char *end;
char sign = 1, nondigit = 0;
int c;
...
```

可以指定 SystemTap 通配符来将匹配限制为仅匹配`bignum.c`文件中的函数，如下所示:

```
$ stap -l 'process("/lib64/libruby.so.2.0").function("*@*/bignum.c")'
```

现在，有了这个可探测函数的集合，我们可以使用 SystemTap 示例脚本`/usr/share/systemtap/examples/general/para-callgraph.stp`中的`para-callgraph.stp`来查看任意精度算法是如何操作的。Ruby 源代码有一个示例程序`pi.rb`，它使用任意精度算法计算圆周率的位数。

```
$ more pi.rb
#!/usr/local/bin/ruby

k, a, b, a1, b1 = 2, 4, 1, 12, 4

loop do
  # Next approximation
  p, q, k = k*k, 2*k+1, k+1
  a, b, a1, b1 = a1, b1, p*a+q*a1, p*b+q*b1
  # Print common digits
  d = a / b
  d1 = a1 / b1
  while d == d1
    print d
    $stdout.flush
    a, a1 = 10*(a%b), 10*(a1%b1)
    d, d1 = a/b, a1/b1
  end
end
```

它将运行，打印出永无止境的圆周率数字流:

```
$ ruby pi.rb
3141592653589793238462643383279502884197169399375105820974944592
3078164062862089986280348253421170679821480865132823066470938446
0955058223172535940812848111745028410270193852110555964462294895
4930381964428810975665933446128475648233786783...
```

现在我们可以看到任意算术函数在程序执行过程中是如何操作的。下面命令的第一行用`para-callgraph.stp`脚本调用 SystemTap。第二行为脚本提供了关于要探测哪些函数的信息。在这种情况下，来自`bignum.c`源文件的所有函数都被跟踪。该命令的最后一行让 SystemTap 在创建完所有 SystemTap 工具后启动`ruby`程序。`pi.rb`程序将无休止地运行。一旦我们有了足够的跟踪数据，我们可以通过按 Ctrl-C 来停止程序和跟踪。

```
stap /usr/share/systemtap/examples/general/para-callgraph.stp \
'process("/lib64/libruby.so.2.0").function("*@*/bignum.c")' \
-c "ruby pi.rb"
```

下面是输出的开始。在我们看到调用`rb_big_resize_big`的`para-callgraph.stp`输出之前，大约打印出 25 个数字。这可能是缓冲输出的副作用，而不是事情发生的实际顺序。`para-callgraph.stp`的每一行都以前一行经过的微秒数、进程名和 PID 开始。`->`表示输入一个函数 function 及其参数值列表。`<-`是函数的返回值，如果它有返回值的话。嵌套函数调用是缩进的。

在程序跟踪的最开始，我们可以看到一些初始化，包括对`Init_Bignum`的调用。`Init_Bignum`函数的源代码显示了一系列使用`rb_define_method`函数设置类中各种方法的调用，该函数在`bignum.c`之外的文件中，没有被检测。因此，没有对`rb_define_method`函数的追踪。然而，在`Init_Bignum`的结尾有一个对`rb_uint2big`函数的调用，这可以在跟踪中看到。`rb_uint2big`源代码中有带两个参数的`bignew`，但是在跟踪中有带三个参数调用的`bignew_1`。搜索`bignum.c`显示了一个`define`，它将`bignew`转换成一个带有三个参数的对`bignew_1`的调用。

我们还看到一个字符串的转换，第一个`rb_cstr_to_inum`是基数 8 ( `0x8`)并返回`0x5`。下一次由`rb_cstr_to_inum`进行的从字符串到数字的转换是以 10 为基数(`0xa`)并返回`0x3`。源代码显示`rb_cstr_to_inum`从`bignorm`返回值，但是 trace 中没有`bignorm`的踪迹。编译器通过将`bignorm`内联到`rb_cstr_to_inum`中来优化代码，以消除函数调用的开销。`bignorm`(和`rb_cstr_to_inum`)返回的值用低位表示对象类型([Bignum 有多大？](http://patshaughnessy.net/2014/1/9/how-big-is-a-bignum))。在`rb_cstr_to_inum`中，对象设置了最低有效位，表示值是固定的数字。

```
31415926535897932384626433     0 ruby(6049):->rb_big_resize big=0x201bfb0 len=0x5
     5 ruby(6049):rb_big_norm x=0x201bfb0
     2 ruby(6049): ->bignorm x=0x201bfb0
     4 ruby(6049):  ->rb_big_resize big=0x201bfb0 len=0x4
     7 ruby(6049):  <-rb_big_resize 
     8 ruby(6049): <-bignorm return=0x201bfb0
     9 ruby(6049):Init_Bignum 
    25 ruby(6049): ->rb_uint2big n=0x3
    28 ruby(6049):  ->bignew_1 klass=0x20168a8 len=0x2 sign=0x1
    31 ruby(6049):  <-bignew_1 return=0x2016858
    33 ruby(6049): <-rb_uint2big return=0x2016858
    36 ruby(6049):rb_uint2big n=0xffffffffffffffff
     4 ruby(6049): ->bignew_1 klass=0x20168a8 len=0x2 sign=0x1
     6 ruby(6049): <-bignew_1 return=0x200b4a8
     8 ruby(6049):rb_cstr_to_inum str=0x2122390 base=0x8 badcheck=0x0
     6 ruby(6049):rb_cstr_to_inum str=0x2122390 base=0xa badcheck=0x0
     3 ruby(6049):<-rb_cstr_to_inum return=0x3 
```

早先对`Init_Bignum`函数代码的检查表明`/`(除法)方法是由`rb_big_div`函数执行的。因为`pi.rb`执行除法，所以应该有对`rb_big_div`的调用。稍后在跟踪输出中，有一个通过`rb_big_div`函数对任意精度算术进行除法运算的例子。它用参数`op=0x2f`调用`rb_big_divide`，一个 ASCII 字符`/`。该跟踪显示了在将值传递给`bigdivmod`之前，需要通过调用`rb_int2big`将一个参数转换成适当的类。`bigdivrem`和`bigdivrem1`功能执行实际的除法运算。除法运算完成后，由`bignorm`函数执行另一个标准化操作，以确定结果有多大，并适当调整存储大小。在这种特定情况下，值(`0xb`)具有低位设置，指示该值是固定数，数值编码在返回值的剩余位中。

```
 0 ruby(6049):->rb_big_div x=0x238abd8 y=0x63c25f1981ac4201
     3 ruby(6049): ->rb_big_divide x=0x238abd8 y=0x63c25f1981ac4201 op=0x2f
     6 ruby(6049):  ->rb_int2big n=0x31e12f8cc0d62100
     8 ruby(6049):   ->rb_uint2big n=0x31e12f8cc0d62100
    11 ruby(6049):    ->bignew_1 klass=0x20168a8 len=0x2 sign=0x1
    13 ruby(6049):    <-bignew_1 return=0x238abb0
    15 ruby(6049):   <-rb_uint2big return=0x238abb0
    16 ruby(6049):  bigdivmod x=0x238abd8 y=0x238abb0 divp=0x7fff57542938 modp=0x0
    22 ruby(6049):   ->bigdivrem x=0x238abd8 y=0x238abb0 divp=0x7fff57542938 modp=0x7fff575428e8
    24 ruby(6049):    ->bignew_1 klass=0x20168a8 len=0x4 sign=0x1
    27 ruby(6049):    rb_big_clone x=0x238abb0
    31 ruby(6049):     ->bignew_1 klass=0x20168a8 len=0x2 sign=0x1
    34 ruby(6049):     <-bignew_1 return=0x238ab60
    36 ruby(6049):    bigdivrem1 ptr=0x7fff57542860
    41 ruby(6049):    rb_big_clone x=0x238ab88
    45 ruby(6049):     ->bignew_1 klass=0x20168a8 len=0x4 sign=0x1
    47 ruby(6049):     <-bignew_1 return=0x238ab38
    48 ruby(6049):    rb_big_clone x=0x238ab88
    62 ruby(6049):     ->bignew_1 klass=0x20168a8 len=0x4 sign=0x1
    64 ruby(6049):     <-bignew_1 return=0x238ab10
    66 ruby(6049):    <-rb_big_clone return=0x238ab10
    67 ruby(6049):   <-bigdivrem return=0x238ab88
    69 ruby(6049):  bignorm x=0x238ab38
    82 ruby(6049):  <-bignorm return=0xb
    84 ruby(6049): <-rb_big_divide return=0xb
    85 ruby(6049):<-rb_big_div return=0xb
```

作为这个小练习的结果，我对 Ruby 解释器如何处理任意算术有了更好的理解。同样的技术可以用来跟踪 Ruby MRI(或任何其他复杂的软件)的其他部分的执行，以便更好地理解它是如何操作的。

*Last updated: May 10, 2018*