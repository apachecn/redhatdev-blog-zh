# 使用软件集合，无需考虑替代路径

> 原文：<https://developers.redhat.com/blog/2017/10/18/use-software-collections-without-bothering-alternative-path>

[软件集合](https://access.redhat.com/documentation/en/red-hat-software-collections/3/) (SCL)让您能够在同一系统上构建、安装和使用多个版本的软件，而不会影响系统范围内已安装的软件包。因此，软件集合打包技术被大量用于构建 Red Hat Enterprise Linux 和 CentOS 的堆栈，尤其是动态语言(Python、Ruby、NodeJS)或数据库(PostgreSQL、MariaDB、MongoDB)。

## 避免冲突

SCL 技术基于在三个层面上避免冲突:

*   文件系统(文件放在/opt/rh 下的备用目录中)
*   RPM 包名(所有包名都以集合名为前缀)
*   RPM 元数据(特别提供的应该是唯一的，所以 SCL 包不会影响来自基础系统的包)

为了使用软件集合包，用户需要做一些与普通 rpm 不同的事情。例如，他们需要使用“scl enable”调用，这将改变 PATH 或 LD_LIBRARY_PATH 等环境变量，以便找到备选路径下的二进制文件。用户还需要为 systemd 服务使用不同的名称。或者，一些脚本可能使用二进制文件的完整路径，如/usr/bin/mysql，因此这些脚本可能无法用于软件集合。虽然很多东西都可以在用户没有注意到他们使用 SCL 的情况下工作，但还是会有一些类似的问题。

不幸的是，解决这些问题并不容易，因为我们将失去不影响底层基础系统的能力，这是软件集合技术的主要特征。

## 当冲突不成问题时

但是，在最新的 Red Hat 软件集合 3.0 版中，新的数据库集合(PostgreSQL 9.6、MariaDB 10.2 和 MongoDB 3.4)使用了一个名为“syspaths”的新概念。这个概念允许用户在不影响其他环境的情况下使用不同版本的包，但是也允许他们像使用其他包一样使用 RPM 包。感觉好像根本不会使用软件集合技术，这允许用户选择，他们是喜欢隔离还是使用简单。

通过安装名为*-syspaths 的可选软件包，例如 rh-mariadb102-syspaths，可以简单地启用 syspaths 的概念。这些包首先提供外壳包装器，它被故意安装到一个标准路径中(通常是/usr/bin)。这意味着安装这些软件包显然破坏了将 SCL 与系统其余部分隔离的特性。另一方面，并不是每个人都需要在一个系统上同时安装和运行同一个包的多个版本，对于数据库服务器来说尤其如此。

## 二进制文件和变量文件的包装外壳脚本

那么，安装*-syspaths 包后，我们可以看到哪些变化呢？第一件显而易见的事情是，如上所述，在/usr/bin 中只有很少的二进制文件(可执行的 shell 脚本)。由于这一点，用户可以简单地运行 mysql 或 mysqladmin，而不需要以前使用“scl enable”。诀窍不是任何魔法，在“/usr/bin”中的包装器只是设置正确的环境并执行仍然存在于/opt/rh 中的原始二进制文件。

事实上，我们使用外壳包装器并不允许我们做任何我们可以用原始二进制做的事情。例如，为了使用 gdb 调试二进制文件，我们需要使用/opt/rh 中的完整路径，因为 gdb 不喜欢包装器外壳脚本。

无论如何，二进制文件并不是我们放在/opt、/etc/opt/或/var/opt/目录之外的唯一东西。例如，数据库文件的路径(对于 MariaDB 10.2 SCL，最初在/var/opt/RH/RH-Maria db 102/lib/MySQL 中)现在将更容易发现，因为符号链接位于用户通常会查看的路径中-在/var/lib 下。然而，我们不能简单地创建一个名为/var/lib/mysql 的符号链接，因为该目录可能仍然包含来自系统数据库的数据，我们真的不想处理任何用户的数据。此外，RPM 不喜欢将符号链接和目录混合用于同一路径。

因此，符号链接被称为/var/lib/rh-mariadb102-mysql，同样，对于日志文件位置，符号链接被称为/var/log/rh-mariadb102-mariadb(您可以看到模式`//-`)。尽管如此，名字本身并不重要；如前所述，这里重要的是在查看/var/lib 或/var/log 时可以很容易地找到这些文件。

## 配置文件和服务

您想从 syspaths 特性中获得更多吗？在这里，与变量文件相同的概念也用于配置文件；名为 rh-mariadb102-my.cnf 和 rh-mariadb102-my.cnf.d 的符号链接可以很容易地在/etc 中找到。同样，这应该有助于定位配置文件。

systd 和 SysV init 服务是用户交互的数据库服务的另一个入口点。用户在启动服务时通常不需要使用“scl enable ”,因为服务是在干净的环境中启动的。尽管如此，用户仍需要使用正确的名称，通常以 SCL 名称为前缀(在没有安装 syspaths 包的情况下，如果是 MariaDB 10.2，则为 rh-mariadb102-mariadb)。syspaths 特性允许用户在安装 syspaths 包时使用服务的正常名称，如 mariadb、mongod 或 postgresql。是的，这意味着运行“systemctl start mariadb”就可以了。

因此，所有这些增强不仅应该帮助用户在终端中更容易地使用新的数据库集合，而且任何现有的脚本都不需要再进行更改。只是不要忘记*-syspaths 包显然与基本系统的包冲突，所以系统包不能与 syspaths 包安装在一起。如果这对您来说是个问题，但是您仍然希望使用*-syspaths 包，那么您也可以考虑容器技术(比如 Docker)来避免冲突。在这种情况下，您可以将*-syspaths 包安装到容器中，但是不能有来自基本系统的冲突包。

## 欢迎反馈

还有什么？那么，享受新的*-syspaths 功能，并在这里或在[sclorg@redhat.com](mailto:sclorg@redhat.com)邮件列表上向我们提供您的反馈。

* * *

[**加入红帽开发者计划**](https://developers.redhat.com/?intcmp=70160000000xZNgAAM) **(免费)并获得相关的备忘单、书籍和产品下载。**

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: October 13, 2017*