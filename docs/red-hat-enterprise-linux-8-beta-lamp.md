# 如何在红帽企业版 Linux 8 Beta 上快速设置灯栈

> 原文：<https://developers.redhat.com/blog/2019/03/14/red-hat-enterprise-linux-8-beta-lamp>

你试过[红帽企业版 Linux 8](https://developers.redhat.com/rhel8/) (RHEL8)测试版吗？继续阅读，了解如何在 RHEL8 Beta 上快速搭建[灯栈](https://developers.redhat.com/blog/2017/03/07/how-to-set-up-a-lamp-stack-on-red-hat-enterprise-linux-7/)，并尝试操作系统内置的新功能。

灯堆由四个主要部件和一些胶水组成。LAMP 堆栈中的第一个主要组件是 Linux。在我的例子中，我使用的是 Red Hat Enterprise Linux 8 Beta，它为我提供了一个安全的操作系统、一个现代化的编程环境和一套用户友好的工具来控制它。

至于 web 服务器，传统上 LAMP 中的“A”代表 *Apache* ，但是在 RHEL8 Beta 中我们实际上在这里有选项。我们提供 Apache httpd 和 RHEL8 Beta，但我们也提供 NGINX。因为我在这里有点传统，所以我会选择 Apache。

在 RHEL8 测试版中，Apache 作为一个 [AppStream](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/) 发布，它允许我们提供具有不同生命周期的内容。例如，使用 AppStreams，我们可以发布多个版本的 Python，并在正常的 RHEL 发布节奏之外添加新版本的编程环境。

在 RHEL8 上安装 Apache 就像在早期版本的 RHEL 上一样简单。运行:

`$ sudo yum -y install httpd`

(你用的是 sudo 吧？如果您在安装时没有将您的用户 ID 设置为管理员，请参见 *[如何在 Red Hat Enterprise Linux](https://developers.redhat.com/blog/2018/08/15/how-to-enable-sudo-on-rhel/) 上启用 sudo。*)

这个命令将启用 Apache 2.4 AppStream 并安装 httpd 包，包括它的默认依赖项。

要启动这个新安装的 web 服务器并确保它在重启后自动启动，我需要运行:

`$ sudo systemctl enable --now httpd`

而且，因为我希望我的服务器可以通过网络访问，所以我需要在我的系统上打开端口 80 和 443。我们可以从 RHEL8 中的 Web 控制台完成这项工作(请参见本文末尾的 DevNation 视频以获得演示)，但是现在，让我们只使用 RHEL8 Beta 提供的命令行工具。它们非常简单:

`$ sudo firewall-cmd --add-service=http --add-service=https`


就是这样。第一个命令立即打开端口 80 和 443，第二个命令确保在重新启动或防火墙重启后，端口仍然打开。

现在是数据库部分。同样，传统上，LAMP 中的“M”代表*MySQL*；然而，如今，它也可以指 MariaDB、MongoDB，甚至 PostgreSQL。您可以通过运行以下命令来查看 RHEL8 Beta 附带了哪些数据库:

`sudo yum module list`

![](img/bf7f724e95af26f4cb8390240e4f57f2.png)

(为了简洁起见，我从输出中去掉了非数据库应用程序流。)

如您所见，MongoDB 不是 RHEL8 测试版的选项。您可以阅读 RHEL8 Beta 发行说明，了解这方面的背景知识。不过，我们确实有 MySQL 8、MariaDB 10.3、PostgreSQL 9.6 和 10 以及 Redis 4 和 5。可供选择的太多了！(但是，不要假设最终产品会附带所有这些版本。毕竟，我们看到的是测试版软件。)

我想在这里构建一个相当传统的 LAMP 堆栈，所以我选择 MariaDB，它是 MySQL 的替代产品。我想安装一个数据库服务器，所以默认的概要文件(*服务器*，由上面输出中的*【d】*表示)将为我工作。如果我只需要客户机位，我可以安装客户机概要文件，为我节省一点磁盘空间。

不过，现在我要跑了:

`$ sudo yum -y module install mariadb`

顺便说一句，标准的`yum -y install mariadb-server`也可以。

没有运行的数据库用处不大，所以让我们从下面开始:

`$ sudo systemctl enable --now mariadb`

我不需要打开防火墙端口，因为我的 web 服务器和数据库服务器运行在同一台机器上。但是，如果您有单独的机器用于 Apache 和 MariaDB，您将需要使用我上面显示的 *firewall-cmd* 命令将 MySQL 服务添加到防火墙。您还需要调整 SELinux 策略，以允许 Apache 通过网络连接到数据库(安全第一！)，通过运行:

`$ sudo setsebool -P httpd_can_network_connect_db on`

最后，因为我已经牢记了关于安全性的课程，所以我将运行*MySQL _ secure _ installation*脚本:

`$ sudo mysql_secure_installation`

我们快到了。我有一台合适的 Linux 机器，我有我的 web 服务器，我有我的数据库服务器。仍然缺少的是一个编程环境，和一些胶水。让我们看看，我有哪些可用的编程环境？

`$ sudo yum module list`

我不会在这里再次展示整个输出，但是我们有 PHP，我们有两个主要版本的 Python，我们有 Ruby，还有许多其他选项。传统的 LAMP 对我来说意味着 PHP，所以这就是我今天要安装的。一个简单命令就能解决这个问题:

`$ sudo yum -y module install php`

还剩下最后两个步骤。首先，一些胶水。为了能够从我的 PHP 页面连接到 MariaDB 数据库，我需要安装一个微型库:

`$ sudo yum -y install php-mysqlnd`

现在，作为最后一步，我将重启 Apache 来获取新安装的 PHP 和 PHP MySQL 库:

`$ sudo systemctl restart httpd`

就这样，我们结束了。我们可以进入`/var/www/html`并在里面放一个 PHP 应用程序，一切都应该正常工作。

几周前，Burr Sutter 在 DevNation Live 上接待了我，我们从开发人员的角度记录了 RHEL8 Beta 的概况。我们讨论了安装和使用编程环境、管理您的开发系统等等。感兴趣吗？观看视频:

https://www.youtube.com/watch?v=4DiLdgtcavo

希望这有帮助。让我知道你在评论或推特上的想法: [@MaximBurgerhout](https://twitter.com/MaximBurgerhout)

*Last updated: May 30, 2022*