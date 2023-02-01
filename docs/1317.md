# GDB Python API

> 原文：<https://developers.redhat.com/blog/2017/11/10/gdb-python-api>

在过去的几年里，GDB 已经发展到提供 Python API。这一系列的文章将着眼于用户如何使用该 API 编写 GDB，并且还将深入研究该 API 的几个特性。但是，在我们开始之前，需要上一堂历史课，看看为什么需要 API。

## 为什么选择 API？

不起眼的调试器。我们都曾在职业生涯的某个时候使用过它，有时有点害怕，有时很沮丧，但总是试图帮助解决一个讨厌的 bug。软件世界发展越来越快，调试器必须与现代编程环境同步发展。所有软件都是如此，但对于调试器来说，尤其如此。为了有用并提供一个高效的工具，调试器必须按照工程师们当时的需求来塑造自己，如果可能的话，要走在需求的前面。这是一个复杂而艰难的愿望。调试器本身是高度复杂的软件，容易受到软件缺陷和新特性的影响，并且必须适应语言和硬件不断变化的需求。如果调试器是开源的，就像 GDB 一样，那么社区也会有起起落落。GDB 的开发人员来来去去，有时社区需要一整天的时间来进行维护。那么，GDB 社区如何预测今天(和明天)的工程师需要什么呢？

在我看来，不可能。如果一切都不顺利，用户可能永远不会联系 GDB 的开发者，完成一个错误报告或者发送一封电子邮件到 GDB 的邮件列表。我们都有自己的问题要解决，有截止日期要满足，有任务要完成。然而，如果一切都不顺利，可能会给 GDB 的开发者带来一个有点沮丧的错误报告。毕竟，当用户试图解决自己代码中的错误时，用户最不希望看到的就是调试器崩溃。因此，交流可能会受到限制。GDB 开发者如何知道用户想要什么？调试器有自己的词汇表，引用复杂的概念。侏儒？精灵？劣质函数调用？这样的例子还有很多。因此，不仅有限的接触是一个问题，而且缺乏共同的词汇也会阻碍这种努力。

几年前，GDB 社区决定引入脚本 API 来解决这个问题。现在，用户不仅可以通过定义的 API 调用特定的 GDB 函数来编写 GDB 脚本，还可以通过注册在 GDB 有数据要展示时调用的脚本来改变 GDB 的输出。这两项创新改变了用户与 GDB 的互动方式。你仍然可以使用 CLI，但它也改变了 GDB 成为可编程的，并给予用户的余地，以适应自己的经验 GDB。这从根本上改变了几种范式。第一点，也是最重要的一点，它将 GDB 的观点从包装在命令行界面中的单一风格的程序发展为更加模块化和可编程的“引擎”。思考第一段中提出的问题，这即使不是一个解决方案，也提供了一个将 GDB 发展成一个新的、更具新生力量的环境的方法。如果 GDB 没有内部命令来执行用户想要的功能，那么用户可能在不了解 GDB 内部机制的情况下将该功能编程到 GDB 中。他们可以用 Python 编写函数，使用 Python API 接收来自 GDB 的函数数据，并用 Python 处理这些数据以提供他们需要的任何功能。有了 API，用户可以以一种有意义且复杂的方式定制 GDB，并将该功能以 GDB 命令的形式或作为 GDB 随后调用的编程挂钩的供应导出回 GDB。

这一系列的文章将会关注一些在 GDB 可用的原料药。这些文章不是权威性的，而是希望鼓励对这些特性的探索，并增强调试体验，使之更有成效。本文将研究的第一个 API 是 GDB 漂亮的打印机。

## Python 漂亮的打印机

### 什么是漂亮的打印机？

