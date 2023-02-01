# 解决 glibc 的 iconv 实用程序中的挂起字符集转换之谜

> 原文：<https://developers.redhat.com/blog/2021/04/23/solving-the-mystery-of-hanging-character-set-conversions-in-glibcs-iconv-utility>

网站访问者在访问数字内容时通常不会考虑字符编码和转换。然而，自从[用户开始将字符串从一台电脑转移到另一台](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/)电脑，工程师们就一直在处理转换问题。今天，当超过 95%的网页使用 [UTF-8](https://tools.ietf.org/html/rfc3629) 时，字符集之间的数据转换可能看起来不太相关。然而，当处理遗留数据和系统，或者处理多字符集比较流行的语言时，它仍然很有用。

例如，在韩国网站上使用 EUC-KR 并不罕见，因为它对韩国文本更有效。一个索引韩国网站的搜索引擎需要能够在 EUC-韩国和 UTF-8 之间转换，以执行相关的字符串比较。

POSIX 提供的`iconv`编程接口，由其同名的实用程序实现，将文本数据从一种字符编码转换成另一种。作为 Red Hat Platform Tools 团队的一名工程师，我的任务是找到并修复 GNU C Library (glibc) `iconv`实用程序中报告的 bug。在这篇文章中，我讨论了我在`iconv`前端模糊化`iconv`和修复 bug 的经验。

## iconv 中的错误处理

典型的`iconv`程序调用如下所示:

```
$ iconv -f ISO-8859-7 -t UTF-8
```

在这种情况下，它应该从标准输入中读取拉丁文和希腊文文本(编码为 ISO/IEC 8859-7 ),并写出与标准输出等效的 UTF-8。

虽然 POSIX 允许`iconv`实用程序在输入无效时出错退出，但是 glibc `iconv`实现有一个扩展，可以更好地容忍错误输入。用户可以在目标字符集规范中添加一个后缀来启用`iconv`功能，该功能可以:

*   忽略无效的输入字符，同时继续处理进一步的输入。
*   将输出字符集中没有对应字符的输入字符音译为足够相似的输出字符组合。

例如，如果我们想将使用音调符号和标准拉丁字母的 UTF-8 编码的捷克语文本转换为 ASCII，下面的调用就可以达到目的:

```
$ iconv -f UTF-8 -t ASCII//TRANSLIT
```

在这种情况下，`iconv`将带有发音符号的字母音译成它们的无发音符号的对等字母。

## iconv_open 中的字符串操作

这个扩展可能被实现为一个字符串后缀，因为底层的`iconv`接口是一个 POSIX 标准，它的签名不允许标志或选项。对`iconv`的调用是通过调用`iconv_open`获得的处理程序进行的，并带有适当的转换源和目标字符编码名称。`iconv_open`函数具有以下签名:

```
iconv_t iconv_open (const char *tocode, const char *fromcode);
```

因此，在不引入新接口的情况下扩展其功能的唯一有意义的方法是在一个字符串参数中传递选项信息。

glibc `iconv`实现识别并使用斜杠分隔的三元组形式的转换源和目标编码:

*   三元组的第一部分可能是编码形式，这意味着字符集。
*   或者，它可能是一个字符集，其中第二个组件理想地是底层字符集的编码形式。如果未指定编码形式，`iconv`将使用默认值。
*   三元组的第三个组成部分是可选的后缀，它支持 GNU 扩展。这意味着 UCS-2//和 ISO-10646/UCS-2/是等效的。

然而，因为大多数常见的`iconv`调用只是使用编码形式，所以在编写接口文档时忘记了斜杠分隔规范的第二个组件，手册页以下面的片段结束:

```
Furthermore the GNU C library and the GNU libiconv library support the following 
two suffixes:

       //TRANSLIT
              When the string "//TRANSLIT" is appended to *tocode*,
              transliteration is activated.  This means that when a
              character cannot be represented in the target character
              set, it can be approximated through one or several
              similarly looking characters.

       //IGNORE
              When the string "//IGNORE" is appended to *tocode*,
              characters that cannot be represented in the target
              character set will be silently discarded.
```

没有对三元组方案以及如何传递多个后缀进行清楚的解释，这导致实现允许如下调用:

```
$ iconv -f ISO-8859-7 -t UTF-8//TRANSLIT//IGNORE
```

注意，缺少的第二个组件和产生的`//`被误解为后缀分隔符。根据我的阅读，这个误解似乎也触及了程序代码。

## 调试 iconv 实用程序

理想情况下，`iconv`实用程序会避免支持字符串后缀形式的选项。相反，它可能只通过这些选项来支持它们，但不是在这种情况下。它接受与基础函数相同的后缀。

2016 年，针对`iconv`实用程序报告了一个错误，揭示了当`TRANSLIT`和`IGNORE`后缀与错误输入和`-c`选项结合使用时的挂起。这种组合意味着`iconv`从输出中省略输入中的无效字符，类似于`IGNORE`，如下所示:

```
$ echo -en '\x80' | iconv -f us-ascii -t us-ascii//translit//ignore -c
```

该报告收集了评论，包括对可能原因和其他类似挂起的提示。直到 2019 年，当我开始深入研究它时，我才在报告上取得了很大进展。

由于不太了解`iconv`接口或实用程序的内部，我决定从一点测试开始。我首先想检查在`iconv`程序中除了报告的挂起之外是否还有其他挂起。一个“信封背面”的计算让我得出结论，对于程序切换和转换的所有组合，测试每一个双字节输入组合只需要几天时间——这几天我不必花时间分析或调试`iconv`代码。

一个 bash 脚本，大约一周后，我提交了结果。转换 165 个字符集时出现挂起。除了四个例外，几乎所有错误都是在使用选项后缀(`TRANSLIT`和`IGNORE`)和`-c`开关时发生的，这些选项后缀在遇到无效字符时会隐藏错误消息。似乎很明显，这个问题是一般性的，并不对应于特定字符集转换器中的错误。

经过一些调试后，我意识到选项后缀在程序中是以字符串形式处理的，而不是一次使用一次来设置随后独占使用的相应标志。代码中的错误导致除了第一个选项之外的所有选项都被放弃。这导致在`TRANSLIT`之后通过`IGNORE`选项时挂起。

修复这个问题需要对`iconv`的选项解析逻辑做一些重大改变，但是一旦[修复了](https://sourceware.org/git/?p=glibc.git;a=commit;h=91927b7c7643)，挂起就消失了。

## 结论

最终，我修复了`iconv`中的[剩余已知挂起](https://sourceware.org/git/?p=glibc.git;a=commit;h=9a99c682144b)。我终于开始对代码库感觉舒服一点了。在 glibc 的`iconv`实现中还有一些工作要做——其中包括改进手册页。新的界面也将是有用的。

关于 C 和 C++的更多信息，请访问 [Red Hat 的主题页面](/topics/c)。

*Last updated: October 14, 2022*