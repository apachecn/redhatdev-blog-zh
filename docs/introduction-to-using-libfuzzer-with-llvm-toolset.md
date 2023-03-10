# 将 libFuzzer 与 llvm 工具集一起使用的介绍

> 原文：<https://developers.redhat.com/blog/2019/03/05/introduction-to-using-libfuzzer-with-llvm-toolset>

“模糊化”应用程序是找到其他测试方法可能遗漏的 bug 的好方法。Fuzzers 通过生成随机字符串输入并将它们输入到应用程序中来测试程序。任何接受用户任意输入的程序都很容易被模糊化。这包括编译器、解释器、web 应用程序、JSON 或 YAML 解析器，以及更多类型的程序。

libFuzzer 是一个帮助应用程序和库模糊化的库。它被集成到 Clang C 编译器中，可以通过添加一个编译标志和在代码中添加一个模糊目标来为您的应用程序启用。libFuzzer 已经被成功地用于发现许多[程序](https://llvm.org/docs/LibFuzzer.html#trophies)中的错误，在本文中，我将展示如何将 libFuzzer 集成到您自己的应用程序中。

要在[Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview/)(RHEL)7 上开始使用 libFuzzer，您需要安装`llvm-toolset-6.0`包，它是 LLVM 工具集软件集合的一部分。LLVM 工具集包括 Clang。要安装 LLVM 工具集，您必须首先启用几个附加的存储库:

```
$ sudo subscription-manager repos --enable rhel-7-server-optional-rpms \
    --enable rhel-server-rhscl-7-rpms \
    --enable rhel-7-server-devtools-rpms
```

(参见 *[如果您的系统上没有设置`sudo`，如何在 RHEL](https://developers.redhat.com/blog/2018/08/15/how-to-enable-sudo-on-rhel/)* 上启用 sudo。)

接下来，安装 llvm-toolset-6.0:

```
$ sudo yum install llvm-toolset-6.0
```

因为 LLVM 工具集是作为 Red Hat 软件集合(RHSCL)交付的，所以您需要使用`scl enable`来启动一个新的 shell，并将 llvm-toolset-6.0 集合添加到您的路径中。

```
$ scl enable llvm-toolset-6.0 bash
```

或者，您可以将`llvm-toolset-6.0`收藏永久添加到您的个人资料中。更多信息参见文章 [*如何在红帽企业版 Linux 7*](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/) 上安装 Clang/LLVM 6 和 GCC 8。

## 让我们开始起毛

我们将从模糊一个简单的 C 函数开始，该函数返回单词中的第一个大写字母:

```
#include 

char get_first_cap(const char *in, int size) {
  const char *first_cap = NULL;

  if (size == 0)
    return ' ';
  for ( ; *in != 0; in++) {
    if (*in >= 'A' && *in <= 'Z') {
      first_cap = in;
      break;
    }
  }
  return *first_cap;
}

int LLVMFuzzerTestOneInput(const char *Data, long long Size) {
  get_first_cap(Data, Size);
  return 0;
}

```

在这个 C 文件中，我们有一个想要测试的函数(`get_first_cap`)和一个目标函数(`LLVMFuzzerTestOneInput`)，fuzzer 将调用该函数将其输入传递给函数。

现在我们可以使用`clang`编译这个函数来创建一个模糊的二进制文件:

```
$ clang -g -fsanitize=fuzzer first-cap.c -o fuzz-first-cap

```

有了`-fsantize=fuzzer`标志，`clang`会自动将我们的程序链接到 fuzzer 库，其中包括它自己的`main`函数。我们现在有了一个可执行文件`fuzz-first-cap`，我们可以用它来模糊`get_first_cap`函数。

如果我们不带参数运行我们的`fuzz-first-cap`程序，libFuzzer 将生成随机输入来测试我们的程序。我们还可以提供一个法律输入语料库，帮助 libFuzzer 更智能地处理它生成的输入类型。

```
$ mkdir corpus
$ echo "Apple" > corpus/Apple.txt
$ echo "aPple" > corpus/aPple.txt
$ echo "apPle" > corpus/apPle.txt

```

现在，如果我们用这个语料库运行我们的程序，我们将看到 libFuzzer 马上识别出一个问题(注意，我们只使用了`-seed=1`选项来获得可再现的输出；这是可选的):

```
$ ./fuzz-first-cap -seed=1 corpus

INFO: Seed: 1
INFO: Loaded 1 modules   (8 inline 8-bit counters): 8 [0x670fa0, 0x670fa8), 
INFO: Loaded 1 PC tables (8 PCs): 8 [0x45fd48,0x45fdc8), 
INFO:        3 files found in corpus
INFO: -max_len is not provided; libFuzzer will not generate inputs larger than 4096 bytes
INFO: seed corpus: files: 3 min: 6b max: 6b total: 18b rss: 33Mb
#4      INITED cov: 5 ft: 7 corp: 2/12b exec/s: 0 rss: 34Mb
#6      REDUCE cov: 5 ft: 7 corp: 2/10b exec/s: 0 rss: 34Mb L: 4/6 MS: 2 ChangeBinInt-EraseBytes-
UndefinedBehaviorSanitizer:DEADLYSIGNAL
==15554==ERROR: UndefinedBehaviorSanitizer: SEGV on unknown address 0x000000000000 (pc 0x00000045473a bp 0x7ffc2eacb4d0 sp 0x7ffc2eacb4a0 T15554)
==15554==The signal is caused by a READ memory access.
==15554==Hint: address points to the zero page.
    #0 0x454739 in get_first_cap /first-cap.c:13:11
    #1 0x4547b6 in LLVMFuzzerTestOneInput /first-cap.c:17:3
    #2 0x415b99 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) (fuzz-first-cap+0x415b99)
    #3 0x418954 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool*) (fuzz-first-cap+0x418954)
    #4 0x41a1f7 in fuzzer::Fuzzer::MutateAndTestOne() (fuzz-first-cap+0x41a1f7)
    #5 0x41a9af in fuzzer::Fuzzer::Loop(std::vector<std::string, fuzzer::fuzzer_allocator > const&) (fuzz-first-cap+0x41a9af)
    #6 0x410193 in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) (fuzz-first-cap+0x410193)
    #7 0x406562 in main (fuzz-first-cap+0x406562)
    #8 0x7fcb5a8a53d4 in __libc_start_main (/lib64/libc.so.6+0x223d4)
    #9 0x4065aa in _start (fuzz-first-cap+0x4065aa)

UndefinedBehaviorSanitizer can not provide additional info.
==15554==ABORTING
MS: 4 ShuffleBytes-ShuffleBytes-ChangeBit-EraseBytes-; base unit: 7469c22975699536c6c6d00767e773b5429fefc6
0x65,0x0,
e\x00
artifact_prefix='./'; Test unit written to ./crash-36282fac116d9fd6b37cc425310e1a8510f08a53
Base64: ZQA=

```

输出中最相关的部分是堆栈跟踪，它向我们显示存在分段错误，然后是导致崩溃的输入信息，这些信息出现在输出的末尾。在这种情况下，我们在一个没有大写字母的 2 字节输入上崩溃:`e, \x00`。

我们的语料库目录中还添加了一个新文件:

```
$ cat corpus/7469c22975699536c6c6d00767e773b5429fefc6
apP

```

这是 libFuzzer 在模糊我们的程序时生成的一个“好”输入。libFuzzer 将把它找到的所有好的输入添加到语料库目录中。

因此，让我们修复这个错误，然后再试一次:

```
#include 

char get_first_cap(const char *in, int size) {
  const char *first_cap = NULL;

  if (size == 0)
    return ' ';
  for ( ; *in != 0; in++) {
    if (*in >= 'A' && *in <= 'Z') {
      first_cap = in;
      break;
    }
  }
  if (first_cap)
    return *first_cap;
  else
    return ' ';
}

int LLVMFuzzerTestOneInput(const char *Data, long long Size) {
  get_first_cap(Data, Size);
  return 0;
}

```

然后重新编译:

```
$ clang -g -fsanitize=fuzzer first-cap.c -o fuzz-first-cap

```

并运行:

```
$ ./fuzz-first-cap  -seed=1 corpus

```

这一次，libFuzzer 在运行了大约 30 秒后没有发现任何问题。默认情况下，libFuzzer 会一直运行，直到它发现一个 bug，但是您可以使用标志`-runs=X`来配置它。

到目前为止，我们的 fuzzer 已经用于检测分段错误，但是您也可以在没有任何`clang`杀毒程序的情况下配对它来检查其他类型的错误。例如，我们可以在启用地址杀毒程序的情况下编译我们的程序:

```
clang -g -fsanitize=fuzzer,address first-cap.c -o fuzz-first-cap

```

现在当我们运行程序时，我们看到一个新的错误:

```
==15569==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6020000000f1 at pc 0x000000558507 bp 0x7fff78272c30 sp 0x7fff78272c28
READ of size 1 at 0x6020000000f1 thread T0
    #0 0x558506 in get_first_cap /first-cap.c:8:11
    #1 0x558766 in LLVMFuzzerTestOneInput /first-cap.c:21:3
    #2 0x42cea9 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) (fuzz-first-cap+0x42cea9)
    #3 0x42fc64 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool*) (fuzz-first-cap+0x42fc64)
    #4 0x4317df in fuzzer::Fuzzer::ReadAndExecuteSeedCorpora(std::vector<std::string, fuzzer::fuzzer_allocator > const&) (fuzz-first-cap+0x4317df)
    #5 0x431b72 in fuzzer::Fuzzer::Loop(std::vector<std::string, fuzzer::fuzzer_allocator > const&) (fuzz-first-cap+0x431b72)
    #6 0x4274a3 in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) (fuzz-first-cap+0x4274a3)
    #7 0x41d852 in main (fuzz-first-cap+0x41d852)
    #8 0x7f88abaca3d4 in __libc_start_main (/lib64/libc.so.6+0x223d4)
    #9 0x41d8bb in _start (fuzz-first-cap+0x41d8bb)
...
```

这里 fuzzer 触发了堆缓冲区溢出，地址杀毒程序捕获了该溢出。在这种情况下，输入是一个没有空终止符的字符串，它捕获了我们程序中的一个错误，我们假设输入将以空终止。

除了地址杀毒器之外，你还可以将 libFuzzer 与 LLVM 的未定义行为杀毒器(UBSAN)一起使用。

除了这个简单的介绍中所展示的，您还可以用 libFuzzer 做更多的事情。更多信息参见 [libFuzzer 文档](https://llvm.org/docs/LibFuzzer.html)。

## 相关文章

*   [Clang/LLVM 6.0、Go 1.10 和 Rust 1.29 现已在 RHEL 正式上市](https://developers.redhat.com/blog/2018/11/13/clang-llvm-6-0-go-1-10-and-rust-1-29-now-ga-for-rhel/)
*   [支持 Clang/LLVM、Go 和 Rust 的生命周期](https://developers.redhat.com/blog/2018/11/20/support-lifecycle-for-clang-llvm-go-and-rust/)
*   [GCC 未定义行为杀毒软件——ubsan](https://developers.redhat.com/blog/2014/10/16/gcc-undefined-behavior-sanitizer-ubsan/)
*   [用 GCC 8 检测字符串截断](https://developers.redhat.com/blog/2018/05/24/detecting-string-truncation-with-gcc-8/)
*   [GCC 8 中的可用性改进](https://developers.redhat.com/blog/2018/03/15/gcc-8-usability-improvements/)

*Last updated: March 8, 2019*