数据可能难以辨认。它可能是隐晦的、不可读的、误导的、令人困惑的以及介于两者之间的所有形容词。数据表示并不是这样设计的。然而，软件维护的现实和计算机存储数据的方式可能使它看起来是这样的，即使这不是数据设计者的意图。当 GDB 用于解密复杂的数据对象时尤其如此。当 GDB 被要求打印一个值时，它试图打印出一个数据结构的成员。它并不试图解释这些成员的含义。它不能。数据的意义并不隐含在对象的结构中，而是隐含在内容和结构中，只有设计者才知道。例如，对 GDB 来说，指向其他数据结构的指针仍然是指针。数据结构中的链表设计对设计者来说可能是显而易见的(或者，在这种情况下，经常是做调试的人)，但是对 GDB 来说，数据结构的含义是不透明的。这种通用的、非解释性的方法有些用处。例如，它适用于多种语言，如果数据对象足够简单明了，它就足够有用。有时它会被证明没什么用。当数据对象的成员很复杂，或者引用远程数据结构的其他成员，或者对象的含义隐含在它所包含的数据中时，GDB 就会感到困惑。下面的例子展示了一个 std::vector，在 C++程序中以通常的方式声明:

```
std::vector<int> vec = {7, 5, 16, 8};
```

以一个没有安装 std::vector Python pretty 打印机的标准 GDB 为例，得到以下 GDB 输出:

```
(gdb) print vec
\$1 = {
  <std::_Vector_base<int, std::allocator<int> >> = {
    _M_impl = {
      <std::allocator<int>> = {
        <__gnu_cxx::new_allocator<int>> = {<No data fields>}, <No data fields>}, 
      members of std::_Vector_base<int, std::allocator<int> >::_Vector_impl: 
      _M_start = 0x615c20, 
      _M_finish = 0x615c30, 
      _M_end_of_storage = 0x615c30
    }
  }, <No data fields>
```

那不是很有用。它向想要检查向量“v”的内容的用户呈现很少真正有用的数据。数据是存在的，但是您必须查看 std::vector 的内部实现。对于这样的对象(在编程社区中经常使用)，要求 std::vector 的每个用户都了解 vector 的内部是没有意义的。在上面的例子中，GDB 一般打印出 vector 类的成员。这是因为 GDB 也不知道 std::vector 的内部实现。

