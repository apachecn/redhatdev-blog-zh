# 如何忽略 Git 中的文件？gitignore

> 原文：<https://developers.redhat.com/blog/2020/02/25/how-to-ignore-files-in-git-without-gitignore>

当文件应该保存在本地时，Git 有一个方便的特性可以防止意外的文件签入。当您只想签入源代码时，显而易见的选择是编译后的二进制文件。其他候选文件是带有本地配置的文件。

可以将所有这些文件和路径放入项目中的一个`.gitignore`文件中。为了持久化这些更改(并与项目中的合作者共享公共文件内容)，通常像项目中的任何其他文件一样将`.gitignore`文件添加到 Git 中。

## 问题是

不幸的是，这种方法有局限性。

将本地文件放入`.gitignore`文件只适用于所有协作者共有的项目，比如项目中的文件和目录。想象一下，如果成百上千的合作者将他们的特定路径放入`.gitignore`会发生什么。这种做法会造成巨大的混乱和流失。

添加文件名和路径还会泄露不应公开的信息。例如，它可能会泄露客户信息(这个问题不仅与文件内容有关，还与客户的姓名有关，客户的姓名可能会泄露作为元数据的信息)。

不签入`.gitignore`文件也是一种痛苦。当一个人切换分支或者更新本地工作树时，他必须经常[隐藏](https://git-scm.com/docs/git-stash)文件(带有本地更改)，切换分支，或者更新然后取消隐藏(潜在地带有合并冲突)。

## 帮助是可用的

幸运的是，Git 提供了替代方法来防止意外的文件签入。例如，文件`.git/info/exclude`的工作方式就像基于每个项目的`.gitignore`一样。如果你需要忽略某些文件模式(例如，一个外来编辑器的备份文件)，你甚至可以使用一个像`~/.config/git/ignore`这样的每个用户的文件。酷的是这些文件位于 Git 不检查的区域。因此 Git 不会将它们添加到变更集中，所以它不会提交和推送到远程。

**注意:**这另外两个文件使用`.gitignore`格式，所以你也可以在其中使用通配符。

## 列出忽略的文件

如果 Git 没有帮助您确定文件或目录是否被忽略的命令，它就不是 Git。这些命令的第一个是 [`git ls-files`](https://git-scm.com/docs/git-ls-files) :

```
$ git ls-files --others --exclude-from=.gitignore
$ git ls-files --others --exclude-from=.git/info/exclude
```

`--others`参数告诉命令显示不在索引中的文件，而`--exclude-from`是一个过滤器，根据它的参数不显示文件。因此，第一个版本显示了没有在`.gitignore`文件中列出的被忽略的文件。

另一个有用的命令是 [`git check-ignore`](https://git-scm.com/docs/git-check-ignore) ，它需要一个路径参数。如果成功，它将返回文件名和退出代码 0。否则，如果参数不在某个忽略文件中，此命令将退出，代码为 1。

Git 有大量的手册页可以帮助您使用命令和文件。最值得注意的是对于我们的目的来说，你会想把重点放在 [`gitignore(5)`](https://git-scm.com/docs/gitignore) 页面上。

*Last updated: April 7, 2022*