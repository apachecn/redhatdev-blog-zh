# 如何从命令行使用 LLDB 调试函数返回的位置

> 原文：<https://developers.redhat.com/blog/2019/09/11/how-to-debug-where-a-function-returns-using-lldb-from-the-command-line>

当我想知道一个函数在哪里返回时，我经常会遇到这种情况。不需要知道返回值，因为这对于多个代码路径可能是相同的(例如，`nullptr`如果出错)。这很尴尬，但我有时会在代码中放入`fprintf(stderr, "T1");`,只是为了跟踪执行的路径。不用说，这种行为需要手动编辑和重新编译，应该尽可能避免。

这里有一种从命令行使用`[lldb](https://lldb.llvm.org/)`优雅地调试函数返回的方法。

考虑这个`test.cpp`程序，你想做的就是找出函数`foo`返回的位置:

```
int foo(int argc) {
  switch (argc) {
  case 1:
    return 1;
  case 2:
    return 2;
  case 3:
    return 3;
  }
  return -1;
}

int main(int argc, char *argv[]) { return foo(argc); }

```

注意这段代码中有五个`return`语句，但是我们只想知道`foo`中的哪四个被命中了。

让我们从用调试符号编译上面的程序开始:

```
clang -g test.cpp
```

要知道`foo`返回到哪里，您可以运行下面的命令。

```
lldb -b -o "br set -X foo -p return" -o r ./a.out -- hello world
```

1.  `-b`开启批处理模式。我发现这很方便，因为它让你以一种“一劳永逸”的方式执行你的程序，而不用在程序完成时离开调试器。
2.  `-o "br set -X foo -p return"`在函数`foo`内部的模式返回上设置一个断点。注意，断点仅限于函数`foo`内部的返回语句(我们有四个，而不是五个位置)。
3.  `-o r`运行程序，并在`foo`内的断点处停止。
4.  `--`之后的所有内容都作为参数传递给我们的程序`./a,out`。

这里你可以看到效果:

```
(lldb) target create "./a.out"
Current executable set to './a.out' (x86_64).
(lldb) settings set -- target.run-args  "hello" "world"
(lldb) br set -X foo -p return
Breakpoint 1: 4 locations.
(lldb) r
Process 7542 stopped
* thread #1, name = 'a.out', stop reason = breakpoint 1.3
    frame #0: 0x0000000000401170 a.out`foo(argc=3) at test.cpp:8:5
   5   	  case 2:
   6   	    return 2;
   7   	  case 3:
-> 8   	    return 3;
   9   	  }
   10  	  return -1;
   11  	}

Process 7542 launched: '/home/kkleine/a.out' (x86_64)

```

我希望你喜欢这个提示。关于断点的更多有用的 LLDB 技巧，请访问这个页面:[https://lldb.llvm.org/use/tutorial.html#setting-breakpoints](https://lldb.llvm.org/use/tutorial.html#setting-breakpoints)

*Last updated: July 1, 2020*