让我们看看当安装了一个 [GDB Python Pretty 打印机](https://sourceware.org/gdb/current/onlinedocs/gdb/Writing-a-Pretty_002dPrinter.html)并且 GDB 调用这个打印机来汇编输出时会发生什么:

```
(gdb) print vec
\$1 = std::vector of length 4, capacity 4 = {7, 5, 16, 8}
```

这是一个更有用的数据视图，包含了向量的实际内容。这个例子中使用的漂亮的打印机现在仍然存在。它是使用 Python API 为 GDB 编写的，由 libstdc++库的开发人员维护。它使用和实现的 API 是 GDB Python pretty printer 接口。这是最早引入 GDB 的 Python APIs 之一，也是最受欢迎的 API 之一。

std::vector 是有用的打印机的一个很好的例子，但是它太复杂了，无法在博客文章中解构。本文展示了漂亮的打印机在 GDB 的巨大效用以及 Python API 的强大功能。

因此，让我们编写自己漂亮的打印机吧。

### 写一个 Python 漂亮的打印机

对于我们将在本文中编写的漂亮打印机，我们将使用一个简单的数据结构。以下面两个 C 结构为例:

```
struct inner_example {
   int bar
};

struct example_struct {
   int foo;
   struct inner_example *ie;
};
```

现在，假设 example_struct 和 inner_example 以通常的方式在堆上分配。分配的结构 example_struct 存储在指针“example”中。在 GDB，打印出“example”会得到:

```
(gdb) print *example
\$1 = {
  foo = 1, 
  ie = 0x602030
}
```

注意内部结构的指针“ie”，“inner_example”显示了指针的地址。打印出那个内部结构可以这样实现:

```
(gdb) print *example->ie
\$2 = {
   bar = 0
 }
```

但是这变得很乏味，尤其是对于有很多这种指针的数据结构。因为这是我们编写的代码，我们对这些结构有内部知识，我们可以通过 Python API 教授和编程 GDB 如何打印这个值，以及具有相同类型的所有值，以提供更好的输出。在下面这个漂亮的打印机中，我们将告诉 GDB 如何解释该类型，并以一种更有用的方式打印值。

这是我们漂亮的打印机示例:

```
import gdb.printing

class examplePrinter:
   """Print an example_struct type struct"""

   def __init__(self, val):
      self.val = val

   def to_string(self):
      return ("example_struct = {foo = " + str(self.val["foo"]) +
             " {inner_example = {bar = "
             + str(self.val["ie"]["bar"]) + "}}")

def build_pretty_printer():
   pp = gdb.printing.RegexpCollectionPrettyPrinter(
   "Example library")
   pp.add_printer('Example Printer', '^example_struct$', examplePrinter)
   return pp

gdb.printing.register_pretty_printer(
    gdb.current_objfile(),
    build_pretty_printer())
```

这是安装了 pretty 打印机打印“example”时的输出。

```
(gdb) print *example
\$1 = example_struct = {foo = 1 {inner_example = {bar = 2}}
```

由于这些是用户熟悉的数据结构，并且用户理解这些数据的含义以及数据的结构，他们可以将 GDB 编程为在打印这种类型的数据时更具自省性。这取代了 GDB 更为通用的方法，即只打印存在的东西而不解释它。

分解这台漂亮的打印机，我们可以看到它是由几个步骤组成的。

#### *初始化*函数。

这是 pretty printer 的构造函数，它被传递要打印的值。在我们的示例打印机中，它将它赋给一个内部变量供以后引用。

*to _ string*功能。

当 GDB 想要打印一个值，并且它有一个为该类型注册的漂亮的打印机时，它将首先用要打印的值调用 *init* 函数。随后，它将调用 pretty printer 的 *to_string* 函数，这是打印机可以组装其输出的地方。这个函数的返回值就是 GDB 要打印的内容。所以在上面的例子中，顺序是:

```
(gdb) print *example
```

*   GDB 找到了榜样的典型。
*   GDB 搜索这种类型的漂亮打印机。
*   如果 GDB 找到了一个打印机，它就调用这个漂亮的打印机的 init 函数，并将要打印的值传递给打印机(在本例中为“example”)。
*   GDB 调用打印机的 to_string 函数。
*   GDB 打印 to_string 打印机的返回值。

打印机通过在 *init* 函数中第一次传递给它的值来访问数据。在上面的例子中，打印机将值 *val* 赋给 *self.val* 以备后用。因为 *val* 表示结构类型的值，并且 GDB 知道这种类型，Python API 允许通过该结构中定义的名称访问该结构的元素。在那个例子中，使用了 [GDB Python 值 API](https://sourceware.org/gdb//onlinedocs/gdb/Values-From-Inferior.html) 。

```
self.val["foo"]
```

相当于

```
example->foo
```

以及本例后面的

```
self.val[“ie”][“bar”]
```

相当于

```
example->ie->bar
```

注意，漂亮的打印机函数 *to_string* 必须返回一个字符串值。由漂亮的打印机的实现者来转换所有的值。

### 更复杂的打印机

有时数据不能用一行字符串来概括。上面的例子将信息压缩成一种可读性更强的格式，但并不是所有这样的结构都能以如此简洁和打包的方式压缩。pretty printing API 有另一组函数，可以帮助您扩展数据的表示，同时保持输出像以前一样简单易懂。

#### 儿童功能

以上面的例子为例，如果它是一个组装成链表的对象集合呢？很难用一个字符串来表示一个完整的列表，这将导致数据呈现更加混乱。 *children* 功能允许打印机将输出拆分成更具层次性的概念。拿上面的例子来说，我们把它修改成一个链表:

```
struct inside_example {
  int bar;
};

struct example {
  int foo;
  struct inside_example *ie;
  struct example *next;
};
```

和以前一样，链表的元素以通常的方式在堆上分配。与所有链表一样，下一个字段指向列表中的下一个元素。如果我们要查看链表中的第三个元素会发生什么？假设 GDB 的对象是第一个元素，打印出来，我们会看到:

```
(gdb) print *example
\$1 = {
  foo = 1, 
  ie = 0x602070, 
  next = 0x602030
}
```

为了得到第三个元素，我们必须:

```
(gdb) print *example->next->next
\$2 = {
  foo = 3, 
  ie = 0x6020b0, 
  next = 0x0
}
```

为了查看第三个元素的内部示例结构，我们必须:

```
(gdb) print *example->next->next->ie
\$3 = {
  bar = 44
}
```

对于任何长度或复杂度的链表来说，这都会变得令人困惑和不知所措。    *子*函数允许您对用户隐藏这些细节。该函数必须返回任何包含两个元素的 Python 元组的 Python iterable 对象。第一个元素是子元素或标签的名称，第二个元素是该元素的值。该值可以是任何值类型，Python 或直接来自 GDB。因此，对于我们的子函数，我们需要迭代链表并输出那些在链表中找到的元素。children 函数的输出示例如下:

```
Python List “Output” = 
[(label,value),
(label,value),
(label,value),
(label,value),
...]
```

但是这里有一个问题。如果链表非常长，我们就必须用 Python 复制整个链表。这有点笨拙，而且取决于链表的大小，可能会占用大量内存。我们想避免这种情况，写一个保守的打印机。解决方案是定义一个 Python 迭代器，它只在每次迭代调用时计算每个链表元素。让我们看看我们新的漂亮的打印机。

```
class examplePrinter:
     """Print an example type foo struct"""

     class _iterator:
         def __init__(self, base):
             self.base  = base
             self.count = 0
             self.end = False

         def __iter__(self):
             return self

         def next(self):
             if self.end == True:
                 raise StopIteration
             value = "example_struct = {foo = %d {inner_example = {bar = %d}}" \
                     % (self.base["foo"], self.base["ie"]["bar"])           
             item = ('[%d]' % self.count, value)
             self.base = self.base['next']
             if (self.base == 0):
                 self.end = True
             self.count = self.count + 1
             return item

     def __init__(self, val):
         self.val = val

     def to_string(self):
         return ("A linked list of example structs containing")

     def children(self):
         return self._iterator(self.val)
```

注意为了简洁，我在这里只包含了 examplePrinter 类。先前打印机中的其余代码完全相同。

这台打印机看起来可能很复杂，但只有三件事发生了变化。

*   *to _ string*功能已更改为打印汇总标签。
*   内班的包容。
*   包含了返回内部类的*子*函数。

这里最有趣的是迭代器。当 GDB 调用孩子函数时，它需要一个可迭代的 Python 对象。无论这个可迭代对象是标准的 Python 列表，还是我们的例子中的迭代器，都无关紧要。对于这个打印机来说，迭代器是更好的选择，因为对于大多数链表，我们不知道链表的长度。在这种情况下，我们不需要知道迭代器的下一个函数被调用的长度，直到它引发 StopIteration 异常。看下一个函数，我们可以看到它做了以下事情:

*   检查打印机是否已经遍历完链表。
*   如果不是，则计算元组的值部分，并存储在*值*中。
*   取元组中的*值*部分，用一个表示计数的标签构造元组并存储在元组中，*项*。
*   为下一次迭代计算链表中的下一项。
*   检查下一项是否为空，表示链表结束。
*   更新标签数量。
*   返回元组。

在 GDB 安装了 pretty 打印机后，它会产生以下输出:

```
(gdb) print *example

$1 = A linked list of example structs containing = {
   [0] = example_struct = {foo = 1 {inner_example = {bar = 42}},
   [1] = example_struct = {foo = 2 {inner_example = {bar = 43}},
   [2] = example_struct = {foo = 3 {inner_example = {bar = 44}}
 }
```

### 显示提示函数

我们在这里没有提到的一个函数是 *display_hint* 函数。这个可选函数提示 GDB 应该如何格式化输出。该函数可以返回的三个预定义值是:

'数组'

以类似数组的格式显示结果。

'地图'

这是一个将两个值映射在一起的特殊选项，表示输出类似于映射。该打印机的子打印机应该作为每次迭代的替换键和值输出。

字符串

这表明输出是类似字符串的，GDB 应该将输出视为字符串。

就这样结束了！我希望你喜欢这个 GDB 漂亮印刷商的快速浏览，我希望你能在未来的文章中再次加入我的行列。

*Last updated: August 9, 2018*