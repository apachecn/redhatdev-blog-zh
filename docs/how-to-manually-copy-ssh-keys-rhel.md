# 如何手动将 SSH 公共密钥复制到 Red Hat Enterprise Linux 上的服务器

> 原文：<https://developers.redhat.com/blog/2018/11/02/how-to-manually-copy-ssh-keys-rhel>

我们经常使用`ssh-copy-id`将 ssh 密钥从我们本地的 [Linux](https://developers.redhat.com/topics/linux/) 计算机复制到 RHEL 的服务器上，以便在不输入密码的情况下进行连接。这不仅是为了方便；它使您能够编写脚本并自动化涉及远程机器的任务。此外，正确使用 ssh 密钥被认为是最佳实践。如果您习惯于在每次提示时都用密码来响应，您可能不会注意到不合法的提示(例如，欺骗)。

当您不能使用`ssh-copy-id`或者目标用户 ID 没有密码(例如，Ansible 服务用户)时怎么办？本文解释了如何手动设置，并避免忘记设置适当的权限这一常见缺陷。

通常，您会这样做:

```
ssh-keygen ... ssh-copy-id USER@IP

```

但是，当`ssh-copy-id`不可用时，您可以执行以下操作。这包括设置适当权限的步骤。如果`.ssh`目录和文件的权限和/或所有权不正确，它仍然会要求您输入密码。如果您没有检查日志的 root 访问权限，这可能很难诊断。

在本地计算机上，执行以下操作:

```
$ ssh-keygen
$ cat ~/.ssh/id_rsa.pub
  ssh-rsa ... stuff ... user@domain

```

现在把这一行从`ssh-rsa`复制到你的*用户@域名*，这样它就在剪贴板上，或者放在 u 盘上，或者写在纸上，通过信鸽发送。这是您的公钥，需要添加到远程服务器上的`~/.ssh/authorized_keys`。

在远程服务器上，执行以下操作:

```
$ mkdir ~/.ssh/
$ chmod 700 ~/.ssh  # this is important.
$ touch ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/authorized_keys  #this is important.

```

在远程系统上，编辑`~/.ssh/authorized_keys`并添加上面`cat`命令的输出。

现在，您应该能够从您的计算机连接到远程系统。不需要重启。

有关更多详细信息，请参见 Red Hat 客户门户网站上的[如何在 Red Hat Enterprise Linux 中设置 SSH 无密码登录](https://access.redhat.com/solutions/9194)。请记住，当您[加入红帽开发者计划时，](https://developers.redhat.com/articles/red-hat-developer-program-benefits/)一个免费的开发者订阅会自动添加到您的帐户中。使用您的 Red Hat ID，您可以访问 access.redhat.com[上的文章和知识库。开发者订阅期为一年。但是，要想续订，您只需再次登录](https://access.redhat.com/)[developers.redhat.com](https://developers.redhat.com/login)即可。

与此相关的是，如果你需要帮助来设置`sudo`，这样你就不必输入 root 密码，参见[如何在 Red Hat Enterprise Linux](https://developers.redhat.com/blog/2018/08/15/how-to-enable-sudo-on-rhel/) 上启用 sudo。

快乐嘘。

*Last updated: November 1, 2018*