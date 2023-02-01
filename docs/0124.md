# 优化 Clang 编译器的行到偏移量映射

> 原文：<https://developers.redhat.com/blog/2021/05/04/optimizing-the-clang-compilers-line-to-offset-mapping>

最近，我一直在努力为 [C 和 C++](/topics/c) 提高 [Clang 编译器](/blog/category/clang-llvm/)的速度。当我分析一个大文件的 Clang 预处理步骤时，有一个函数很快就突出来了:

```
clang::LineOffsetMapping::get(llvm::MemoryBufferRef Buffer, llvm::BumpPtrAllocator &Alloc)
```

这个函数基本上分配了一个向量(通过`Alloc`)，该向量将行号映射到文件中的偏移量(在`Buffer`中加载)。这是一个令人惊讶的独立函数，因此很容易在微基准中提取它并开始优化之旅。这篇文章算是那次旅行的*日志*。

## 问题，和幼稚的实现

我这样描述`LineOffsetMapping`函数的目的:

> 给定一个加载到字符串`Buffer`中的文件，创建一个大小为`N`的向量`LineOffsets`，其中`N`是`Buffer`中的行数，因此`LineOffsets[I]`包含该行开始处的字节偏移量。

因为跨平台很重要，所以让我们加入一些乐趣:

> 一个文件可以包含任意组合的行分隔符，包括`\n`、`\r`和`\r\n`。

一个简单的实现——这实际上是 clang 12.0.0 中使用的实现——可能是:

```
std::vector<unsigned> LineOffsets;
LineOffsets.push_back(0);

const unsigned char *Buf = (const unsigned char *)Buffer.data();
const std::size_t BufLen = Buffer.size();

unsigned I = 0;
while (I < BufLen) {
  if (Buf[I] == '\n') {
    LineOffsets.push_back(I + 1);
  } else if (Buf[I] == '\r') {
    // If this is \r\n, skip both characters.
    if (I + 1 < BufLen && Buf[I + 1] == '\n')
      ++I;
    LineOffsets.push_back(I + 1);
  }
  ++I;
}

```

这很简单:读取文件，一次一个字节，检查其内容，并记录新行。但是我们能做得更好吗？

## 手动轮廓导向优化

因为 Clang 的输入是源代码，换行符与非换行符的比例将有利于后者。例如，在 LLVM 源代码本身中:

```
$ find llvm -name '*.cpp' -exec cat {} \; | tr -cd '\n' | wc -c
2130902
$ find llvm -name '*.cpp' -exec cat {} \; | wc -c
78537977
```

这大约占新行的 2%。

