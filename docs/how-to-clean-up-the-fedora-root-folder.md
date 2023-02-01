# 如何清理 Fedora 根文件夹

> 原文：<https://developers.redhat.com/blog/2020/12/10/how-to-clean-up-the-fedora-root-folder>

当升级软件包或 Fedora 发布版本时，我有时会遇到以下错误:

```
Disk Requirements: At least XXX more space needed on the / filesystem.
```

这条消息告诉我磁盘空间不足。在执行升级之前，我需要清理 Fedora 根文件夹空间。

当浏览以了解更多关于这个问题的信息时，我可以很容易地找到许多有同样问题的人，以及许多不同情况下发生的问题。此外，解决这个问题的可能方法通常分布在各种论坛上，这使得选择正确的方法变得很复杂。

在本文中，我收集了一些有趣的资料，描述了解决这个问题的不同方法，即使我的情况不是根本原因。我希望我的经验和对这个问题的理解有助于节省您解决这个问题的时间。

## 搜索最大的文件夹

首先，我执行本地搜索来帮助指导我的下一步行动。一次搜索显示了`"/home"`之外的 30 个更大的文件夹。排除`/home`很重要，因为它通常是挂载在根分区之外的数据分区，如下面的命令所示。如果我还需要排除几个数据分区:

```
sudo du --exclude="/home" -x -h -a / | sort -r -h | head -30
```

(感谢 CabSud 在这个[线程](https://forums.fedora-fr.org/viewtopic.php?id=64241)中提出命令。)

之后，我检查最大的文件夹，以确定它们的大小和用途。然后我就可以安全地删除这些文件夹，或者使用 safe 命令(如果有的话)。查看需要大量空间的常见物品列表，以及用于安全清理它们的命令。

## 常见可清理文件夹

在这些搜索中，我发现了几种解决磁盘空间不足警告的推荐方法。每一个都描述如下。

### 码头工人

Docker 将所有映像和容器存储在根分区中。对于重度用户来说，很多存储的图像很可能不再使用。当我运行这个命令时，它清理了 27GB:

```
docker system prune -a
```

Docker 命令展示了最有效的结果。

### 内核早期版本

默认情况下，会保留三个内核版本。您可以使用以下命令删除一个以释放空间，只保留两个:

```
dnf remove $(dnf repoquery --installonly --latest-limit=-2 -q)
```

不要期望`dnf`命令提供太多的尺寸清理。它不会释放几千兆字节，但它仍然是清理一些空间的一个选项。更多详情，请看[这篇文章](https://www.if-not-true-then-false.com/2012/delete-remove-old-kernels-on-fedora-centos-red-hat-rhel/)。

### Fedora 版本缓存

当升级到 Fedora 的新版本时，会创建一个缓存。理论上，升级后会清理缓存。否则，可以使用以下命令强制清理:

```
dnf system-upgrade clean
```

### 日志日志

日记日志会占用相当大的空间。在我的例子中，运行这个命令清除了 1.5GB:

```
sudo journalctl --vacuum-size=100M
```

然而，我注意到一些人提到它为他们节省了多达 5GB 的空间。更多选项在[这篇博文](https://nts.strzibny.name/cleaning-up-systemd-journal-logs-on-fedora/)中解释。

### Dnf 包缓存

我提到最后一个选项是因为它通常是我所查看的论坛中显示的第一个建议，因此值得注意。在我的例子中，这个命令没有清理任何东西:

```
dnf clean packages
```

在[命令文档中了解更多选项。](https://dnf.readthedocs.io/en/latest/command_ref.html#clean-command)

## 增加根分区的大小

If you still don't have space, you can increase the root partition size. Several people say [this tutorial](https://gist.github.com/181192/cf7eb42a25538ccdb8d0bb7dd57cf236) worked for them. I didn't use it myself because it requires you to unmount the home folder. As a newbie in Fedora usage, I preferred to investigate again to reduce the size before attempting this manipulation, and I finally found success.

希望分享我的经验能帮助您找到解决根分区空间问题的正确方法。

*Last updated: December 20, 2020*