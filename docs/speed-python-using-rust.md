# 使用 Rust 加速你的 Python

> 原文：<https://developers.redhat.com/blog/2017/11/16/speed-python-using-rust>

## 什么是铁锈？

Rust 是一种系统编程语言，运行速度极快，防止 segfaults，并保证线程安全。

**特色**

*   零成本抽象
*   移动语义
*   保证内存安全
*   没有数据竞争的线程
*   基于性状的仿制药
*   模式匹配
*   类型推理
*   最短运行时间
*   高效的 C 绑定

描述摘自[rust-lang.org](http://rust-lang.org)。

## 为什么这对 Python 开发人员很重要？

对铁锈更好的描述我是从 [**伊利亚**](https://github.com/dlight)([**铁锈巴西电报团成员)**](https://t.me/rustlangbr) 那里听到的。

> Rust 是一种语言，它允许你构建高级抽象，但不放弃低级控制——也就是说，控制数据如何在内存中表示，控制你想要使用的线程模型等等。
> **Rust 是**一种语言，它通常可以在编译期间检测到最严重的并行性和内存管理错误(例如在不同线程上访问数据而不同步，或者在数据被解除分配后使用数据)，但如果你真的知道自己在做什么，它会给你一个逃生出口。
> **Rust 是**一种语言，因为它没有运行时，所以可以用来与任何运行时集成；你可以在 Rust 中编写一个本地扩展，由 node.js 程序调用，或者由 python 程序调用，或者由 ruby、lua 等程序调用。但是，您可以使用这些语言在 Rust 中编写程序。-“埃利亚斯·加布里埃尔·阿马拉尔·达席尔瓦”

有很多 Rust 包可以帮助你用 Rust 扩展 Python。

我可以提到阿明·罗纳彻(Flask 的创造者)创造的[乳蛇](https://github.com/getsentry/milksnake)和[pyo 3](https://github.com/PyO3/pyo3)Python 解释器的 Rust bindings。

### 参见本文底部的完整参考列表。

让我们**看看它的作用**

在这篇文章中，我将使用 [Rust Cpython](https://github.com/dgrunwald/rust-cpython) ，这是我测试过的唯一一个，它与 Rust 的稳定版本兼容，并且使用起来很简单。

> **注意** : [PyO3](https://github.com/PyO3/pyo3) 是 rust-cpython 的一个分支，有许多改进，但只适用于 rust 的夜间版本，所以我更喜欢在这篇文章中使用 stable，无论如何这里的例子也适用于 PyO3。

**优点:**编写 Rust 函数并从 Python 中导入很容易，正如您将从性能方面的基准测试中看到的那样。

**缺点:**你的**项目/库/框架**的发布会要求 Rust 模块在目标系统上编译，因为环境和架构的变化，会有一个**编译**阶段，这是你在安装纯 Python 库时没有的，你可以更容易地使用 [rust-setuptools](https://pypi.python.org/pypi/setuptools-rust) 或使用 [MilkSnake](https://github.com/getsentry/milksnake) 将二进制数据嵌入 Python Wheels。

## Python 有时很慢

是的，Python 在某些情况下是出了名的“慢”，好消息是这真的不重要，取决于您的项目目标和优先级。对于大多数项目来说，这个细节不会很重要。

然而，您可能会面临**罕见的**情况，即单个函数或模块花费太多时间，被检测为项目性能的瓶颈，这通常发生在字符串解析和图像处理中。

## 例子

假设您有一个执行字符串处理的 Python 函数，以下面的简单示例`counting pairs of repeated chars`为例，但是请记住，这个示例可以用其他`string processing`函数或 Python 中任何其他通常较慢的过程来重现。

```
# How many subsequent-repeated group of chars are in the given string? 
abCCdeFFghiJJklmnopqRRstuVVxyZZ... {millions of chars here}
  1   2    3        4    5   6
```

Python 在处理大型`string`时速度很慢，所以可以使用`pytest-benchmark`来比较`Pure Python (with Iterator Zipping)`函数和`Regexp`实现。

```
# Using a Python3.6 environment
$ pip3 install pytest pytest-benchmark

```

然后编写一个名为`doubles.py`的新 Python 程序

```
import re
import string
import random

# Python ZIP version
def count_doubles(val):
    total = 0
    # there is an improved version later on this post
    for c1, c2 in zip(val, val[1:]):
        if c1 == c2:
            total += 1
    return total

# Python REGEXP version
double_re = re.compile(r'(?=(.)\1)')

def count_doubles_regex(val):
    return len(double_re.findall(val))

# Benchmark it
# generate 1M of random letters to test it
val = ''.join(random.choice(string.ascii_letters) for i in range(1000000))

def test_pure_python(benchmark):
    benchmark(count_doubles, val)

def test_regex(benchmark):
    benchmark(count_doubles_regex, val)
```

运行 **pytest** 进行比较:

```
$ pytest doubles.py                                                                                                           
=============================================================================
platform linux -- Python 3.6.0, pytest-3.2.3, py-1.4.34, pluggy-0.4.
benchmark: 3.1.1 (defaults: timer=time.perf_counter disable_gc=False min_roun
rootdir: /Projects/rustpy, inifile:
plugins: benchmark-3.1.1
collected 2 items

doubles.py ..

-----------------------------------------------------------------------------
Name (time in ms)         Min                Max               Mean          
-----------------------------------------------------------------------------
test_regex            24.6824 (1.0)      32.3960 (1.0)      27.0167 (1.0)    
test_pure_python      51.4964 (2.09)     62.5680 (1.93)     52.8334 (1.96)   
-----------------------------------------------------------------------------

```

让我们拿`Mean`来做比较:

*   **Regexp**-27.0167**-越少越好**
*   **Python Zip** - 52.8334

# 用 Rust 扩展 Python

## 创建新的板条箱

> 我们称之为锈包。

安装了 rust(推荐的方式是[https://www.rustup.rs/](https://www.rustup.rs/))Rust 也可以通过 [rust 工具集](https://developers.redhat.com/blog/2017/11/01/getting-started-rust-toolset-rhel/)在 Fedora 和 RHEL 库上获得

> 我用了`rustc 1.21.0`

在同一文件夹中运行:

```
cargo new pyext-myrustlib
```

它在同一个名为`pyext-myrustlib`的文件夹中创建了一个新的 Rust 项目，其中包含了`Cargo.toml` (cargo 是 Rust 包管理器)和一个`src/lib.rs`(我们在这里编写我们的库实现)。

## 编辑工作，toml

它将使用`rust-cpython`板条箱作为依赖，并告诉 cargo 生成一个从 Python 导入的`dylib`。

```
[package]
name = "pyext-myrustlib"
version = "0.1.0"
authors = ["Bruno Rocha <rochacbruno@gmail.com>"]

[lib]
name = "myrustlib"
crate-type = ["dylib"]

[dependencies.cpython]
version = "0.1"
features = ["extension-module"]
```

## 编辑资源/库文件

我们需要做的是:

1.  从`cpython`箱导入所有宏。
2.  将 CPython 中的`Python`和`PyResult`类型放入我们的 lib 范围。
3.  在`Rust`中编写`count_doubles`函数实现，注意这与纯 Python 版本非常相似，除了:
    *   它将一个`Python`作为第一个参数，这是对 Python 解释器的引用，并允许 Rust 使用`Python GIL`。
    *   接收一个类型为`val`的`&str`作为参考。
    *   返回一个`PyResult`，这是一个允许出现 Python 异常的类型。
    *   在`Ok(total)`中返回一个`PyResult`对象(**结果**是一个代表成功(Ok)或失败(Err)的枚举类型)，由于我们的函数被期望返回一个`PyResult`，编译器将负责在该类型上包装**我们的`Ok`。(注意，我们的 PyResult 期望一个`u64`作为返回值)。**
4.  使用`py_module_initializer!`宏，我们向库注册新的属性，包括`__doc__`，我们还添加了引用我们的`Rust implementation of the function`的`count_doubles`属性。
    *   注意名字 **lib** myrustlib、 **initlib** myrustlib、 **PyInit。**
    *   我们还使用了`try!`宏，它相当于 Python 的`try.. except`。
    *   返回`Ok(())`-`()`是一个空的结果元组，相当于 Python 中的`None`。

```
#[macro_use]
extern crate cpython;

use cpython::{Python, PyResult};

fn count_doubles(_py: Python, val: &str) -> PyResult<u64> {
    let mut total = 0u64;

    // There is an improved version later on this post
    for (c1, c2) in val.chars().zip(val.chars().skip(1)) {
        if c1 == c2 {
            total += 1;
        }
    }

    Ok(total)
}

py_module_initializer!(libmyrustlib, initlibmyrustlib, PyInit_myrustlib, |py, m | {
    try!(m.add(py, "__doc__", "This module is implemented in Rust"));
    try!(m.add(py, "count_doubles", py_fn!(py, count_doubles(val: &str))));
    Ok(())
});

```

## 现在让我们用货物建造它

```
$ cargo build --release
    Finished release [optimized] target(s) in 0.0 secs

$ ls -la target/release/libmyrustlib*
target/release/libmyrustlib.d
target/release/libmyrustlib.so*  <-- Our dylib is here
```

现在让我们将生成的`.so` lib 复制到我们的`doubles.py`所在的同一个文件夹中。

> 注意:在 **Fedora** 上你必须得到一个`.so`在其他系统你可以得到一个`.dylib`并且你可以把它改名为`.so`。

```
$ cd ..
$ ls
doubles.py pyext-myrustlib/

$ cp pyext-myrustlib/target/release/libmyrustlib.so myrustlib.so

$ ls
doubles.py myrustlib.so pyext-myrustlib/
```

> 将`myrustlib.so`放在同一个文件夹中或者添加到 Python 路径中允许直接导入它，就像它是一个 Python 模块一样透明。

## 从 Python 导入并比较结果

编辑你的`doubles.py`，现在导入我们的`Rust implemented`版本，并为它添加一个`benchmark`。

```
import re
import string
import random
import myrustlib   #  <-- Import the Rust implemented module (myrustlib.so)

def count_doubles(val):
    """Count repeated pair of chars ins a string"""
    total = 0
    for c1, c2 in zip(val, val[1:]):
        if c1 == c2:
            total += 1
    return total

double_re = re.compile(r'(?=(.)\1)')

def count_doubles_regex(val):
    return len(double_re.findall(val))

val = ''.join(random.choice(string.ascii_letters) for i in range(1000000))

def test_pure_python(benchmark):
    benchmark(count_doubles, val)

def test_regex(benchmark):
    benchmark(count_doubles_regex, val)

def test_rust(benchmark):   #  <-- Benchmark the Rust version
    benchmark(myrustlib.count_doubles, val)

```

# 基准

```
$ pytest doubles.py
==============================================================================
platform linux -- Python 3.6.0, pytest-3.2.3, py-1.4.34, pluggy-0.4.
benchmark: 3.1.1 (defaults: timer=time.perf_counter disable_gc=False min_round
rootdir: /Projects/rustpy, inifile:
plugins: benchmark-3.1.1
collected 3 items

doubles.py ...

-----------------------------------------------------------------------------
Name (time in ms)         Min                Max               Mean          
-----------------------------------------------------------------------------
test_rust              2.5555 (1.0)       2.9296 (1.0)       2.6085 (1.0)    
test_regex            25.6049 (10.02)    27.2190 (9.29)     25.8876 (9.92)   
test_pure_python      52.9428 (20.72)    56.3666 (19.24)    53.9732 (20.69)  
-----------------------------------------------------------------------------
```

让我们拿`Mean`来做比较:

*   **生锈** - 2.6085 **< -越少越好**
*   **正则表达式** - 25.8876
*   **Python Zip** - 53.9732

Rust 实现可以比 Python Regex 快**10 倍**，比纯 Python 版本快**21 倍**。

> 有趣的是 **Regex** 版本只比纯 Python 快 2 倍:)

> 注意:这些数字仅在这种特殊情况下有意义，对于其他情况，比较可能会有所不同。

# 更新和改进

这篇文章发表后，我得到了一些关于 [r/python](https://www.reddit.com/r/Python/comments/7dct9v/use_rust_to_write_python_modules/) 和 [r/rust](https://www.reddit.com/r/rust/comments/7dctmp/red_hat_developers_blog_speed_up_your_python/) 的评论

贡献来自于[拉请求](https://github.com/rochacbruno/rust-python-example/pulls?utf8=%E2%9C%93&q=is%3Apr)，如果你认为功能可以改进，你可以发送一个新的。

感谢: [Josh Stone](https://github.com/cuviper) 我们得到了 Rust 的一个更好的实现，它只迭代一次字符串，也是 Python 的等价物。

感谢:[紫色精灵](https://github.com/purple-pixie)我们用`itertools`实现了 Python，然而这个版本并没有表现得更好，仍然需要改进。

## 仅迭代一次

```
fn count_doubles_once(_py: Python, val: &str) -> PyResult<u64> {
    let mut total = 0u64;

    let mut chars = val.chars();
    if let Some(mut c1) = chars.next() {
        for c2 in chars {
            if c1 == c2 {
                total += 1;
            }
            c1 = c2;
        }
    }

    Ok(total)
}
```

```
def count_doubles_once(val):
    total = 0
    chars = iter(val)
    c1 = next(chars)
    for c2 in chars:
        if c1 == c2:
            total += 1
        c1 = c2
    return total
```

## 使用 itertools 的 Python

```
import itertools

def count_doubles_itertools(val):
    c1s, c2s = itertools.tee(val)
    next(c2s, None)
    total = 0
    for c1, c2 in zip(c1s, c2s):
        if c1 == c2:
            total += 1
    return total
```

### 为什么不是 c/c++/nim/go/ĺua/pypy/{other 语}？

好吧，这不是这篇文章的目的，这篇文章从来都不是关于比较`Rust` X `other language`，这篇文章是专门关于**如何使用 Rust 来扩展和加速 Python** ，这样做意味着你有一个很好的理由选择 Rust 而不是`other language`，或者是因为它的生态系统，或者是因为它的安全性和工具性，或者仅仅是因为你喜欢 Rust 无关紧要的原因，这篇文章在这里展示如何将它与 **Python** 一起使用。

我(个人)可能会说 Rust 更多的是`future proof`因为它是新的，有很多改进要来，也因为它的生态系统、工具和社区，也因为我对 Rust 语法感到舒服，我真的很喜欢它！

因此，不出所料，人们开始抱怨其他语言的使用，这成为一种基准，我认为这很酷！

因此，作为我要求改进的一部分，一些人在[黑客新闻](https://news.ycombinator.com/item?id=15719254)上也发表了想法， [martinxyz](https://github.com/martinxyz) 发表了一个使用 C 和 SWIG 的实现，表现非常好。

c 代码(swig 样板省略)

```
uint64_t count_byte_doubles(char * str) {
  uint64_t count = 0;
  while (str[0] && str[1]) {
    if (str[0] == str[1]) count++;
    str++;
  }
  return count;
}
```

我们的红帽匠同事[乔希·斯通](https://github.com/cuviper)再次改进了 Rust 的实现，用`bytes`代替了`chars`，所以这是与`C`的公平竞争，因为 C 比较的是字节而不是 Unicode 字符。

```
fn count_doubles_once_bytes(_py: Python, val: &str) -> PyResult<u64> {
    let mut total = 0u64;

    let mut chars = val.bytes();
    if let Some(mut c1) = chars.next() {
        for c2 in chars {
            if c1 == c2 {
                total += 1;
            }
            c1 = c2;
        }
    }

    Ok(total)
}
```

也有比较 Python `list comprehension`和`numpy`的想法，所以我包括在这里

Numpy:

```
import numpy as np

def count_double_numpy(val):
    ng=np.fromstring(val,dtype=np.byte)
    return np.sum(ng[:-1]==ng[1:])
```

列表理解

```
def count_doubles_comprehension(val):
    return sum(1 for c1, c2 in zip(val, val[1:]) if c1 == c2)
```

完整的测试用例在存储库`test_all.py`文件中。

## 新结果

**注**:请记住，比较是在相同的环境中进行的，如果在不同的环境中使用另一个编译器和/或不同的标签，可能会有一些差异。

```
-------------------------------------------------------------------------------------------------
Name (time in us)                     Min                    Max                   Mean          
-------------------------------------------------------------------------------------------------
test_rust_bytes_once             476.7920 (1.0)         830.5610 (1.0)         486.6116 (1.0)    
test_c_swig_bytes_once           795.3460 (1.67)      1,504.3380 (1.81)        827.3898 (1.70)   
test_rust_once                   985.9520 (2.07)      1,483.8120 (1.79)      1,017.4251 (2.09)   
test_numpy                     1,001.3880 (2.10)      2,461.1200 (2.96)      1,274.8132 (2.62)   
test_rust                      2,555.0810 (5.36)      3,066.0430 (3.69)      2,609.7403 (5.36)   
test_regex                    24,787.0670 (51.99)    26,513.1520 (31.92)    25,333.8143 (52.06)  
test_pure_python_once         36,447.0790 (76.44)    48,596.5340 (58.51)    38,074.5863 (78.24)  
test_python_comprehension     49,166.0560 (103.12)   50,832.1220 (61.20)    49,699.2122 (102.13) 
test_pure_python              49,586.3750 (104.00)   50,697.3780 (61.04)    50,148.6596 (103.06) 
test_itertools                56,762.8920 (119.05)   69,660.0200 (83.87)    58,402.9442 (120.02) 
-------------------------------------------------------------------------------------------------

```

*   这个`new Rust implementation comparing bytes`比旧的对比 Unicode `chars`好了**2 倍**
*   使用 SWIG 的`Rust`版本仍然优于`C`
*   `Rust`对比`unicode chars`还是比`numpy`好
*   然而,`Numpy`比`first Rust implementation`要好，后者在 unicode 字符上有**双重迭代的问题**
*   使用`list comprehension`并不比使用`pure Python`有显著的不同

> 注意:如果你想提出改变或改进，请在此发送一份 PR:[https://github.com/rochacbruno/rust-python-example/](https://github.com/rochacbruno/rust-python-example/)

# 结论

回到这篇文章的目的“如何用 Rust 加速你的 Python ”,我们从下面开始:

- **纯 Python** 函数取 **102 ms.**
-用 **Numpy** (用 C 实现)改进取 **3 ms.**
-以 **Rust** 取 **1 ms.** 结束

在这个例子中, **Rust** 比我们的 **Pure** Python 快**100 倍**。

不会奇迹般地拯救你，你必须懂得语言才能实现巧妙的解决方案，一旦正确实现，它在性能方面的价值不亚于 C 语言，还会带来惊人的工具、生态系统、社区和安全奖励。

从复杂程度来看，`Rust`可能还不是**和**的选择，也可能不是编写普通简单`applications`如`web`站点和`test automation`脚本的更好选择。

然而，对于项目的`specific parts`来说，Python 是已知的瓶颈，您的自然选择是实现一个`C/C++`扩展，用 Rust 编写这个扩展看起来更容易维护。

Rust 中仍有许多改进，还有许多其他的板条箱提供集成。即使你现在还没有把这种语言包括在你的工具带中，关注未来也是值得的！

## 参考

这里展示的例子的代码片段可以在 GitHub repo:[https://github.com/rochacbruno/rust-python-example](https://github.com/rochacbruno/rust-python-example)中找到。

这本出版物中的例子受到了**塞缪尔·科尔米耶-饭岛**在**加拿大 Pycon】的`Extending Python with Rust`演讲的启发。视频在这里:[https://www.youtube.com/watch?v=-ylbuEzkG4M](https://www.youtube.com/watch?v=-ylbuEzkG4M)。**

同样由`My Python is a little Rust-y`由**丹·卡拉汉**在**皮孔蒙特**。视频在这里:[https://www.youtube.com/watch?v=3CwJ0MH-4MA](https://www.youtube.com/watch?v=3CwJ0MH-4MA)。

其他参考:

*   [https://github.com/mitsuhiko/snaek](https://github.com/mitsuhiko/snaek)
*   [https://github.com/PyO3/pyo3](https://github.com/PyO3/pyo3)
*   [https://pypi . python . org/pypi/setup tools-rust](https://pypi.python.org/pypi/setuptools-rust)
*   [https://github . com/mckaymatt/cookiecutter-py package-rust-cross-platform-publish](https://github.com/mckaymatt/cookiecutter-pypackage-rust-cross-platform-publish)
*   [http://jakegoulding.com/rust-ffi-omnibus/](http://jakegoulding.com/rust-ffi-omnibus/)
*   [https://github . com/urschrei/poly label-RS/blob/master/src/FFI . RS](https://github.com/urschrei/polylabel-rs/blob/master/src/ffi.rs)
*   [https://bheisler.github.io/post/calling-rust-in-python/](https://bheisler.github.io/post/calling-rust-in-python/)
*   [https://github . com/saethlin/rust-lather](https://github.com/saethlin/rust-lather)

**加入社区:**

加入 Rust 社区，你可以在[https://www.rust-lang.org/en-US/community.html](https://www.rust-lang.org/en-US/community.html)找到群链接。

如果你会说葡萄牙语，我推荐你加入 https://t.me/rustlangbr 的 T2，Youtube 上还有 http://bit.ly/canalrustbr 的 T4。

## 作者

布鲁诺·罗查

*   红帽高级质量工程师
***   在[CursoDePython.com.br](http://CursoDePython.com.br)教授 Python 和 Flask*   Python 软件基金会的会员*   RustBR 研究小组成员**

**更多信息:[http://about.me/rochacbruno](http://about.me/rochacbruno)和[http://brunorocha.org](http://brunorocha.org/)**

 *** * *

**下一步何去何从——在 Red Hat Enterprise Linux 上开发**

*   [**如何安装 Python 3、pip、venv、virtualenv、pipenv**](https://developers.redhat.com/blog/2018/08/13/install-python3-rhel/)
*   [**通过`yum`安装 Rust 并建立 Hello World**](https://developers.redhat.com/products/clang-llvm-go-rust/hello-world/#fndtn-rust)

* * *

**无论你是容器新手还是有经验的人，下载这个** [**备忘单**](https://developers.redhat.com/search?t=docker+cheatsheet) **可以在遇到你最近没有完成的任务时帮助你。**

*Last updated: November 5, 2021***