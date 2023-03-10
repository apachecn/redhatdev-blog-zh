# RPM 打包:创建第一个 RPM 的简化指南

> 原文：<https://developers.redhat.com/blog/2019/03/18/rpm-packaging-guide-creating-rpm>

RPM 打包的概念对于第一次接触的人来说可能是势不可挡的，因为它给人的印象是一个陡峭的学习曲线。在本文中，我将展示用最少的知识和经验构建一个 RPM 是可能的。请注意，本文只是一个起点，而不是 RPM 打包的完整指南。

## 基于 ASCII 的俄罗斯方块游戏

在我的演示中，我选择了一个简单的基于 ASCII 的俄罗斯方块游戏，用 C 语言编写，我做了一些小小的调整以确保相对简单的 RPM 构建。我将用一个叫 Vitetris 的游戏作为我的例子，你可以[下载](https://www.victornils.net/tetris/vitetris-0.57.tar.gz)。

为了确保在创建 RPM 包时没有错误，我删除了 Makefile 中将文件权限更改为 root 的引用，以便允许非 root 用户构建 RPM。这一修改如下:

```
$ cat Makefile |grep 'INSTALL '
INSTALL = install
#INSTALL = install -oroot -groot # non-root users building the rpm won't be able to set this and the RPM build will fail.
```

完成这个更改后，创建一个新的同名 gzipped tarball:`vitetris-0.57.tar.gz`。

## 准备环境

要在订阅的[Red Hat Enterprise Linux](https://developers.redhat.com/topics/linux/)(RHEL)7 系统上安装您的开发环境，您需要以下存储库来构建您的 rpm:`rhel-7-server-rpms`、`rhel-7-server-extras-rpms`和`rhel-7-server-optional-rpms`。

您必须安装以下软件包:

```
# yum install -y rpm* gcc gpg* rng-tools
```

我使用`rpm*`和`gpg*`,因为这使得要记住安装的东西的数量更易于管理。

## 手动编译软件

RPM 打包成功的很大一部分是理解你正在使用的软件。首先，手动编译软件最好作为非根用户来完成。在本例中，我使用的是`rpmbuilder`用户。

首先解压缩 gzipped tarball ( `vitetris-0.57.tar.gz`)，然后检查 README 文件。在这种情况下，自述文件不包含从源代码编译游戏的信息。一般来说，用 C 写的开源软件编译有三个步骤:`configure`、`make`和`make install`。然而，单独运行`make`来看看这个软件是否可以构建是值得的。

作为非 root 用户，测试运行`make`是否足够:

```
[rpmbuilder@rpm vitetris-0.57]$ make
generating src/src-conf.mk
./src-conf.sh 'cc' '' ''
...
Done.
Now run ./tetris (or make install)

```

要测试游戏，只需运行`./tetris`来查看游戏是否加载并可以玩:

[![Running Tetris after building from source code](img/bf19da7f900ea6da69b3af6c7b1be502.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/03/tetris2.png)

## 创建 GPG 键

对 RPM 包进行签名为您的包增加了一层额外的信任。因此，要以`rpmbuilder`用户的身份创建 GPG 密钥，您需要一个具有 root 访问权限的会话来运行`rngd`(以加速生成过程)，以及一个以`rpmbuilder`用户的身份使用 X11 转发的会话。

以 root 用户身份运行:

```
# rngd -r /dev/urandom
```

如果出现以下错误:

```
Failed to init entropy source 2: Intel RDRAND Instruction RNG
```

尝试:

```
# rngd -r /dev/urandom -o /dev/random -f
```

您必须使用 X11 转发以`rpmbuilder`用户身份登录主机(否则密钥生成将失败):

```
$ gpg --gen-key
...
Please select what kind of key you want:
(1) RSA and RSA (default)
(2) DSA and Elgamal
(3) DSA (sign only)
(4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 
Requested keysize is 2048 bits
Please specify how long the key should be valid.
0 = key does not expire
<n> = key expires in n days
<n>w = key expires in n weeks
<n>m = key expires in n months
<n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: RPM Build User
Email address: rpmbuilder@rpm
Comment: RPM Builder GPG Signing Key
You selected this USER-ID:
"RPM Build User (RPM Builder GPG Signing Key) <rpmbuilder@rpm>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.
```

[![Prompt that is displayed if you have X11 forwarding turned on](img/4c4efd2910de5d8c4bacbe32a15be4b6.png "gpg-password")](/sites/default/files/blog/2019/03/gpg-password.png)

This prompt will only appear if you have X11 forwarding turned on.

```
...
gpg: depth: 0 valid: 1 signed: 0 trust: 0-, 0q, 0n, 0m, 0f, 1u
pub 2048R/EEF6D9AD 2019-03-02
Key fingerprint = 6ED1 2456 B7ED EEC6 D0DF B870 444A 40A7 EEF6 D9AD
uid RPM Build User (RPM Builder GPG Signing Key) <rpmbuilder@rpm>
sub 2048R/D498F883 2019-03-02
```

如果您需要导出要在 satellite for custom 软件或 yum repo 配置中使用的密钥，请使用以下命令:

```
$ gpg --armor --export
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v2.0.22 (GNU/Linux)
```

...

## 为 Vitetris 创建 RPM 包

要构建 RPM，首先创建构建树和规范文件，并将源代码放在正确的位置:

```
$ rpmdev-setuptree
$ rpmdev-newspec rpmbuild/SPECS/vitetris.spec
rpmbuild/SPECS/vitetris.spec created; type minimal, rpm version >= 4.11.
$ cp vitetris-0.57.tar.gz rpmbuild/SOURCES/

```

### 将初始细节添加到等级库文件中

提供初始细节:

```
Name:           vitetris
Version:        0.57
Release:        1%{?dist}
Summary:        ASCII based tetris game

License:        BSD
URL:            https://www.victornils.net/tetris/
Source0:        vitetris-0.57.tar.gz

BuildRequires:  gcc
```

### 删除不必要的部分

*   删除`Requires`部分，因为该软件没有任何依赖关系。
*   删除`%configure`部分，因为该软件可以在没有`configure`的情况下构建(如果在运行`make`之前需要`configure`，请保留该部分)。

### 添加描述

你可以添加任何你喜欢的东西，但这里有一个建议:

```
%description
vitetris is a multiplayer ASCII-based Tetris game
```

### 测试初始版本

使用刚才提供的信息测试构建，看看出现了什么错误:

```
$ rpmbuild -ba rpmbuild/SPECS/vitetris.spec
...
RPM build errors:
Installed (but unpackaged) file(s) found:
/usr/local/bin/tetris
/usr/local/share/applications/vitetris.desktop
/usr/local/share/doc/vitetris/README
/usr/local/share/doc/vitetris/licence.txt
/usr/local/share/pixmaps/vitetris.xpm
```

### 将上一步中的文件列表添加到等级库文件中

您必须将上一步中的文件列表添加到`%files`部分，如下所示:

```
%files
/usr/local/bin/tetris
/usr/local/share/applications/vitetris.desktop
%doc /usr/local/share/doc/vitetris/README
/usr/local/share/doc/vitetris/licence.txt
/usr/local/share/pixmaps/vitetris.xpm
```

请注意，`%doc`被放在自述文件的前面，以表明它是一个信息文档。没有它，RPM 仍然会建立。

### 重新运行生成

```
$ rpmbuild -ba rpmbuild/SPECS/vitetris.spec
Wrote: /home/rpmbuilder/rpmbuild/SRPMS/vitetris-0.57-1.el7.src.rpm
Wrote: /home/rpmbuilder/rpmbuild/RPMS/x86_64/vitetris-0.57-1.el7.x86_64.rpm
Wrote: /home/rpmbuilder/rpmbuild/RPMS/x86_64/vitetris-debuginfo-0.57-1.el7.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.AqR4Aq
+ umask 022
+ cd /home/rpmbuilder/rpmbuild/BUILD
+ cd vitetris-0.57
+ /usr/bin/rm -rf /home/rpmbuilder/rpmbuild/BUILDROOT/vitetris-0.57-1.el7.x86_64
+ exit 0

```

### 签署 RPM 包

为您的 RPM 签名就像运行:

```
$ rpmsign --addsign /home/rpmbuilder/rpmbuild/RPMS/x86_64/vitetris-0.57-1.el7.x86_64.rpm 
Enter pass phrase: Pass phrase is good. 
/home/rpmbuilder/rpmbuild/RPMS/x86_64/vitetris-0.57-1.el7.x86_64.rpm:

```

如果出现此错误:

```
You must set "%_gpg_name" in your macro file
```

用公钥值设置`%_gpg_name`:

```
$ gpg --list-keys 
/home/rpmbuilder/.gnupg/pubring.gpg
-----------------------------------
pub 2048R/EEF6D9AD 2019-03-02
uid RPM Build User (RPM Builder GPG Signing Key) <rpmbuilder@rpm>
sub 2048R/D498F883 2019-03-02

$ echo "%_gpg_name EEF6D9AD" >> .rpmmacros
```

再试一次:

```
$ rpmsign --addsign /home/rpmbuilder/rpmbuild/RPMS/x86_64/vitetris-0.57-1.el7.x86_64.rpm
Enter pass phrase: 
Pass phrase is good.
/home/rpmbuilder/rpmbuild/RPMS/x86_64/vitetris-0.57-1.el7.x86_64.rpm:
```

### 以 root 用户身份测试 RPM 的安装/卸载

```
# rpm -i /home/rpmbuilder/rpmbuild/RPMS/x86_64/vitetris-0.57-1.el7.x86_64.rpm
warning: /home/rpmbuilder/rpmbuild/RPMS/x86_64/vitetris-0.57-1.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID eef6d9ad: NOKEY

# /usr/local/bin/tetris
```

[![Installing the RPM as root](img/e66c2e546ee6bd13f70b1aed966d6f4b.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/03/rpm_tetris.png)

卸载 RPM:

```
# rpm -qa |grep tetris
vitetris-0.57-1.el7.x86_64
=# rpm -e vitetris-0.57-1.el7.x86_64
# /usr/local/bin/tetris
-bash: /usr/local/bin/tetris: No such file or directory
```

## 结论

根据您打算打包的软件，定制 RPM 打包可能会很有挑战性。在本文中，我旨在尽可能少地展示默认设置通常足以构建一个 RPM。

要了解更多信息，请参见 [Red Hat 的 RPM 打包指南](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/rpm_packaging_guide/index)。

*Last updated: March 15, 2019*