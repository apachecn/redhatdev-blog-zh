# ISO C++标准会议上的 red Hat(2016 年 11 月，伊萨夸，2017 年 2 月，科纳):图书馆

> 原文：<https://developers.redhat.com/blog/2017/06/30/red-hat-at-the-iso-c-standards-meetings-november-2016-issaquah-and-february-2017-kona-library>

我参加了最近的 Issaquah 和 Kona ISO C++标准会议，代表 Red Hat 和 GCC 项目，并帮助完成 C++17 标准。像往常一样，我大部分时间都花在了库工作组(LWG)会议上，但也参加了一个专注于文件系统库的小组，下面会详细介绍。

在国家机构(NB)对 C++17 草案投票之后，LWG 有大量的意见需要处理。一些问题在伊萨夸得到解决，但许多问题只在伊萨夸得到审理，并被搁置到科纳会议上处理。这些会议带来的一些更有趣的变化是:

*   API 对新的 std::any、std::optional 和 std::variant 类型进行了更改，使它们彼此更加一致。

对 std::tuple_size <const t="">和节点句柄进行了更改，以便更好地与核心语言中新的结构化绑定特性配合使用。</const>

*   std:: regex 的新多行语法选项(相当于 ECMAScript 中的 multiline 属性)。
*   让 std::char_traits 和 std::string_view 中的更多函数在常量表达式中可用。
*   一个新的 std::is_aggregate 类型特征。
*   一个新的 std::byte 类型，它是一个非整数类型，与 unsigned char 具有相同的表示形式，但不支持算术运算或隐式转换为整数。
*   禁止使用右值参数调用 std::addressof。
*   允许将 nullptr 写入 ostreams。

C++11 中的一些新特性在 C++17 中已经过时:

*   std::result_of，它被不太容易出错的 std::invoke_result 所取代。std::is_callable 特征(C++17 中的新特征)被重命名为 STD::is _ invokable，同时检查到另一种类型的可转换性的功能被拆分到 STD::is _ invokable _ r 中(对应的 is _ not row _ invokable 和 is _ nothrow _ invocable _ r 特征也用于检查 noexcept 保证)。

不推荐 std::result_of 的原因是它的语法对于非专业人士来说总是一个陷阱，在某些情况下它会给出错误，或者在某些奇怪的情况下给出错误的答案。现在你可以使用 invoke_result <callabletype argtypes...="">而不是使用 result_of <callabletype>，它在所有情况下都能正常工作。请参见 [P0604R0](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0604r0.html) 了解此次变更的全部理由。</callabletype></callabletype>

*   代码转换方面 std::codecvt_utf8、std::codecvt_utf16 和 std::codecvt_utf8_utf16 已被弃用，因为它们的接口中存在几个问题，并且它们的预期行为规范不明确。实用程序 std::wstring_convert 和 std::wbuffer_convert 也已经被弃用，因为它们唯一真正的用例是那些 codecvt 方面。这不赞成 C++标准库中仅有的用于在不同 Unicode 编码之间进行转换的工具，但目的是尽快添加更易于使用、支持所有编码并具有更好 API 的新特性。更多原理见 [P0618R0](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0618r0.html) 。

基于 NB 的评论，两个新的 C++17 特性被从草案中删除:

*   std::default_order 特征(由 P0181R1 添加)被取消，使 std::lock_guard 成为变量模板的更改被恢复，并添加了新的 std::scoped_lock(参见 P0156R2)。在这两种情况下，这样做是因为这些特性引入了 ABI 不兼容性，而且这些好处并没有被认为足够重要，以至于如果用户使用 C++17，就应该强制他们进行 ABI 更改。

在 C++17 中加入的 STL 算法的新并行版本有几个变化，源自并行 TS。与 TS 相比，Kona C++17 之后的草案:

