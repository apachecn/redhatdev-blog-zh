# GCC 8 中的可用性改进

> 原文：<https://developers.redhat.com/blog/2018/03/15/gcc-8-usability-improvements>

我在 GCC 的 Red Hat 工作，GNU 编译器集合。

去年我主要关注的是让 GCC 更容易使用，所以我想我应该写一些我在 GCC 的下一个主要版本 GCC 8 中对 C 和 C++的改进。您可以使用 Red Hat Developer Toolset (DTS) 在 Red Hat Enterprise Linux 6 和 7 上轻松安装 GCC 8。GCC 8 是[红帽企业版 Linux 8 Beta](https://developers.redhat.com/rhel8) 中的默认编译器。GCC 8 在 Fedora 28 和更高版本中也有。

## 帮你改正愚蠢的错误

### 快速测验#1

快速找到以下错误:

```
int test(void)
{
  return 42
}

```

在 gcc 的早期版本中，我们打印了相当无用的:

```
$ gcc t.c
t.c: In function ‘test’:
t.c:4:1: error: expected ‘;’ before ‘}’ token
 }
 ^

```

对于 gcc 8，我已经修复了一些东西，以便正确地突出显示丢失的分号的位置:

```
$ gcc t.c
t.c: In function ‘test’:
t.c:3:12: error: expected ‘;’ before ‘}’ token
   return 42
            ^
            ;
 }
 ~

```

特别是，错误消息现在显示了正确的行。它现在还建议通过“fix-it 提示”插入丢失的分号，以便支持它们的 IDE 可以为您修复问题(例如， [Eclipse 的 CDT](https://bugs.eclipse.org/bugs/show_bug.cgi?id=497670) )。

### 快速测验#2

在以下位置找到语法错误:

```
double MIN = 68.0;
double MAX = 72.0;

int logging_enabled;

extern void write_to_log(double);
extern int check_range(void);

void log_when_out_of_range(double temperature)
{
  if (logging_enabled && check_range ()
      && (temperature < MIN || temperature > MAX) {
    write_to_log (temperature);
  }
}

```

gcc 印刷的旧版本:

```
$ gcc unclosed.c
unclosed.c: In function ‘log_when_out_of_range’:
unclosed.c:12:51: error: expected ‘)’ before ‘{’ token
       && (temperature < MIN || temperature > MAX) {
 ^
unclosed.c:15:1: error: expected expression before ‘}’ token
 }
 ^

```

它在抱怨丢失了右括号，但是它对应的是哪个左括号呢？

gcc 8 现在突出显示了相关的左括号:

```
$ gcc unclosed.c
unclosed.c: In function ‘log_when_out_of_range’:
unclosed.c:12:50: error: expected ‘)’ before ‘{’ token
       && (temperature < MIN || temperature > MAX) {
                                                  ^~
                                                  )
unclosed.c:11:6: note: to match this ‘(’
   if (logging_enabled && check_range ()
      ^
unclosed.c:15:1: error: expected expression before ‘}’ token
 }
 ^

```

并提供了一个修复提示，这样 ide 就可以提供自动修复。(遗憾的是，我无法修复 gcc 8 的那个额外的错误)。

如果它们在同一条线上，它会更紧凑地突出显示:

```
$ gcc unclosed-2.c
unclosed-2.c: In function ‘test’:
unclosed-2.c:8:45: error: expected ‘)’ before ‘{’ token
   if (temperature < MIN || temperature > MAX {
      ~                                      ^~
                                             )
unclosed-2.c:11:1: error: expected expression before ‘}’ token
 }
 ^

```

我还修正了我们如何处理:

```
int i
int j;

```

早期的 gcc 版本通常会打印这种难以理解的诊断:

```
$ gcc q.c
q.c:2:1: error: expected ‘=’, ‘,’, ‘;’, ‘asm’ or ‘__attribute__’ before ‘int’
 int j;
 ^

```

如果这两行跨越了两个不同的头文件，那就特别糟糕。

对于 gcc 8，它将错误放在正确的位置，现在打印:

```
$ gcc q.c
q.c:1:6: error: expected ‘;’ before ‘int’
 int i
      ^
      ;
 int j;
 ~~~

```

我觉得这更容易理解。

### 一个棘手的问题？

以下代码有什么问题？

```
const char *test(void)
{
    FILE *f;
    int limit = INT_MAX;

    /* etc */

    return NULL;
}

```

这是一个棘手的问题——代码很好，但是，就像在随机网站上看到的代码片段一样，它缺少`#include`指令。如果您只是简单地将它复制到一个新文件中，并尝试按原样编译它，它会失败。

当复制和粘贴示例时，这可能会令人沮丧——想一想，上面的示例需要哪些头文件？-所以对于 gcc 8，我添加了提示，告诉您缺少哪些头文件(对于最常见的情况):

```
$ gcc incomplete.c
incomplete.c: In function ‘test’:
incomplete.c:3:5: error: unknown type name ‘FILE’
     FILE *f;
     ^~~~
incomplete.c:3:5: note: ‘FILE’ is defined in header ‘<stdio.h>’; did you forget to ‘#include <stdio.h>’?
incomplete.c:1:1:
+#include <stdio.h>
 const char *test(void)
incomplete.c:3:5:
     FILE *f;
     ^~~~
incomplete.c:4:17: error: ‘INT_MAX’ undeclared (first use in this function)
     int limit = INT_MAX;
                 ^~~~~~~
incomplete.c:4:17: note: ‘INT_MAX’ is defined in header ‘<limits.h>’; did you forget to ‘#include <limits.h>’?
incomplete.c:1:1:
+#include <limits.h>
 const char *test(void)
incomplete.c:4:17:
     int limit = INT_MAX;
                 ^~~~~~~
incomplete.c:4:17: note: each undeclared identifier is reported only once for each function it appears in
incomplete.c:8:12: error: ‘NULL’ undeclared (first use in this function)
     return NULL;
            ^~~~
incomplete.c:8:12: note: ‘NULL’ is defined in header ‘<stddef.h>’; did you forget to ‘#include <stddef.h>’?
incomplete.c:1:1:
+#include <stddef.h>
 const char *test(void)
incomplete.c:8:12:
     return NULL;
            ^~~~

```

和以前一样，这些修复提示可以通过`-fdiagnostics-parseable-fixits`变成机器可读的(供 IDE 使用),或者您可以使用`-fdiagnostics-generate-patch`让 gcc 发出:

```
--- incomplete.c
+++ incomplete.c
@@ -1,3 +1,6 @@
+#include <stdio.h>
+#include <limits.h>
+#include <stddef.h>
 const char *test (void)
 {
     FILE *f;

```

类似地，对于 C++，gcc 现在会发出提示，提示缺少的“std”包括:

```
std::string s("hello world");

```

```
$ gcc incomplete.cc
incomplete.cc:1:6: error: ‘string’ in namespace ‘std’ does not name a type
 std::string s("hello world");
      ^~~~~~
incomplete.cc:1:1: note: ‘std::string’ is defined in header ‘<string>’; did you forget to ‘#include <string>’?
+#include <string>
 std::string s("hello world");
 ^~~

```

## 参数类型不匹配

我发现我的很多时间都花在了尝试调用 API 和处理像下面这样的愚蠢错误上:

```
extern int callee(int one, const char *two, float three);

int caller(int first, int second, float third)
{
  return callee(first, second, third);
}

```

GCC 的旧版本在描述这个问题时没有多大帮助:

```
$ gcc arg-type-mismatch.cc
arg-type-mismatch.cc: In function ‘int caller(int, int, float)’:
arg-type-mismatch.cc:5:37: error: invalid conversion from ‘int’ to ‘const char*’ [-fpermissive]
   return callee(first, second, third);
 ^
arg-type-mismatch.cc:1:12: error:   initializing argument 2 of ‘int callee(int, const char*, float)’ [-fpermissive]
 extern int callee(int one, const char *two, float three);
 ^

```

你被告知参数编号，但是你必须计算源代码中的逗号。

对于 gcc 8，它打印如下内容:

```
$ gcc arg-type-mismatch.cc
arg-type-mismatch.cc: In function ‘int caller(int, int, float)’:
arg-type-mismatch.cc:5:24: error: invalid conversion from ‘int’ to ‘const char*’ [-fpermissive]
   return callee(first, second, third);
                        ^~~~~~
arg-type-mismatch.cc:1:40: note:   initializing argument 2 of ‘int callee(int, const char*, float)’
 extern int callee(int one, const char *two, float three);
                            ~~~~~~~~~~~~^~~

```

我希望您同意这更具可读性:编译器在调用点给有问题的参数加下划线，并在被调用方的声明中给相应的参数加下划线，因此您可以立即看到不匹配。

实现这一点比你想象的要困难得多。特别是，GCC 表达式的内部表示没有区分参数的声明和使用该参数的,所以在上面的例子中没有自然的地方存储使用`second`的源位置。所以我不得不做大量的“幕后”工作来让这一切发生。

## 我如何到达某个`private`字段？

我经常发现自己试图访问 C++对象的一个字段，知道字段名，但发现它是私有的，然后试图记住访问器的名称——是`get_foo`、`getFoo`、`read_foo`，还是仅仅是`foo`？

所以对于 GCC 8，我在“field is private”错误中添加了提示，展示了如何使用访问器来访问有问题的字段(如果存在的话):

例如，假设:

```
class foo
{
public:
  double get_ratio() const { return m_ratio; }

private:
  double m_ratio;
};

void test(foo *ptr)
{
  if (ptr->m_ratio >= 0.5)
    ;// etc
}

```

现在，编译器在抱怨直接访问时会给出以下提示:

```
$ gcc accessor.cc
accessor.cc: In function ‘void test(foo*)’:
accessor.cc:12:12: error: ‘double foo::m_ratio’ is private within this context
   if (ptr->m_ratio >= 0.5)
            ^~~~~~~
accessor.cc:7:10: note: declared private here
   double m_ratio;
          ^~~~~~~
accessor.cc:12:12: note: field ‘double foo::m_ratio’ can be accessed via ‘double foo::get_ratio() const’
   if (ptr->m_ratio >= 0.5)
            ^~~~~~~
            get_ratio()

```

## 模板类型差异

涉及 C++模板的编译器错误以难以阅读而闻名。下面是一个简单示例的旧版本 gcc 发出的内容:

```
$ gcc templates.cc
templates.cc: In function ‘void test()’:
templates.cc:9:25: error: could not convert ‘vector<double>()’ from ‘vector<double>’ to ‘vector<int>’
   fn_1(vector<double> ());
 ^
templates.cc:10:26: error: could not convert ‘map<int, double>()’ from ‘map<int, double>’ to ‘map<int, int>’
   fn_2(map<int, double>());
 ^

```

即使在这种简单的情况下，我发现有一个“文字墙”的问题，我的眼睛开始变得呆滞。

对于 gcc 8，我借鉴了 clang 的一些好的想法来改进这种模板诊断。我给消息添加了颜色，因此模板类型中不匹配的部分用绿色突出显示。此外，我们现在省略了两个不匹配模板之间的公共参数，改为打印`[...]`:

```
$ gcc templates.cc
templates.cc: In function ‘void test()’:
templates.cc:9:8: error: could not convert ‘vector<double>()’ from ‘vector<double>’ to ‘vector<int>’
   fn_1(vector<double> ());
        ^~~~~~~~~~~~~~~~~
templates.cc:10:8: error: could not convert ‘map<int, double>()’ from ‘map<[...],double>’ to ‘map<[...],int>’
   fn_2(map<int, double>());
        ^~~~~~~~~~~~~~~~~~

```

使用`-fno-elide-type`可以看到那些`[...]`省略的参数:

```
$ gcc templates.cc -fno-elide-type
templates.cc: In function ‘void test()’:
templates.cc:9:8: error: could not convert ‘vector<double>()’ from ‘vector<double>’ to ‘vector<int>’
   fn_1(vector<double> ());
        ^~~~~~~~~~~~~~~~~
templates.cc:10:8: error: could not convert ‘map<int, double>()’ from ‘map<int,double>’ to ‘map<int,int>’
   fn_2(map<int, double>());
        ^~~~~~~~~~~~~~~~~~

```

对于更复杂的错误，我实现了`-fdiagnostics-show-template-tree`，它以分层的形式可视化不匹配的模板(对于这个相当不自然的例子):

```
$ gcc templates-2.cc -fdiagnostics-show-template-tree
templates-2.cc: In function ‘void test()’:
templates-2.cc:9:8: error: could not convert ‘vector<double>()’ from ‘vector<double>’ to ‘vector<int>’
  vector<
    [double != int]>
   fn_1(vector<double> ());
        ^~~~~~~~~~~~~~~~~
templates-2.cc:10:8: error: could not convert ‘map<map<int, vector<double> >, vector<double> >()’ from ‘map<map<[...],vector<double>>,vector<double>>’ to ‘map<map<[...],vector<float>>,vector<float>>’
  map<
    map<
      [...],
      vector<
        [double != float]>>,
    vector<
      [double != float]>>
   fn_2(map<map<int, vector<double>>, vector<double>> ());
        ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```

这同样适用于`-fno-elide-type`:

```
$ gcc templates-2.cc -fdiagnostics-show-template-tree -fno-elide-type
templates-2.cc: In function ‘void test()’:
templates-2.cc:9:8: error: could not convert ‘vector<double>()’ from ‘vector<double>’ to ‘vector<int>’
  vector<
    [double != int]>
   fn_1(vector<double> ());
        ^~~~~~~~~~~~~~~~~
templates-2.cc:10:8: error: could not convert ‘map<map<int, vector<double> >, vector<double> >()’ from ‘map<map<int,vector<double>>,vector<double>>’ to ‘map<map<int,vector<float>>,vector<float>>’
  map<
    map<
      int,
      vector<
        [double != float]>>,
    vector<
      [double != float]>>
   fn_2(map<map<int, vector<double>>, vector<double>> ());
        ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```

## 加上一堆其他的东西...

如果你在定义宏之前使用它，C++前端会给你一个提示(例如，如果你弄乱了#includes 的顺序):

```
$ gcc ordering.cc
ordering.cc:2:24: error: expected ‘;’ at end of member declaration
   virtual void clone() const OVERRIDE { }
                        ^~~~~
                             ;
ordering.cc:2:30: error: ‘OVERRIDE’ does not name a type
   virtual void clone() const OVERRIDE { }
                              ^~~~~~~~
ordering.cc:2:30: note: the macro ‘OVERRIDE’ had not yet been defined
In file included from ordering.cc:5:
c++11-compat.h:2: note: it was later defined here
 #define OVERRIDE override

```

我在 C++前端的-Wold-style-cast 诊断中添加了 fix-it 提示，告诉您是否可以使用`static_cast`、`const_cast`、`reinterpret_cast`等:

```
$ gcc -c old-style-cast-fixits.cc -Wold-style-cast
old-style-cast-fixits.cc: In function ‘void test_1(void*)’:
old-style-cast-fixits.cc:5:19: warning: use of old-style cast to ‘struct foo*’ [-Wold-style-cast]
   foo *f = (foo *)ptr;
                   ^~~
            ----------
            static_cast<foo *> (ptr)

```

...诸如此类。

## 尝试一下

参见 *[如何在红帽企业版 Linux](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/)* [7](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/) 上安装 GCC 8。你也可以在 [RHEL 8 Beta](https://developers.redhat.com/rhel8) 中试试 GCC 8。GCC 8 在 Fedora 28 和更高版本中也有。

或者，对于简单的代码示例，您可以在[https://godbolt.org/](https://godbolt.org/)试验新的 gcc。

玩得开心！

您可能还想看看:

*   *[GNU 工具链更新——2018 年春季](https://developers.redhat.com/blog/2018/03/26/gnu-toolchain-update-2018/)* 作者[尼克·克里夫顿](https://developers.redhat.com/blog/author/nickclifton/)
*   [*由*](https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/) *[Florian Weimer](https://developers.redhat.com/blog/author/florianweimer/) 为 GCC* 推荐的编译器和链接器标志
*   [*检测字符串截断用 GCC 8*](https://developers.redhat.com/blog/2018/05/24/detecting-string-truncation-with-gcc-8/) 由[马丁·塞博尔](https://developers.redhat.com/blog/author/msebor/)

*Last updated: March 8, 2019*