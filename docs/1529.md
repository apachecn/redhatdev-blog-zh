# 行程报告:2017 年 4 月 WG14 会议

> 原文：<https://developers.redhat.com/blog/2017/05/17/trip-report-april-2017-wg14-meeting>

### 概观

4 月 3 日那一周，我参加了 C 标准化委员会 WG14 在马卡姆召开的会议。马卡姆是多伦多的郊区，向北开车大约 40 分钟。不像多伦多本身，它不是一个特别有趣的目的地。我们有四天的雨，接着是雪，寒冷的温度，还有风，这是在室内度过时间的完美选择，并且很容易抵制任何去观光的诱惑。

### 位置

会议由 IBM 在他们的多伦多软件实验室举办，尽管它的名字叫 T1，但它位于马卡姆而不是多伦多。该实验室是由几栋附属建筑组成的综合体，可容纳约 2，500 名员工(大约 80%已满)。我们的主人很友好，给我们做了一个关于与 [IBM z 系统](https://www-03.ibm.com/systems/z/)操作环境互动的演示，随后参观了计算中心，就纯粹的计算能力而言，这是加拿大最大的计算中心。

### 参与

参加会议的有房间里的大约 20 人，在不同时间打电话的多达 3 人，分别代表 [CERT](http://cert.org/) 、[思科](http://www.cisco.com)、 [GrammaTech](https://www.grammatech.com/) 、 [IBM](http://www.ibm.com) 、 [INRIA](https://www.inria.fr/en/) 、 [LDRA](http://www.ldra.com/en/) 、 [LLNL](https://www.llnl.gov/) 、[甲骨文](http://oracle.com/)、[常年](https://www.peren.com/)、[梅花堂](http://www.plumhall.com/)、[红](http://www.redhat.com/)

### 目标

会议的主要目标是完成 C11 的 2017 年“技术勘误”,并在 C2X 方面取得更多进展。我将“技术勘误”放在引号中，因为由于 ISO 的限制，它不会真正成为一个 TC，而是一个全新的标准，尽管它仅限于 TC 更改，只有错误修复，没有技术更改，这意味着没有新功能。

### 缺陷报告处理

在四天的过程中，我们审查了开放的缺陷报告，关闭了那些准备关闭的报告，并花了几个小时讨论剩余的和任何新的缺陷将会遵循的过程。我们甚至花了更多的时间来讨论一些 C11 缺陷，我们中的一些人认为应该在 2017 年 TC 中修复这些缺陷，但这些缺陷只能通过技术(即实质性)更改来解决，而不是简单的澄清。举一个极端的例子，我们花了两天多的时间才习惯于删除三个单词作为 [DR 485](http://www.open-std.org/jtc1/sc22/wg14/www/docs/summary.htm#dr_485) 的解决方案(对现有的实现或程序没有任何影响)。C11 缺陷报告的会前状态在 [N2109](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2109.htm) 文件中。最终状态将在几周后的会后邮件中公布。

### 其他论文

我们还听取了 LLNL 代表的发言，他对即将更新的 CPLEX 规范表达了一些严重的担忧。问题在于 CPLEX 代码与写入其他 API 的其他代码(如 C11 线程、Pthreads 或 OpenMP)的互操作性。详见 [N2131](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2131.htm) 。鉴于这些问题及其严重性，CPLEX 研究小组主席 Clark Nelson 受命就如何解决这些问题向第 14 工作组提供指导。根据 Clark 的说法，CPLEX 研究小组在过去的 6 个月到一年中基本上是不活跃的，这使得它不确定这对该规范的未来意味着什么。

### C2X 提案审查

除了缺陷处理，我们还讨论了相对少量的 C2X 特性的提议，并在 C 标准被移植到 [LaTeX](https://www.latex-project.org/) 后花了一些时间回顾了 C 标准。新文档非常漂亮，并且更好地利用了链接甚至颜色等现代功能。

### 浮点建议

除了一些小的例外，其余的论文都集中在提议合并部分*TS 18661——C*的浮点扩展。

[N2117](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2117.pdf) : *TS 18661-3 -互换和扩展型*。建议添加 ISO 60559 推荐的交换和扩展浮点类型，如 GCC 的 _FloatN 和 _FloatNx。

[N2118](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2118.pdf) 和 [N2119](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2119.pdf) : *TS 18661-4 —数学和归约函数*。一对合并了许多函数以支持当前 IEC 60559 推荐的数学运算的提议，例如 exp10 和 rsqrt。

[N2120](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2120.pdf) 、 [N2121](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2121.pdf) 、 [N2122](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2122.pdf) 和 [N2123](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2123.pdf) : TS 18661-5 —评估格式 pragma、优化控制 pragma、可重现结果和备选异常处理。按照 ISO 60559 的建议，提出以编译指令的形式添加旋钮来控制浮点表达式求值的各个细节的论文。在这些方法中，最后一种方法被证明是有争议的，因为它使用了带外方法来拦截浮点异常，并且编译指示与周围的代码进行了交互。pragmas 的意图是实现能够忽略那些它们没有实现的，而不会影响程序的正确性。最后一项提案中提出的实用程序将违背这一意图。

### 其他提议

除了浮点建议之外，提交给委员会的其他几篇论文也值得一提:

[n 2101](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2101.htm):*_ _ has _ include for C*。该论文建议在 C2X 中添加 C++ __has_include 特征，以便能够确定指定的头是否存在。该小组支持在 C2X 采用这一功能。

[N2129](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2129.pdf) : *不赞成 __LINE__* 。本文指出了 __LINE__ 宏的一个令人惊讶的问题，并建议用其他一些更安全的宏来代替它。由于现有代码中普遍存在宏，该提议几乎被一致否决。

### 未来会议

下一次会议暂定于 11 月的第一周在新墨西哥州的阿尔伯克基举行。随着 C 2017 的最终确定，WG14 将把所有资源集中在 C2X。如果您对语言的增强有想法或建议，这是写下来并提交给您组织的代表或您的国家机构代表以便在 10 月邮寄的最佳时机。

之后的下一次会议将于 2018 年 4 月举行，由捷克共和国的 [Red Hat Brno office](https://www.redhat.com/en/global/czech-republic) 主办。

* * *

**开发者现在可以通过[developers.redhat.com](https://developers.redhat.com/)注册和下载，免费获得[Red Hat Enterprise Linux Developer Suite](https://developers.redhat.com/products/rhel/download/?intcmp=7016000000124eKAAQ)用于开发目的。**

*Last updated: November 15, 2018*