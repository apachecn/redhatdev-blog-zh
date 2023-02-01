# Red Hat Enterprise Linux 8 中的模块化 Perl

> 原文：<https://developers.redhat.com/articles/2022/03/10/modular-perl-red-hat-enterprise-linux-8>

[](/products/rhel)[版本 8](https://www.redhat.com/en/enterprise-linux-8) (RHEL 8)中的红帽企业版 Linux 自带*模块*，这是一个打包概念，允许系统管理员从多个打包版本中选择想要的软件版本。本文向您展示了如何将 Perl 作为一个模块来管理，以及如何在 RHEL 8.0 中管理 Perl 提供的 CPAN 模块。

**注意**:本文中的术语*模块*同时用于 RHEL 模块和 Perl 模块。我将针对 Red Hat Enterprise Linux 类型引用“模块化模块”,针对 Perl 类型引用“CPAN 模块”。

## 从默认流安装 Perl

首先以简单的方式安装 Perl:

```
# yum --allowerasing install perl
Last metadata expiration check: 1:37:36 ago on Tue 07 May 2019 04:18:01 PM CEST.
Dependencies resolved.
==========================================================================================
 Package                       Arch    Version                Repository             Size
==========================================================================================
Installing:
 perl                          x86_64  4:5.26.3-416.el8       rhel-8.0.z-appstream   72 k
Installing dependencies:
[…]
Transaction Summary
==========================================================================================
Install  147 Packages

Total download size: 21 M
Installed size: 59 M
Is this ok [y/N]: y
[…]
  perl-threads-shared-1.58-2.el8.x86_64

Complete!
```

接下来，检查您拥有哪个版本的 Perl:

```
$ perl -V:version
version='5.26.3';
```

输出显示您拥有 Perl 5.26.3。这是未来 10 年支持的默认版本。如果你对此没有意见，那么你就不需要担心模块。但是如果你想尝试不同的版本呢？启用软件的另一个版本有多种原因。最常见的情况是，您可能有一个依赖于模块组合的现有应用程序，其中一些不能与 Perl 的更新版本一起工作。并非所有模块都与其他模块兼容。

在本文的剩余部分，我们将研究如何使用流安装 Perl 模块。

## 探索 RHEL 的溪流 8

使用`yum module list`命令找出可用的 Perl 模块:

```
# yum module list
Last metadata expiration check: 1:45:10 ago on Tue 07 May 2019 04:18:01 PM CEST.
[…]
Name                 Stream           Profiles     Summary
[…]
parfait              0.5              common       Parfait Module
perl                 5.24             common [d],  Practical Extraction and Report Languag
                                      minimal      e
perl                 5.26 [d]         common [d],  Practical Extraction and Report Languag
                                      minimal      e
perl-App-cpanminus   1.7044 [d]       common [d]   Get, unpack, build and install CPAN mod
                                                   ules
perl-DBD-MySQL       4.046 [d]        common [d]   A MySQL interface for Perl
perl-DBD-Pg          3.7 [d]          common [d]   A PostgreSQL interface for Perl
perl-DBD-SQLite      1.58 [d]         common [d]   SQLite DBI driver
perl-DBI             1.641 [d]        common [d]   A database access API for Perl
perl-FCGI            0.78 [d]         common [d]   FastCGI Perl bindings
perl-YAML            1.24 [d]         common [d]   Perl parser for YAML
php                  7.2 [d]          common [d],  PHP scripting language
                                      devel, minim
                                      al
[…] 
```

输出显示一个模块对 Perl 5.24 和 Perl 5.26 都可用。在模块化的世界中，这些被称为*流*，它们表示同一软件栈的独立变体，通常是不同的版本。`[d]`标志标记了一个默认流，如果您没有显式地启用一个不同的流，它就是安装的流。所以前面的输出解释了为什么`yum`安装了 Perl 5.26.3 而不是 5.24 微版本之一。

通常，任何模块都可以有多个流。最多可以默认一个流，启用另一个流。启用的流优先于默认的流。如果没有启用或默认的流，则模块的内容不可用。

现在，让我们做一些假设:

*   您有一个要从 RHEL 7 迁移的旧应用程序。
*   应用程序依赖于`rh-perl524` [红帽软件集合](https://www.redhat.com/en/resources/red-hat-software-collections)环境。
*   你想在 RHEL 8 号上试试吗？

基于这些假设，您将希望在 RHEL 8 上安装 Perl 5.24。

## 启用流

首先，将 Perl 模块切换到 5.24 流:

```
# yum module enable perl:5.24
Last metadata expiration check: 2:03:16 ago on Tue 07 May 2019 04:18:01 PM CEST.
Problems in request:
Modular dependency problems with Defaults:

 Problem 1: conflicting requests
  - module freeradius:3.0:8000020190425181943:75ec4169-0.x86_64 requires module(perl:5.26), but none of the providers can be installed
  - module perl:5.26:820181219174508:9edba152-0.x86_64 conflicts with module(perl:5.24) provided by perl:5.24:820190207164249:ee766497-0.x86_64
  - module perl:5.24:820190207164249:ee766497-0.x86_64 conflicts with module(perl:5.26) provided by perl:5.26:820181219174508:9edba152-0.x86_64
 Problem 2: conflicting requests
  - module freeradius:3.0:820190131191847:fbe42456-0.x86_64 requires module(perl:5.26), but none of the providers can be installed
  - module perl:5.26:820181219174508:9edba152-0.x86_64 conflicts with module(perl:5.24) provided by perl:5.24:820190207164249:ee766497-0.x86_64
  - module perl:5.24:820190207164249:ee766497-0.x86_64 conflicts with module(perl:5.26) provided by perl:5.26:820181219174508:9edba152-0.x86_64
Dependencies resolved.
==========================================================================================
 Package              Arch                Version              Repository            Size
==========================================================================================
Enabling module streams:
 perl                                     5.24

Transaction Summary
==========================================================================================

Is this ok [y/N]: y
Complete!

Switching module streams does not alter installed packages (see 'module enable' in dnf(8)
for details)
```

我们得到一个警告，提示`freeradius:3.0`流与`perl:5.24`不兼容。这是因为 FreeRADIUS 只是为 Perl 5.26 构建的。让我们假设您的应用程序幸运地不需要 FreeRADIUS。

接下来，该命令显示一条确认消息，确认它正在启用 Perl 5.24 流。最后，还有一个关于已安装软件包的警告。最后一个警告意味着系统可能仍然包含来自 Perl 5.26 流的 RPM 包，您需要明确地将它们分类。

如果您碰巧收到以下错误消息，则您已经启用了`perl:5.26`流，并且出于安全原因`yum`不允许您从已经启用的流切换模块:

```
The operation would result in switching of module 'perl' stream '5.26' to stream '5.24'
Error: It is not possible to switch enabled streams of a module unless explicitly enabled via configuration option module_stream_switch.
It is recommended to rather remove all installed content from the module, and reset the module using 'yum module reset <module_name>' command. After you reset the module, you can install the other stream.
```

要从安全块中恢复，请遵循消息中的建议，并使用`yum module reset perl`命令重置`perl`模块。然后您将能够启用`perl:5.24`流。

更改模块和更改包是两个独立的阶段。您可以通过同步分发内容来修复它，如下所示:

```
# yum --allowerasing distrosync
Last metadata expiration check: 0:00:56 ago on Tue 07 May 2019 06:33:36 PM CEST.
Modular dependency problems:

 Problem 1: module freeradius:3.0:8000020190425181943:75ec4169-0.x86_64 requires module(perl:5.26), but none of the providers can be installed
  - module perl:5.26:820181219174508:9edba152-0.x86_64 conflicts with module(perl:5.24) provided by perl:5.24:820190207164249:ee766497-0.x86_64
  - module perl:5.24:820190207164249:ee766497-0.x86_64 conflicts with module(perl:5.26) provided by perl:5.26:820181219174508:9edba152-0.x86_64
  - conflicting requests
 Problem 2: module freeradius:3.0:820190131191847:fbe42456-0.x86_64 requires module(perl:5.26), but none of the providers can be installed
  - module perl:5.26:820181219174508:9edba152-0.x86_64 conflicts with module(perl:5.24) provided by perl:5.24:820190207164249:ee766497-0.x86_64
  - module perl:5.24:820190207164249:ee766497-0.x86_64 conflicts with module(perl:5.26) provided by perl:5.26:820181219174508:9edba152-0.x86_64
  - conflicting requests
Dependencies resolved.
==========================================================================================
 Package           Arch   Version                              Repository            Size
==========================================================================================
[…]
Downgrading:
 perl              x86_64 4:5.24.4-403.module+el8+2770+c759b41a
                                                               rhel-8.0.z-appstream 6.1 M
[…]
Transaction Summary
==========================================================================================
Upgrade    69 Packages
Downgrade  66 Packages

Total download size: 20 M
Is this ok [y/N]: y
[…]
Complete!
```

再次使用`perl`命令检查您的版本:

```
$ perl -V:version
version='5.24.4';
```

太好了！安装非默认模块有效。Perl 的期望版本被安装到一个标准路径(`/usr/bin/perl`)，因此可以用`perl`命令调用。不需要`scl enable`咒语，与旧软件集合相关的需求形成对比。

**注意:**未来的`yum`更新将会清理关于 FreeRADIUS 的不必要的警告。在本文的后面，我将展示一些与任何 Perl 流兼容的 Perl-ish 模块。

## 从属模块

让我们假设前面提到的旧应用程序使用了`DBD::SQLite` Perl CPAN 模块。所以，我们来装吧。Yum 可以在模块化模块中搜索 CPAN 模块，所以可以尝试以下方法:

```
# yum --allowerasing install 'perl(DBD::SQLite)'
[…]
Dependencies resolved.
==========================================================================================
 Package          Arch    Version                             Repository             Size
==========================================================================================
Installing:
 perl-DBD-SQLite  x86_64  1.58-1.module+el8+2519+e351b2a7     rhel-8.0.z-appstream  186 k
Installing dependencies:
 perl-DBI         x86_64  1.641-2.module+el8+2701+78cee6b5    rhel-8.0.z-appstream  739 k
Enabling module streams:
 perl-DBD-SQLite          1.58
 perl-DBI                 1.641

Transaction Summary
==========================================================================================
Install  2 Packages

Total download size: 924 k
Installed size: 2.3 M
Is this ok [y/N]: y
[…]
Installed:
  perl-DBD-SQLite-1.58-1.module+el8+2519+e351b2a7.x86_64
  perl-DBI-1.641-2.module+el8+2701+78cee6b5.x86_64

Complete!
```

在属于`perl-DBD-SQLite:1.58`模块化模块的`perl-DBD-SQLite` RPM 包中发现了`DBD::SQLite` CPAN 模块。显然，它也需要一些对`perl-DBI:1.641`模块化模块的依赖。在请求确认之后，`yum`启用了必要的流并安装了软件包。

在使用 Perl 5.24 下的`DBD::SQLite`之前，看一下模块化模块的清单，并与我们第一次得到的进行比较:

```
# yum module list
[…]
parfait              0.5              common       Parfait Module
perl                 5.24 [e]         common [d],  Practical Extraction and Report Languag
                                      minimal      e
perl                 5.26 [d]         common [d],  Practical Extraction and Report Languag
                                      minimal      e
perl-App-cpanminus   1.7044 [d]       common [d]   Get, unpack, build and install CPAN mod
                                                   ules
perl-DBD-MySQL       4.046 [d]        common [d]   A MySQL interface for Perl
perl-DBD-Pg          3.7 [d]          common [d]   A PostgreSQL interface for Perl
perl-DBD-SQLite      1.58 [d][e]      common [d]   SQLite DBI driver
perl-DBI             1.641 [d][e]     common [d]   A database access API for Perl
perl-FCGI            0.78 [d]         common [d]   FastCGI Perl bindings
perl-YAML            1.24 [d]         common [d]   Perl parser for YAML
php                  7.2 [d]          common [d],  PHP scripting language
                                      devel, minim
                                      al
[…] 
```

请注意，`perl:5.24`已启用(`[e]`)，因此优先于`perl:5.26`，否则它将是默认流(`[d]`)。其他启用的模块化模块有`perl-DBD-SQLite:1.58`和`perl-DBI:1.641`。这些在你安装`DBD::SQLite`的时候就已经启用了。这两个模块没有其他流。

**注意:**如果你出于某种原因需要禁用一个流，甚至是一个默认流，使用`yum module disable <module>:<stream>`命令。

回去做些有成效的工作。你已经准备好测试`DBD::SQLite` CPAN 模块。让我们创建一个`test`数据库，包含一个带有一个名为`bar`的文本列的`foo`表，并在那里存储一个包含字符串`Hello`的行:

```
$ perl -MDBI -e '$dbh=DBI->connect(q{dbi:SQLite:dbname=test});
    $dbh->do(q{CREATE TABLE foo (bar text)});
    $sth=$dbh->prepare(q{INSERT INTO foo(bar) VALUES(?)});
    $sth->execute(q{Hello})'
```

接下来，通过查询数据库来验证`Hello`字符串确实被存储了:

```
$ perl -MDBI -e '$dbh=DBI->connect(q{dbi:SQLite:dbname=test}); print $dbh->selectrow_array(q{SELECT bar FROM foo}), qq{\n}'
Hello
```

输出显示`DBD::SQLite`工作。

## 使用非默认流安装非模块化软件包

到目前为止，我们想要的一切都在工作。但是现在让我们看看如果您试图安装不兼容的 RPM 包会发生什么:

```
# yum --allowerasing install 'perl(LWP)'
[…]
Error:
 Problem: package perl-libwww-perl-6.34-1.el8.noarch requires perl(:MODULE_COMPAT_5.26.2), but none of the providers can be installed
  - cannot install the best candidate for the job
  - package perl-libs-4:5.26.3-416.el8.i686 is excluded
  - package perl-libs-4:5.26.3-416.el8.x86_64 is excluded
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
```

Yum 报告了一个关于`perl-libwww-perl` RPM 包不兼容的错误。被打包成`perl-libwww-perl`的`LWP` CPAN 模块只为 Perl 5.26 构建，所以不能满足 RPM 依赖性。

当`perl:5.24`流被启用时，来自`perl:5.26`流的包被屏蔽，这意味着它们变得不可用。然而，这种屏蔽不适用于非模块化软件包，如`perl-libwww-perl`。许多包还没有模块化。如果您需要它们中的一些可用并与非默认流兼容(即，不仅仅与 Perl 5.26 兼容)，请不要犹豫，联系 [Red Hat 支持团队](https://access.redhat.com/support)提出您的请求。但是，请确保您想要的非默认流[仍然受支持](https://access.redhat.com/support/policy/updates/rhel8-app-streams-life-cycle)。

## 重置模块

假设您想知道您现有的应用程序是否能与新的 Perl 5.26 版本一起工作。为此，您需要切换回`perl:5.26`流。

不幸的是，从启用的流切换回默认流或另一个非默认流并不是一个简单的过程。您需要执行模块重置:

```
# yum module reset perl
[…]
Dependencies resolved.
==========================================================================================
 Package              Arch                Version              Repository            Size
==========================================================================================
Resetting module streams:
 perl                                     5.24

Transaction Summary
==========================================================================================

Is this ok [y/N]: y
Complete!
```

那不太疼。现在，您可以再次同步发行版，将 5.24 RPM 包替换为 5.24 RPM 包:

```
# yum --allowerasing distrosync
[…]
Transaction Summary
==========================================================================================
Upgrade    65 Packages
Downgrade  71 Packages

Total download size: 22 M
Is this ok [y/N]: y
[…]
```

之后，您可以检查 Perl 版本:

```
$ perl -V:version
version='5.26.3';
```

并检查启用的模块:

```
# yum module list
[…]
parfait              0.5              common       Parfait Module
perl                 5.24             common [d],  Practical Extraction and Report Languag
                                      minimal      e
perl                 5.26 [d]         common [d],  Practical Extraction and Report Languag
                                      minimal      e
perl-App-cpanminus   1.7044 [d]       common [d]   Get, unpack, build and install CPAN mod
                                                   ules
perl-DBD-MySQL       4.046 [d]        common [d]   A MySQL interface for Perl
perl-DBD-Pg          3.7 [d]          common [d]   A PostgreSQL interface for Perl
perl-DBD-SQLite      1.58 [d][e]      common [d]   SQLite DBI driver
perl-DBI             1.641 [d][e]     common [d]   A database access API for Perl
perl-FCGI            0.78 [d]         common [d]   FastCGI Perl bindings
perl-YAML            1.24 [d]         common [d]   Perl parser for YAML
php                  7.2 [d]          common [d],  PHP scripting language
                                      devel, minim
                                      al
[…] 
```

你又回到了起点。`perl:5.24`流未启用，`perl:5.26`是默认的，因此是首选的。只有`perl-DBD-SQLite:1.58`和`perl-DBI:1.641`流保持启用状态，这并不重要，因为它们是仅有的流。尽管如此，如果你愿意，你可以使用`yum module reset perl-DBI perl-DBD-SQLite`重置它们。

## 多上下文流

CPAN 模块发生了什么事？它还在那里工作着:

```
$ perl -MDBI -e '$dbh=DBI->connect(q{dbi:SQLite:dbname=test}); print $dbh->selectrow_array(q{SELECT bar FROM foo}), qq{\n}'
Hello
```

这个命令可以工作，因为`perl-DBD-SQLite`模块是为 5.24 和 5.26 Perl 版本构建的。我们称这些模块为*多上下文*。对于`perl-DBD-MySQL`或`perl-DBI`来说也是如此，但是对于 FreeRADIUS 来说就不是这样了，这解释了你之前看到的警告。如果您想查看这些底层细节——比如哪些上下文可用，需要哪些依赖项，或者模块中包含哪些包——您可以使用`yum module info <module>:<stream>`命令。

## 结论

我希望本教程对模块有所启发，模块是 Red Hat Enterprise Linux 8 的一个特性，它允许您在一个 Linux 平台上安装多个版本的软件。如果您需要更多细节，请阅读产品附带的[文档(即用户空间组件管理文档和](/rhel8/) [yum(8)手册页](http://man7.org/linux/man-pages/man8/yum.8.html))或向支持团队寻求帮助。