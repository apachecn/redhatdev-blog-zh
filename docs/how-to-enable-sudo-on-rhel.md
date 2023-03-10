# 如何在 Red Hat Enterprise Linux 上启用 sudo

> 原文：<https://developers.redhat.com/blog/2018/08/15/how-to-enable-sudo-on-rhel>

你可能已经看过使用`sudo`作为根用户运行管理命令的教程。然而，当你尝试它时，你被告知你的用户 ID“不在 sudoers 文件中，这个事件将被报告。”对于开发人员来说，`sudo`对于在构建脚本中运行需要 root 访问权限的步骤非常有用。

本文涵盖:

*   如何在[红帽企业版 Linux](https://developers.redhat.com/products/rhel/overview/) (RHEL)和 CentOS 上配置`sudo`访问权限，这样你就不需要使用`su`并一直输入 root 密码
*   将`sudo`配置为不询问您的密码
*   系统安装时如何启用`sudo`
*   为什么`sudo`对某些用户来说似乎是现成的，而对其他用户却不是

## TL；博士:基础`sudo`

要在 RHEL 上为您的用户 ID 启用`sudo`，请将您的用户 ID 添加到`wheel`组:

1.  通过运行`su`成为 root
2.  运行`usermod -aG wheel *your_user_id*`
3.  注销并再次登录

现在，您可以使用普通用户 ID 登录，使用`sudo`。当您运行`sudo`命令时，会要求您输入您的用户 ID 的密码*。在接下来的五分钟内，`sudo`会记住你已经通过了身份验证，因此不会再次询问你的密码。*

这是因为 RHEL 上的默认`/etc/sudoers`文件包含以下行:

```
%wheel  ALL=(ALL)  ALL
```

该行允许组`wheel`中的所有用户使用`sudo`运行任何命令，但是用户将被要求用他们的密码证明他们的身份。注意:该行前面没有注释符号(`#`)。

注销并再次登录后，您可以通过运行`id`命令来验证您是否在组`wheel`中:

```
$ id
uid=1000(rct) gid=10(wheel) groups=10(wheel),1000(rct)

```

## 使用无密码的`sudo`

您还可以配置`sudo`不要求输入密码来验证您的身份。对于许多情况(比如真实的服务器)，这被认为是太大的安全风险。然而，对于在笔记本电脑上运行 RHEL 虚拟机的开发人员来说，这是一件合理的事情，因为对他们笔记本电脑的访问可能已经受到密码保护。

为了进行设置，显示了两种不同的方法。您可以编辑`/etc/sudoers`或者在`/etc/sudoers.d/`中创建一个新文件。前者更简单，但后者更容易编写脚本和自动化。

### 编辑`/etc/sudoers`

作为 root 用户，运行`visudo`来编辑`/etc/sudoers`并进行以下更改。使用`visudo`的好处是它会验证对文件的更改。

默认的`/etc/sudoers`文件包含两行用于组`wheel`；`NOPASSWD:`行被注释掉了。取消注释该行，并注释掉不带`NOPASSWD`的`wheel`行。完成后，它应该是这样的:

```
## Allows people in group wheel to run all commands
# %wheel ALL=(ALL) ALL

## Same thing without a password
%wheel ALL=(ALL) NOPASSWD: ALL
```

### 替代方法:在`/etc/sudoers.d`中创建一个新文件

您可以在`/etc/sudoers.d`中创建文件，这些文件将成为`sudo`配置的一部分。这种方法更容易编写脚本和自动化。此外，由于这不涉及更改组，您不必注销并再次登录。把*你的 id* 改成你的用户 id。

```
$ su -
# echo -e “*your_id*\tALL=(ALL)\tNOPASSWD: ALL" > /etc/sudoers.d/020_sudo_for_me

# cat /etc/suders.d/020_my_sudo
*your_id* ALL=(ALL) NOPASSWD: ALL

```

## 系统安装期间启用`sudo`

在 RHEL 系统安装过程中，您可以为您在安装过程中创建的用户启用`sudo`。在您输入用户 ID 和密码的*用户创建*屏幕上，有一个经常被忽略(和误解)的*使这个用户成为管理员*选项。如果选择*使该用户成为管理员*框，该用户将在安装过程中成为`wheel`组的一部分。

我不得不承认，我忽略了这个选项，不明白它的作用，直到我在 *Fedora 杂志* 上偶然发现[这篇文章。虽然这篇文章是关于 Fedora 的，但是这个功能对于 RHEL 来说基本上是一样的，因为 Fedora 是上游社区项目，是 RHEL 的基础。](https://fedoramagazine.org/howto-use-sudo/)

对我来说，这最终澄清了为什么有些 RHEL 用户可以开箱即用，而其他人却不行。这在 RHEL 安装指南中没有很好地解释。

[![RHEL 7 Install Create User](img/d0cf7c9cf7cc262fb8cef2b439a4df2e.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/08/rhel_7_createuser.png)

## 了解更多信息

*   参见 *Red Hat Enterprise Linux 7 系统管理员指南*中的[获取特权](http://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/chap-Gaining_Privileges.html)章节。
*   *参见“* [如何允许普通用户使用 sudo](https://access.redhat.com/solutions/1527) *作为根用户运行命令。”*本文位于 Red Hat 客户门户网站上。加入 Red Hat 开发人员计划，获得一个 Red Hat ID，这样您就可以在 Red Hat 客户门户网站上查看知识库文章。
*   参见 *Fedora 杂志*中的“[配置您的 Fedora 系统以使用 sudo](https://fedoramagazine.org/howto-use-sudo/) ”一文。

*Last updated: October 18, 2018*