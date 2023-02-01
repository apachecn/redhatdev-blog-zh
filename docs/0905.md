# GCC 9 中的可用性改进

> 原文：<https://developers.redhat.com/blog/2019/03/08/usability-improvements-in-gcc-9>

我在 Red Hat 的 GNU 编译器集合 GCC 上工作，去年我花了大部分时间让 GCC 更容易使用。让我们看看 [C](https://developers.redhat.com/blog/category/c/) 和 [C++](https://developers.redhat.com/blog/category/c-plus-plus/) 的改进，它们将出现在 GCC 的下一个主要版本中， [GCC 9](https://gcc.gnu.org/gcc-9/changes.html) 。

## 诊断学的新面貌

举例来说，让我们看看 GCC 8 如何报告试图在 C++中使用缺失的二进制“+”:

```
$ gcc-8 t.cc
t.cc: In function ‘int test(const shape&, const shape&)’:
t.cc:15:4: error: no match for ‘operator+’ (operand types are ‘boxed_value<double>’ and ‘boxed_value<double>’)
   return (width(s1) * height(s1)
           ~~~~~~~~~~~~~~~~~~~~~~
  + width(s2) * height(s2));
    ^~~~~~~~~~~~~~~~~~~~~~~~

```

下面是 GCC 9 中的样子:

```
$ gcc-9 t.cc
t.cc: In function ‘int test(const shape&, const shape&)’:
t.cc:15:4: error: no match for ‘operator+’ (operand types are ‘boxed_value<double>’ and ‘boxed_value<double>’)
   14 |   return (width(s1) * height(s1)
      |           ~~~~~~~~~~~~~~~~~~~~~~
      |                     |
      |                     boxed_value<[...]>
   15 |    + width(s2) * height(s2));
      |    ^ ~~~~~~~~~~~~~~~~~~~~~~
      |                |
      |                boxed_value<[...]>

```

这里有一些变化。我添加了一个左边的空白，显示行号。“error”行提到了第 15 行，但是这个表达式跨越了多行，我们实际上是从第 14 行开始的。我认为这是值得的一点额外的水平空间，以明确哪一行是哪一行。它还有助于将您的源代码与 GCC 发出的注释区分开来。我相信，通过在最左边的栏中直观地分解事物，它们也使每个诊断从哪里开始变得更容易一些。

说到注释，这个例子展示了 GCC 9 的另一个新特性:诊断可以标记源代码区域以显示相关信息。这里，最重要的是“+”操作符左边和右边的类型，所以 GCC 内联突出显示它们。请注意，诊断还使用颜色来区分两个操作数以及操作符。

左边距影响我们如何打印一些东西，比如缺失头文件的修复提示:

```
$ gcc-9 -xc++ -c incomplete.c
incomplete.c:1:6: error: ‘string’ in namespace ‘std’ does not name a type
    1 | std::string test(void)
      |      ^~~~~~
incomplete.c:1:1: note: ‘std::string’ is defined in header ‘<string>’; did you forget to ‘#include <string>’?
  +++ |+#include <string>
    1 | std::string test(void)

```

默认情况下，我已经打开了这些更改；它们可以分别通过[-fno-诊断-显示-线号](https://gcc.gnu.org/onlinedocs/gcc/Diagnostic-Message-Formatting-Options.html#index-fno-diagnostics-show-line-numbers)和[-fno-诊断-显示-标签](https://gcc.gnu.org/onlinedocs/gcc/Diagnostic-Message-Formatting-Options.html#index-fno-diagnostics-show-labels)禁用。

另一个例子可以在我去年写的文章“GCC 8 中的可用性改进”中的类型不匹配错误中看到:

```
extern int callee(int one, const char *two, float three);

int caller(int first, int second, float third)
{
  return callee(first, second, third);
}

```

其中表达式的伪类型现在以内联方式突出显示:

```
$ gcc-9 -c param-type-mismatch.c
param-type-mismatch.c: In function ‘caller’:
param-type-mismatch.c:5:24: warning: passing argument 2 of ‘callee’ makes pointer from integer without a cast [-Wint-conversion]
    5 |   return callee(first, second, third);
      |                        ^~~~~~
      |                        |
      |                        int
param-type-mismatch.c:1:40: note: expected ‘const char *’ but argument is of type ‘int’
    1 | extern int callee(int one, const char *two, float three);
      |                            ~~~~~~~~~~~~^~~

```

在这个糟糕的`printf`调用中可以看到另一个例子:

```
$ g++-9 -c bad-printf.cc -Wall
bad-printf.cc: In function ‘void print_field(const char*, float, long int, long int)’:
bad-printf.cc:6:17: warning: field width specifier ‘*’ expects argument of type ‘int’, but argument 3 has type ‘long int’ [-Wformat=]
    6 |   printf ("%s: %*ld ", fieldname, column - width, value);
      |                ~^~~               ~~~~~~~~~~~~~~
      |                 |                        |
      |                 int                      long int
bad-printf.cc:6:19: warning: format ‘%ld’ expects argument of type ‘long int’, but argument 4 has type ‘double’ [-Wformat=]
    6 |   printf ("%s: %*ld ", fieldname, column - width, value);
      |                ~~~^                               ~~~~~
      |                   |                               |
      |                   long int                        double
      |                %*f

```

它将格式字符串所期望的类型与传入的类型进行“内联”对比。(令人尴尬的是，在旧版本的 C++前端中，我们没有正确地突出格式字符串的位置；对于 GCC 9，我已经实现了这一点，所以它与 C 前端的奇偶校验相同，如下所示)。

## 不仅仅是人类

在改变 GCC 打印诊断信息的方式时，我听到的一个担心是，它可能会破坏某人解析 GCC 输出的脚本。我不认为这些改变会做到这一点:大多数这样的脚本都是为了解析

```
  "FILENAME:LINE:COL: error: MESSAGE"

```

行，忽略其余部分，我不会接触输出的这一部分。

但是这让我觉得是时候为诊断提供机器可读的输出格式了，所以对于 GCC 9，我添加了一个 JSON 输出格式:[-fdiagnostics-format = JSON](https://gcc.gnu.org/onlinedocs/gcc/Diagnostic-Message-Formatting-Options.html#index-fdiagnostics-format)。

考虑这个警告:

```
$ gcc-9 -c cve-2014-1266.c -Wall
cve-2014-1266.c: In function ‘SSLVerifySignedServerKeyExchange’:
cve-2014-1266.c:629:2: warning: this ‘if’ clause does not guard... [-Wmisleading-indentation]
  629 |  if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
      |  ^~
cve-2014-1266.c:631:3: note: ...this statement, but the latter is misleadingly indented as if it were guarded by the ‘if’
  631 |   goto fail;
      |   ^~~~

```

使用`-fdiagnostics-format=json`，诊断作为一个大的 JSON blob 发送到 stderr。通过方便的`python -m json.tool`对它们进行格式化，可以了解其结构:

```
$ (gcc-9 -c cve-2014-1266.c -Wall -fdiagnostics-format=json 2>&1) | python -m json.tool | pygmentize -l json
[
    {
        "children": [
            {
                "kind": "note",
                "locations": [
                    {
                        "caret": {
                            "column": 3,
                            "file": "cve-2014-1266.c",
                            "line": 631
                        },
                        "finish": {
                            "column": 6,
                            "file": "cve-2014-1266.c",
                            "line": 631
                        }
                    }
                ],
                "message": "...this statement, but the latter is misleadingly indented as if it were guarded by the \u2018if\u2019"
            }
        ],
        "kind": "warning",
        "locations": [
            {
                "caret": {
                    "column": 2,
                    "file": "cve-2014-1266.c",
                    "line": 629
                },
                "finish": {
                    "column": 3,
                    "file": "cve-2014-1266.c",
                    "line": 629
                }
            }
        ],
        "message": "this \u2018if\u2019 clause does not guard...",
        "option": "-Wmisleading-indentation"
    }
]

```

特别是，补充的“注释”嵌套在 JSON 级别的“警告”中，例如，允许 ide 对它们进行分组。我们的一些 C++诊断可能有许多子诊断提供额外的细节，因此能够对它们进行分组，例如，通过一个 disclosure 小部件，可能会有所帮助。

## 更简单的 C++错误

C++是一种复杂的语言。例如，确定在调用点调用哪个 C++函数的规则是[非平凡的](https://en.cppreference.com/w/cpp/language/overload_resolution)。

编译器可能需要在给定的调用点考虑几个函数，出于不同的原因拒绝所有的函数，而`g++`的错误消息必须处理这种一般性，解释为什么每个函数都被拒绝。

这种通用性会使简单的情况变得更加难以理解，所以对于 GCC 9，我添加了特殊情况来简化一些常见情况下的`g++`错误，在这些情况下只有一个候选函数。

例如，GCC 8 可以发出这样的消息:

```
$ g++-8 param-type-mismatch.cc
param-type-mismatch.cc: In function ‘int test(int, const char*, float)’:
param-type-mismatch.cc:8:45: error: no matching function for call to ‘foo::member_1(int&, const char*&, float&)’
   return foo::member_1 (first, second, third);
                                             ^
param-type-mismatch.cc:3:14: note: candidate: ‘static int foo::member_1(int, const char**, float)’
   static int member_1 (int one, const char **two, float three);
              ^~~~~~~~
param-type-mismatch.cc:3:14: note:   no known conversion for argument 2 from ‘const char*’ to ‘const char**’

```

对于 GCC 9，我对此进行了特殊处理，给出了一个更直接的错误消息，它突出显示了有问题的参数和它不能转换成的参数:

```
$ g++-9 param-type-mismatch.cc
param-type-mismatch.cc: In function ‘int test(int, const char*, float)’:
param-type-mismatch.cc:8:32: error: cannot convert ‘const char*’ to ‘const char**’
    8 |   return foo::member_1 (first, second, third);
      |                                ^~~~~~
      |                                |
      |                                const char*
param-type-mismatch.cc:3:46: note:   initializing argument 2 of ‘static int foo::member_1(int, const char**, float)’
    3 |   static int member_1 (int one, const char **two, float three);
      |                                 ~~~~~~~~~~~~~^~~

```

类似地，GCC 8 接收了两条消息，为各种拼写错误的姓名提供建议:

```
$ g++-8 typo.cc
typo.cc:5:13: error: ‘BUFSIZE’ was not declared in this scope
 uint8_t buf[BUFSIZE];
             ^~~~~~~
typo.cc:5:13: note: suggested alternative: ‘BUF_SIZE’
 uint8_t buf[BUFSIZE];
             ^~~~~~~
             BUF_SIZE

```

对于 GCC 9，我整合了这些信息:

```
$ g++-9 typo.cc
typo.cc:5:13: error: ‘BUFSIZE’ was not declared in this scope; did you mean ‘BUF_SIZE’?
    5 | uint8_t buf[BUFSIZE];
      |             ^~~~~~~
      |             BUF_SIZE

```

在某些情况下，GCC 8 知道在名称空间中提供建议:

```
$ g++-8 typo-2.cc
typo-2.cc: In function ‘void mesh_to_strip()’:
typo-2.cc:8:3: error: ‘tri_strip’ was not declared in this scope
   tri_strip result;
   ^~~~~~~~~
typo-2.cc:8:3: note: suggested alternative:
typo-2.cc:2:9: note:   ‘engine::tri_strip’
   class tri_strip {
         ^~~~~~~~~

```

GCC 9 现在可以提供修复提示:

```
$ g++-9 typo-2.cc
typo-2.cc: In function ‘void mesh_to_strip()’:
typo-2.cc:8:3: error: ‘tri_strip’ was not declared in this scope; did you mean ‘engine::tri_strip’?
    8 |   tri_strip result;
      |   ^~~~~~~~~
      |   engine::tri_strip
typo-2.cc:2:9: note: ‘engine::tri_strip’ declared here
    2 |   class tri_strip {
      |         ^~~~~~~~~

```

## 位置，位置，位置

GCC 内部表示的一个长期存在的问题是，语法树中的每个节点都有一个源位置。

对于 GCC 8，我[添加了一个方法](https://github.com/gcc-mirror/gcc/commit/d76863c8a62920c5a156125e68ad315b47bfcd24)来确保 C++调用点的每个参数都有一个源位置。

对于 GCC 9，我已经[扩展了这项工作](https://github.com/gcc-mirror/gcc/commit/d582d14011fec247f203a49e79bdab05f56197b0),这样 C++语法树中更多的地方现在可以更长时间地保留位置信息。

这在追踪错误的初始化时非常有用。GCC 8 和更早版本可能会在最后一个右括号或大括号中发出错误，例如:

```
$ g++-8 bad-inits.cc
bad-inits.cc:12:1: error: cannot convert ‘json’ to ‘int’ in initialization
 };
 ^
bad-inits.cc:14:47: error: initializer-string for array of chars is too long [-fpermissive]
 char buffers[3][5] = { "red", "green", "blue" };
                                               ^
bad-inits.cc: In constructor ‘X::X()’:
bad-inits.cc:17:35: error: invalid conversion from ‘int’ to ‘void*’ [-fpermissive]
   X() : one(42), two(42), three(42)
                                   ^

```

而现在，GCC 9 可以准确地指出各种问题所在:

```
$ g++-9 bad-inits.cc
bad-inits.cc:10:14: error: cannot convert ‘json’ to ‘int’ in initialization
   10 |   { 3, json::object },
      |        ~~~~~~^~~~~~
      |              |
      |              json
bad-inits.cc:14:31: error: initializer-string for array of chars is too long [-fpermissive]
   14 | char buffers[3][5] = { "red", "green", "blue" };
      |                               ^~~~~~~
bad-inits.cc: In constructor ‘X::X()’:
bad-inits.cc:17:13: error: invalid conversion from ‘int’ to ‘void*’ [-fpermissive]
   17 |   X() : one(42), two(42), three(42)
      |             ^~
      |             |
      |             int

```

## 优化器在做什么？

GCC 可以自动“矢量化”循环，重新组织它们以同时进行多次迭代，从而利用 CPU 上的向量单元。然而，它只能对某些循环做到这一点；如果偏离了路径，GCC 将不得不使用标量代码来代替。

不幸的是，从历史上看，从 GCC 那里了解它在优化代码时所做的决策并不容易。我们有一个选项， [-fopt-info](https://gcc.gnu.org/onlinedocs/gcc/Developer-Options.html#index-fopt-info) ，它发出优化信息，但它更多的是为 GCC 本身的开发者提供的工具，而不是针对最终用户的。

例如，考虑这个(人为的)例子:

```
#define N 1024

void test (int *p, int *q)
{
  int i;

  for (i = 0; i < N; i++)
    {
      p[i] = q[i];
      asm volatile ("" ::: "memory");
    }
}

```

我试着用 GCC 8 和`-O3 -fopt-info-all-vec`来编译它，但它并不太有启发性:

```
$ gcc-8 -c v.c -O3 -fopt-info-all-vec

Analyzing loop at v.c:7
v.c:7:3: note: ===== analyze_loop_nest =====
v.c:7:3: note: === vect_analyze_loop_form ===
v.c:7:3: note: === get_loop_niters ===
v.c:7:3: note: not vectorized: loop contains function calls or data references that cannot be analyzed
v.c:3:6: note: vectorized 0 loops in function.
v.c:3:6: note: ===vect_slp_analyze_bb===
v.c:3:6: note: ===vect_slp_analyze_bb===
v.c:10:7: note: === vect_analyze_data_refs ===
v.c:10:7: note: got vectype for stmt: _5 = *_3;
vector(4) int
v.c:10:7: note: got vectype for stmt: *_4 = _5;
vector(4) int
v.c:10:7: note: === vect_analyze_data_ref_accesses ===
v.c:10:7: note: not consecutive access _5 = *_3;
v.c:10:7: note: not consecutive access *_4 = _5;
v.c:10:7: note: not vectorized: no grouped stores in basic block.
v.c:7:3: note: === vect_analyze_data_refs ===
v.c:7:3: note: not vectorized: not enough data-refs in basic block.
v.c:7:3: note: ===vect_slp_analyze_bb===
v.c:7:3: note: ===vect_slp_analyze_bb===
v.c:12:1: note: === vect_analyze_data_refs ===
v.c:12:1: note: not vectorized: not enough data-refs in basic block.

```

对于 GCC 9，我在矢量器中重新组织了问题跟踪，因此输出的形式如下:

```
  [LOOP-LOCATION]: couldn't vectorize this loop
  [PROBLEM-LOCATION]: because of [REASON]

```

对于上面的例子，这给出了下面的内容，确定了向量器不能处理的循环中的结构的位置。(我希望它也显示源代码，但这并没有使功能冻结):

```
$ gcc-9 -c v.c -O3 -fopt-info-all-vec
v.c:7:3: missed: couldn't vectorize loop
v.c:10:7: missed: statement clobbers memory: __asm__ __volatile__("" :  :  : "memory");
v.c:3:6: note: vectorized 0 loops in function.
v.c:10:7: missed: statement clobbers memory: __asm__ __volatile__("" :  :  : "memory");

```

这改进了一些东西，但仍然有一些限制，所以对于 GCC 9，我还添加了一个新选项来发出机器可读的优化信息:[-f save-optimization-record](https://gcc.gnu.org/onlinedocs/gcc/Developer-Options.html#index-fsave-optimization-record)。

这会写出一个包含更丰富数据的`SRCFILE.opt-record.json.gz`文件:例如，每条消息都标记有概要信息(如果有的话)，这样您就可以查看代码中“最热”的部分，它会捕获内联信息，这样如果一个函数被内联到几个地方，您就可以看到该函数的每个实例是如何被优化的。

## 其他改进

GCC 可以发出“修复提示”,建议如何修复代码中的问题。这些可以由 IDE 自动应用。

对于 GCC 9，我添加了各种新的修复提示。现在有一些修复提示，用于忘记各种 C++操作符所需的`return *this;`:

```
$ g++-9 -c operator.cc
operator.cc: In member function ‘boxed_ptr& boxed_ptr::operator=(const boxed_ptr&)’:
operator.cc:7:3: warning: no return statement in function returning non-void [-Wreturn-type]
    6 |     m_ptr = other.m_ptr;
  +++ |+ return *this;
    7 |   }
      |   ^

```

当编译器需要一个`typename`时:

```
$ g++-9 -c template.cc
template.cc:3:3: error: need ‘typename’ before ‘Traits::type’ because ‘Traits’ is a dependent scope
    3 |   Traits::type type;
      |   ^~~~~~
      |   typename 

```

当您试图将访问器成员当作数据成员使用时:

```
$ g++-9 -c fncall.cc
fncall.cc: In function ‘void hangman(const mystring&)’:
fncall.cc:12:11: error: invalid use of member function ‘int mystring::get_length() const’ (did you forget the ‘()’ ?)
   12 |   if (str.get_length > 0)
      |       ~~~~^~~~~~~~~~
      |                     ()

```

对于 C++11 的作用域枚举:

```
$ g++-9 -c enums.cc
enums.cc: In function ‘void json::test(const json::value&)’:
enums.cc:12:26: error: ‘STRING’ was not declared in this scope; did you mean ‘json::kind::STRING’?
   12 |     if (v.get_kind () == STRING)
      |                          ^~~~~~
      |                          json::kind::STRING
enums.cc:3:44: note: ‘json::kind::STRING’ declared here
    3 |   enum class kind { OBJECT, ARRAY, NUMBER, STRING, TRUE, FALSE, NULL_ };
      |                                            ^~~~~~

```

我添加了一个调整，将关于拼写错误的成员的建议与关于访问者的建议集成在一起:

```
$ g++-9 -c accessor-fixit.cc
accessor-fixit.cc: In function ‘int test(t*)’:
accessor-fixit.cc:17:15: error: ‘class t’ has no member named ‘ratio’; did you mean ‘int t::m_ratio’? (accessible via ‘int t::get_ratio() const’)
   17 |   return ptr->ratio;
      |               ^~~~~
      |               get_ratio()

```

我还调整了建议代码，使其考虑了换位的字母，因此它应该能更好地解决拼写错误。

## 展望未来

以上涵盖了我为 GCC 9 所做的一些改变。

也许一个更深层次的变化是，我们现在有了一套针对 GCC 的[用户体验指南，当我们实现新的诊断时，试图保持对程序员体验的关注。如果你想参与 GCC 开发，请加入我们的](https://gcc.gnu.org/onlinedocs/gccint/User-Experience-Guidelines.html) [GCC 邮件列表](https://gcc.gnu.org/lists.html)。入侵诊断系统是一个很好的开始方式。

## 尝试一下

GCC 9 将会在 [Fedora 30](https://fedoraproject.org/wiki/Releases/30/Schedule) 中，应该会在几周内出来。

对于简单的代码示例，您可以在[https://godbolt.org/](https://godbolt.org/)使用新的
GCC(选择 GCC“trunk”)。

玩得开心！

## 请参见

如果您在红帽企业版 Linux 6、7 或 T2 8 测试版 T3 上使用 GCC 8，您可能会感兴趣:

*   [GCC 8 中的可用性改进](https://developers.redhat.com/blog/2018/03/15/gcc-8-usability-improvements/)
*   [GCC 的推荐编译器和链接器标志](https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/)
*   [如何在红帽企业版 Linux 上安装 GCC 8](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/)

*Last updated: March 7, 2019*