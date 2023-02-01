# 使用 Red Hat JBoss Developer Studio 避免 Windows rsync 权限问题

> 原文：<https://developers.redhat.com/blog/2018/02/07/windows-rsync-jboss-developer-studio>

JBoss Tools OpenShift 工具使用 rsync 在本地工作站和 OpenShift 集群上运行的 pods 之间同步文件。OpenShift 服务器适配器使用它为开发人员提供热部署和调试功能。如果您在 Windows 上运行 Red Hat JBoss Developer Studio 或 JBoss Tools，那么文件权限会出现一些令人头疼的问题。这些问题是由于底层 Windows 文件系统上使用的文件权限模型不同于 Linux 使用的模型。Linux 和 macOS 用户不会遇到这些问题。本文的目的是解释根本原因以及如何修复它。

## 问题是

OpenShift 工具使用 rsync 将项目的本地更改同步到运行在 OpenShift 集群上的远程 pods。Windows 上的问题是 rsync 非常基于 Linux/UNIX，默认情况下，它试图在两个平台之间映射文件的所有权/权限。由于使用 ACL 的 Windows 文件所有权和权限模型与 Linux 模型完全不同，它会导致在您的文件上设置奇怪的权限。这可能会导致在尝试更新这些文件时出现一些故障。

使用一些 rsync 标志可以防止 rsync 同步文件所有权和权限。不幸的是，rsync 是在 OpenShift oc 客户端的帮助下启动的，该客户端不允许您传入必要的标志。

## 解决方案

在 Windows 上，rsync(和其他 Linux 实用程序)可以从几个来源获得。比较知名的有 [Cygwin](https://www.cygwin.com/) 和 [cwRsync](https://itefix.net/cwrsync) 。cwRsync 基于 Cygwin，但为 Windows 添加了额外的功能。下面的解决方案是给 Cygwin 的。这可能不适用于 rsync for Windows 的其他来源。如果您没有使用 Cygwin 或基于 Cygwin 的东西，请查看您的发行版的路径映射文档。

要在 Windows 上使用基于 Linux 的工具，如 rsync，需要在 Windows 路径(基于 C:\的)和 Linux/UNIX 路径之间进行映射。对于 Cygwin，映射是通过一个名为`/etc/fstab`的文件来控制的。我们在这里提出的解决方案是指示 Cygwin 不要在这个级别映射所有权和权限。请注意，该文件可能不存在，因此您可能需要创建它。

### 如何在您的系统上找到 etc/fstab

fstab 文件通常位于 cygwin/rsync 发行版的`etc`目录中，而 rsync 本身位于`bin`中。Cygwin 的默认安装通常是`C:\cygwin`，所以`rsync.exe`会在`C:\cygwin\bin`而`fstab`会在`C:\cygwin\etc`。找到 Cygwin 或 cwRsync 安装位置的一种快速方法是打开 Windows 命令 shell (cmd ),键入以下命令，查看您的系统在哪里找到 Rsync:

```
where rsync.exe
```

这将给出 rsync.exe 可执行文件的路径..它的一般形式是`<some path>\bin\rsync.exe`。`\bin\rsync`之前的路径是您系统上 cygwin 或 cwRsync 的安装目录。我将把这条路称为`$MY_RSYNC_DIST_DIR`。

fstab 文件位于`$MY_RSYNC_DIST_DIR/etc/fstab`。

### 更新 etc/fstab

如果该文件已经存在，那么您可能会看到类似如下的一行:

```
none /cygdrive cygdrive binary,posix=0,user 0 0
```

因此，只需将`noacl`标志添加到选项列表中。此外，确保该行没有被注释掉。Cygwin 中包含的示例`fstab`有上面的代码行，但是用“#”注释掉了。当你完成后，你应该在`etc/fstab`中看到下面一行:

`none /cygdrive cygdrive binary,posix=0,user,noacl 0 0`

如果您的发行版没有 fstab 文件，这对于 cwRsync 来说是很典型的，那么这个问题很容易解决。在`$MY_RSYNC_DIST_DIR/etc/fstab`处创建文件，并粘贴以下内容:

```
# /etc/fstab
#
#    This file is read once by the first process in a Cygwin process tree.
#    To pick up changes, restart all Cygwin processes.  For a description
#    see https://cygwin.com/cygwin-ug-net/using.html#mount-table
#
none /cygdrive cygdrive binary,posix=0,user,noacl 0 0 
```

保存文件，你就完成了。现在你应该不会在 rsync 操作中遇到权限错误了！！！。

*Last updated: March 19, 2020*