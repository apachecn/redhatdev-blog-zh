# 2018 年 11 月 ISO C++会议行程报告(核心语言)

> 原文：<https://developers.redhat.com/blog/2019/02/15/november-2018-iso-c-meeting-trip-report-core-language>

2018 年 11 月的 ISO C++标准会议在加州圣地亚哥举行。像往常一样，Red Hat 派了我们三个人去参加会议:我(核心语言工作组)、Jonathan Wakely(图书馆工作组[LEWG])和 Thomas Rodgers(并发和并行研究组[SG1])。我觉得这次会议是富有成效的，尽管一些曾被认为是 C++20 的特性现在受到了质疑。

以下是会上接受的 C++新特性:

*   扩展的 constexpr:常量表达式现在可以包含 [try 块](http://wg21.link/p1002r1)(在 constexpr 函数中)、 [dynamic_cast](http://wg21.link/p1327r1) 和 typeid，但前提是它们不会抛出异常。抛出一个表达式仍然会使表达式变得非常数。常量表达式现在也可以[改变常量求值期间创建的联合的活动成员](http://wg21.link/p1330r0)。
*   [char 8 _ t](http://wg21.link/p0482r6):UTF-8 代码单元的一种特殊类型，也没有 C char 类型的混淆问题。
*   [immediate functions(" const eval ")](http://wg21.link/p1073r3):const expr 函数的一种更强形式，它总是立即计算一个常数值。我仍然不相信这是与普通 constexpr 函数的足够有用的区别，但其他人似乎对此很感兴趣。
*   [STD::is _ constant _ evaluated()](http://wg21.link/p0595r2):允许 constexpr 函数在常量求值期间使用一个实现，在运行时求值期间使用一个更高效但非 constexpr 的实现。
*   [嵌套内联名称空间](http://wg21.link/p1094r2):为 C++嵌套名称空间定义特性增加了对声明内联名称空间的支持。
*   [【Constrained auto】](http://wg21.link/p1141r2):最重要的是从概念 ts 中以稍微不同的语法重新引入缩写的函数模板；不是用一个类型概念的名字来声明一个被约束的泛型类型的函数参数，而是用后面跟有“auto”的概念来声明，例如:

    ```
    auto f(Copyable auto x) { return x; }
    ```

以下是我们在会上讨论的各种其他提案:

*   P0881R3 ，一个堆栈跟踪库的新提议:这看起来像是在反射方面走得太远了；这个设计需要更多的时间来烘焙。
*   [P1103R2](http://wg21.link/p1103r2) ，合并模块:统一模块设计继续取得进展，预计将在下次会议上纳入工作文件。
*   Two papers about operator <=> ("spaceship"), P1185 and P1186:
    *   指出仅仅调用< = >操作符的==操作符要比为子对象编写的 call ==操作符慢得多。设计仍然有些变化，但似乎有一个强烈的共识，我们希望改变这一点，并将类类型的非类型模板参数改为依赖==而不是< = >。
    *   [1186](http://wg21.link/p1186) 提议允许默认的< = >使用现有的<和==运算符，因为大多数现有的类本身没有< = >。但这遇到了麻烦；更多详情请见[作者的帖子](https://brevzin.github.io/c++/2018/11/12/improve-spaceship/)。

    随着这些问题的出现，人们变得不确定<=>是否真的准备好成为 C++20 的一部分，但在我看来，还有足够的时间来解决问题。

*   使 type_info::operator== constexpr:对我来说这似乎是一个明显的修正。

我们甚至在报纸之间找到时间讨论提交的问题。委员会之外的人对其中的许多并不感兴趣，但是 [issue 2362](http://wg21.link/cwg2362) 认为 __func__ 应该是 constexpr。Core 倾向于向相反的方向移动，将它的类型从数组改为指针，以避免名字的大小在常量表达式中变得可用。现在回想起来，我不确定这是个好主意；有很多方法可以违反 ODR，我不确定这种方法会坏到足以阻止人们想要的用途。我们把这个问题称为输入进化。

下一次会议将于二月份在夏威夷的凯卢阿-科纳举行。

*Last updated: February 14, 2019*