*   移除 inner_product 的并行形式，添加 transform_reduce 的新重载，并对 transform_inclusive_scan 和 transform_exclusive_scan 的参数进行重新排序。
*   放宽要求以允许通过并行算法制作额外的副本，并放宽复杂性要求以允许更优化的实现。
*   将所有接受输入迭代器的并行算法改为至少需要前向迭代器，这提供了多遍保证，因此允许并行操作范围内的不同子序列。

最后，对新的文件系统库进行了大量的修改，这些修改源于文件系统 TS。LWG 的一些人从主组中分离出来，专注于文件系统问题，这也是我花了大部分时间的地方。该小组包括从事 GCC、Boost、Windows 和 IBM 实现的人员，以及许多更棘手问题的提交者，因此委员会中的大多数相关人员都参与了该小组。

在这一周，小组向图书馆发展工作组(LEWG)提出了一些想法，以获得他们对我们提出的设计变更的输入和批准。在 Issaquah 会议结束时，我们就如何解决其中一些问题向 LWG 提出了一些建议。在两次会议之间，就更复杂的问题开展了工作，并根据我们建议的决议编写了一份文件，由 LWG 在 Kona 会议期间审查，并由整个委员会批准。

对文件系统库所做的许多更改对用户来说并不明显，只是更改了规范，或者清理了一些错误处理或死角。一组重要的规范更改删除了一些隐含的假设，因此除了 POSIX 和 Windows 文件系统上的传统目录之外，该规范还可以应用于大型机中的“数据集”。

一个更容易被用户看到的变化是如何将文件名分解成一个词干和一个扩展名。在原始设计中，文件名类似于“.bashrc“将被分解成一个空的主干和一个扩展”。bashrc”，这可能会让习惯于 POSIX 类环境的人感到惊讶。这一点已经改变。“bashrc”是词干，没有扩展名(例如，“. bashrc.bak”的扩展名是“.bak”，如您所料)。

最显著的变化是重新设计了路径分解如何确定路径的文件名部分。原来，表达式路径("/foo/")。filename()将返回“.”(这与路径("/foo/"的结果相同).filename()返回)。这样做是为了区分像“/foo/”和“/foo”这样的路径，因为结尾的斜杠意味着它必须指向一个目录，就像“/foo/”确实如此。然而，这导致了一些令人惊讶的属性，比如 path("/foo/")。has_filename()为 true，因为它的文件名为"."、和路径(“/”)。has_filename()为真。其实 p.has_filename()对任何非空路径都是真的！

路径中“文件名”部分的定义意味着 remove_filename()成员函数会给出一些令人惊讶的结果。因为每个非空路径都有一个文件名，所以调用 remove_filename()更像是一个“pop back”操作，它会不断地删除路径中的一个部分，直到一个也不剩。

Kona 中接受的更改意味着路径的“文件名”部分是从最后一个斜杠到路径末尾的路径部分。所以:

路径("/foo/")。filename() == " "(即空路径)

路径("/foo/").filename() == "。"

路径("/foo ")。filename() == "foo "

有了这些新的语义，remove_filename()的含义就更有意义了，因为它只是删除了文件名部分。所以路径("/foo/")。remove_filename()没有效果，但是 path("/foo ")。remove_filename()仍然像以前一样产生“/”。

调整描述路径的语法并更新成员函数的规范以满足新的语义(不引入任何新的不一致或意外)是一项相当复杂的任务。为了确保新的措辞确实表达了正确的意思，我们进行了仔细的重新设计和大量的检查，并且我们都同意“正确的事情”就是我们几个月前在伊萨夸会议上决定的事情！

对文件系统设计的大部分更改可以在 Kona 批准的 [P0492R2](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0492r2.html) 中找到。正如标准化工作中经常出现的情况一样，可能需要几年时间才能知道我们是否做出了正确的选择。有希望从尝试过的&测试助推力偏离。文件系统语义将使 C++17 文件系统库更容易学习和使用。

* * *

**下载 Kubernetes** [**备忘单**](https://developers.redhat.com/promotions/kubernetes-cheatsheet/)****跨主机集群自动部署、扩展和操作应用容器，提供以容器为中心的基础设施。****

***Last updated: June 29, 2017***