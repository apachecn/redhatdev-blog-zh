# 为什么 Windows 和 Linux 行尾不对齐(以及如何修复)

> 原文：<https://developers.redhat.com/blog/2021/05/06/why-windows-and-linux-line-endings-dont-line-up-and-how-to-fix-it>

我最近写了几个自动填充数据库的脚本。具体来说，我在 Kubernetes 集群的一个容器中运行 Microsoft SQL Server 好吧，它是 [Red Hat OpenShift](/products/openshift/overview) ，但它仍然是 [Kubernetes](/topics/kubernetes) 。这一切都很有趣，直到我开始混合使用 [Windows](/blog/category/windows/) 和[Linux](/topics/linux)；我是在我的 Windows 机器上开发的，但是显然容器运行的是 Linux。这时，我发现了图 1 所示的错误。与其说是错误，不如说是错误的输出。

[![Weird line endings in SQL statement output.](img/fa04904fb3339e9c30625eda93221240.png "crlf_errant_output")](/sites/default/files/blog/2021/04/crlf_errant_output.png)

Figure 1: Errant output from an SQL statement.

究竟是什么？下面是我用来填充表格的 CSV 数据:

```
1,Active
2,Inactive
3,Ordered
4,Billed
5,Shipped

```

下面是我用于相同目的的 T-SQL 代码:

```
BULK INSERT dbo.StatusCodes FROM '/tmp/StatusCodes.csv' WITH (FORMAT='CSV',FIELDTERMINATOR=',',KEEPIDENTITY);
GO
SELECT * FROM dbo.StatusCodes;
GO

```

这是怎么回事？

## TL；DR:行尾

这是行尾。他们是问题所在。

具体来说，Windows 和 Linux 处理行尾的方式不同。要了解原因，我们需要追溯历史。

## 阿斯多夫 KL

用过手动打字机吗？好吧，好吧...受够了“那是旧的！”笑话。图 2 展示了。

[![typewriter](img/b460186c89c2324b73533d5a2f78eb63.png "typewriter-1138293_640")](/sites/default/files/blog/2021/04/typewriter-1138293_640.png)Image by <a href="https://pixabay.com/users/blende12-201217/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1138293">Gerhard G.</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1138293">Pixabay</a>

Figure 2: What a manual typewriter looks like.

容纳橡皮滚筒的打字机机构被称为*滑架*，因为它运送纸张。(那个橡胶圆筒在技术上被称为*压板*，但是请记住我，因为我使用了诗意的许可并使用了“carriage”)

当您键入时，马车会向左移动。当你到达纸的边缘时，你使用最左边的大杠杆使马车回到起始位置；也就是说，你执行一个*回车*。此外，随着控制杆的移动，它将纸张向前推进一行，这被称为*换行*。

当你做这两个动作时，你得到“回车加换行”，有时缩写为 CRLF 或 CR/LF。你*可以*移动小车不进一行，你*可以*前进一行不移动小车。它们是两种截然不同且独立的动作，但任何一个掌握了手动打字机的人都知道，它们通常是以一种快速、充满激情的运动方式完成的，类似于最高级别的桌面体操。(当我对打字进行浪漫化时，请原谅我更多的诗意。)

## 电传打字机

与此同时，在自动化领域，电传打字机变得非常流行。这使得文本可以通过电话线在世界范围内传输。但是长途电话很贵，所以尽量减少发送的时间和数据是最重要的。因此，决定只使用一个字符作为回车符和换行符，即所谓的*新行字符*。你在代码中看到的是“`\n`”。那时，你为每一个字节付费，所以降低成本很重要。

伙计们，我们说的是 300 波特的调制解调器。想想看。每秒 300 比特；*三百*。现在，我们希望千兆位无处不在。

## 回到行尾

原因不重要:Windows 选择了 CR/LF 模式，而 Linux 使用的是`\n`模式。因此，当你在一个系统上创建一个文件并在另一个系统上使用它时，有趣的事情就发生了。或者，在这种情况下，两个小时的调试在疯狂中结束，我在考虑木工的新职业。

## 快速修复 Linux 和 Windows 行尾

对这些不兼容的行尾的快速修复非常简单:我修改了我的 T-SQL 以包含 ROWTERMINATOR 规范，如下所示:

```
BULK INSERT dbo.StatusCodes FROM '/tmp/StatusCodes.csv' WITH (FORMAT='CSV',FIELDTERMINATOR=',',ROWTERMINATOR = '\r\n',KEEPIDENTITY);
GO
SELECT * FROM dbo.StatusCodes;
GO

```

当从我的 Windows 机器上传我的 CSV 文件时，这是有效的。当从我的 Linux 机器上传时，我使用下面的代码，其中 ROWTERMINATOR 是简单的换行符:

```
BULK INSERT dbo.StatusCodes FROM '/tmp/StatusCodes.csv' WITH (FORMAT='CSV',FIELDTERMINATOR=',',ROWTERMINATOR = '\n',KEEPIDENTITY);
GO
SELECT * FROM dbo.StatusCodes;
GO

```

很简单，但是除非您了解它，否则您要么得到奇怪的结果，要么得到一些看似无关的错误消息。所以，建议你。例如，如果我尝试在我的 Linux 环境中使用特定于 Windows 的命令(其中 ROWTERMINATOR 是“`\r\n`”)，我会得到以下错误:

```
Msg 4879, Level 16, State 1, Server mssql-1-h2c96, Line 2
Bulk load failed due to invalid column value in CSV data file /tmp/StatusCodes.csv in row 1, column 2.
Msg 7399, Level 16, State 1, Server mssql-1-h2c96, Line 2
The OLE DB provider "BULK" for linked server "(null)" reported an error. The provider did not give any information about the error.
Msg 7330, Level 16, State 2, Server mssql-1-h2c96, Line 2
Cannot fetch a row from OLE DB provider "BULK" for linked server "(null)".
Id statusCodeDescription
----------- ---------------------

```

## 这一切意味着什么？

结果是这样的:当你在 Windows 和 Linux 中使用一个文件时，你可能会看到一些小问题和奇怪的行为。意识到这一点就没事了。

访问我的 GitHub 库 [NetCandyStore](https://github.com/donschenck/netcandystore) 获取本文中引用的所有代码。

*Last updated: October 14, 2022*