我们可以使用`__builtin_expect(expression, value)`内在属性来指导优化过程，从而将该属性考虑在内。我们还可以利用 ASCII 表中的`\n`和`\r`之间的两个字符——垂直制表符(`0x0B`和换页符(`0x0C`)——相当罕见的巧合，并使用快速检查来捕捉这两个换行符:

```
unsigned I = 0;
while (I < BufLen) {
  // Use a fast check to catch both newlines
  if (__builtin_expect((Buf[I] - '\n') <= ('\r' - '\n'), 0)) {
    if (Buf[I] == '\n') {
      LineOffsets.push_back(I + 1);
    } else if (Buf[I] == '\r') {
      if (I + 1 < BufLen && Buf[I + 1] == '\n')
        ++I;
      LineOffsets.push_back(I + 1);
    }
  }
  ++I;
}

```

模糊检查在大多数情况下都会失败，编译器知道这一点，这将导致更高效的代码。

**注意**:使用更快的检查很有诱惑力，`Buf[I] <= '\r'`。然而，制表符具有 ASCII 码`0x09`，并且会被检查所捕获。对于使用制表符的人来说，这将是一个很好的惩罚，但不是一个非常温和的举动。

## 添加快速路径

作为一名 [Linux](/topics/linux) 用户，我发现为这些永远不会出现在我的代码库中的 DOS 和 OS X 风格的换行符付费令人沮丧。可以更新算法来快速扫描`\r`字符，如果没有就使用快速路径。如果文件确实有这个字符，并且使用一致的换行符样式，那么这个字符会很快被找到，所以成本应该很低:

```
if(!memchr(Buf, '\r', BufLen)) {
  while (I < BufLen) {
    if (__builtin_expect(Buf[I] == '\n', 0)) {
      LineOffsets.push_back(I + 1);
    }
    ++I;
  }
}
else {
// one of the version above... or below
}

```

## 翻阅历史

Clang 源代码包含一个[优化提示](https://github.com/llvm/llvm-project/blob/release/12.x/clang/lib/Basic/SourceManager.cpp#L1255):在某个时候，`clang::LineOffsetMapping::get`有一个 SSE 版本，在修订版 [d906e731](https://github.com/llvm/llvm-project/commit/d906e731ece9942260df4097ed8e8b4cbfa70d32) 中，为了可维护性起见，它被移除了。

这个版本努力使用一致的负载，这是旧架构的一个重要优化点。从那时起，一些现代英特尔架构上的非对齐负载的性能成本已经降低，因此让我们尝试一个更易于阅读和维护的 SSE 版本:

```
#include <emmintrin.h>

// Some renaming to help the reader not familiar with SSE
#define VBROADCAST(v) _mm_set1_epi8(v)
#define VLOAD(v) _mm_loadu_si128((const __m128i*)(v))
#define VOR(x, y) _mm_or_si128(x, y)
#define VEQ(x, y) _mm_cmpeq_epi8(x, y)
#define VMOVEMASK(v) _mm_movemask_epi8(v)

const auto LFs = VBROADCAST('\n');
const auto CRs = VBROADCAST('\r');

while (I + sizeof(LFs) + 1 < BufLen) {
  auto Chunk1 = VLOAD(Buf + I);
  auto Cmp1 = VOR(VEQ(Chunk1, LFs), VEQ(Chunk1, CRs));
  unsigned Mask = VMOVEMASK(Cmp1) ;

  if(Mask) {
    unsigned N = __builtin_ctz(Mask);
    I += N;
    I += ((Buf[I] == '\r') && (Buf[I + 1] == '\n'))? 2 : 1;
    LineOffsets.push_back(I);
  }
  else
    I += sizeof(LFs);
}

```

除了 SSE 之外，这个版本还使用了`__builtin_ctz`内置函数来计算尾随零的数量，从而允许直接访问匹配的字符。因为我们知道这个角色要么是`\n`要么是`\r`，所以也有可能避免一些比较，而使用一个可能在 x86 上被优化为条件`mov`(又名`cmov`)的一行程序。

## 比特黑客

我们试图创建的函数与普通的`memchr`共享属性。让我们向古代致敬，并检查一下 [glibc 实现](https://github.com/lattera/glibc/blob/master/string/memchr.c#L48)。它使用了一种类似于矢量化的技术，但是具有较少的可移植性问题(以一些脑力劳动为代价)，有时被称为*多字节字打包*。

Bit hack 爱好者已经知道美味的 [Hacker 的乐趣](https://en.wikipedia.org/wiki/Hacker%27s_Delight)和它的密友 [Bit Twiddling Hacks](http://graphics.stanford.edu/~seander/bithacks.html) 。它们包含许多与多字节单词相关的有用方法，包括一种[确定一个单词是否在 m 和 n](http://graphics.stanford.edu/~seander/bithacks.html#HasBetweenInWord) 之间有一个字节的方法。让我们将该技术应用于我们的问题:

```
template <class T>
static constexpr inline T likelyhasbetween(T x, unsigned char m,
unsigned char n) {
// see http://graphics.stanford.edu/~seander/bithacks.html#HasBetweenInWord
  return (((x) - ~static_cast<T>(0) / 255 * (n)) & ~(x) &
          ((x) & ~static_cast<T>(0) / 255 * 127) + ~static_cast<T>(0) / 255 * (127 - (m))) &  
          ~static_cast<T>(0) / 255 * 128;
}

uint64_t Word;

// scan sizeof(Word) bytes at a time for new lines.
// This is much faster than scanning each byte independently.
if (BufLen > sizeof(Word)) {
  do {
    memcpy(&Word, Buf + I, sizeof(Word));
#if defined(BYTE_ORDER) && defined(BIG_ENDIAN) && BYTE_ORDER == BIG_ENDIAN
    Word = __builtin_bswap64(Word); // some endianness love
#endif

    // no new line => jump over sizeof(Word) bytes.
    auto Mask = likelyhasbetween(Word, '\n' - 1, '\r'+1 );
    if (!Mask) {
      I += sizeof(Word);
      continue;
    }

    // Otherwise scan for the next newline - it's very likely there's one.
    // Note that according to
    // http://graphics.stanford.edu/~seander/bithacks.html#HasBetweenInWord,
    // likelyhasbetween may have false positive for the upper bound.
    unsigned N = __builtin_ctzl(Mask) - 7;
    Word >>= N;
    I += N / 8 + 1;
    unsigned char Byte = Word;
    if (Byte == '\n') {
      LineOffsets.push_back(I);
    } else if (Byte == '\r') {
      // If this is \r\n, skip both characters.
      if (Buf[I] == '\n')
        ++I;
      LineOffsets.push_back(I);
    }
  }
  while (I < BufLen - sizeof(Word) - 1);
}
```

代码有点长，依赖于`likelyhasbetween`函数的一个未记录的属性:它将保存一个搜索值的字节设置为`0x80`。我们可以使用这个标记结合`__builtin_ctzl`(你已经见过的`__builtin_ctz`内置的`long`副本)来指出可能的匹配，就像我们对 SSE 版本所做的那样。

## 计算数字

到目前为止，我们只探索了优化策略，没有进行一次运行、分析或任何事情。这当然不是旅行实际发生的方式。我建立了一个[小型回购](https://github.com/serge-sans-paille/mapping-line-to-offset)来收集各种实现(和一些额外的东西)。它使用 [SQLite 合并](https://www.sqlite.org/amalgamation.html)作为输入，为基准测试和验证一些实现提供了自动化。

以下是在我的笔记本电脑上运行的结果，运行的是 Core i7-8650U 和 gcc 8.2.1(时间单位为毫秒，平均运行 100 次):

```
ref: 11.37
seq: 11.12
seq_memchr: 6.53
bithack: 4
bithack_scan: 4.78
sse_align: 5.08
sse: 3
sse_memchr: 3.7

```

`ref`是参考实现。额外的分析提示和`seq`中的模糊检查稍微改善了这种情况，但是幅度很小。带有对`\r`的`memchr`检查的快速路径产生了令人印象深刻的加速，但只针对一种特定情况。比较了两个版本的 bit hacks，表明本文中介绍的`bithack`版本几乎与 SSE 版本一样快。试图匹配对齐约束的遗留 SSE 版本`sse_align`的性能不如`bithack`版本，因为匹配对齐是有成本的。在`sse_memchr`中试图在 SSE 版本上使用快速路径被证明是适得其反的。

有趣的是，但不奇怪的是，配置很重要。在 Mac Mini 上(在 arm64 上)，使用 Apple clang 版本 12.0.0，结果完全不同:

```
ref: 6.49
seq: 4
seq_memchr: 4
bithack: 2
bithack_scan: 2.05

```

对`memchr`的调用在那里没有太大的影响。来自`seq`的 profile-guided 版本性能明显更好，两个`bithack`版本的性能都是参考版本的三倍。

## 结论

一个[补丁已经提交给 LLVM](https://reviews.llvm.org/D99409) 使用`bithack`版本。虽然它的性能不如 SSE 版本，但评审人员强调了拥有一个始终优于参考实现的架构不可知版本的好处，而`bithack`版本符合这些标准。

有关 Clang 及其 LLVM 后端的更多信息，请访问[红帽开发者主题页面](/blog/category/clang-llvm/)。

## 承认

我要感谢 Adrien Guinet 对这篇文章的宝贵而准确的评论。

*Last updated: October 14, 2022*