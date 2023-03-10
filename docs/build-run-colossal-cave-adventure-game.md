# 巨大的洞穴探险:从游戏的黎明开始构建和运行 40 年前的代码

> 原文：<https://developers.redhat.com/articles/build-run-colossal-cave-adventure-game>

[https://www.youtube.com/embed/UFu1WGJP6rQ?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/UFu1WGJP6rQ?autoplay=0&start=0&rel=0)

```
You are standing at the end of a road before a small brick building. 
Around you is a forest. A small stream flows out of the building 
and down a gully.
```

听起来熟悉吗？如果你是一个老派的游戏玩家，你可能已经见过这些单词几百次了。它们是第一款大型电脑游戏巨大洞穴探险[的起点。它是由威利·克罗泽和唐·伍兹在 1977 年创建的，并在 GUI 出现之前、互联网出现之前的年代迅速成为一种现象。帮助游戏从一个数据中心传播到另一个数据中心的是源代码可以免费获得。](https://en.wikipedia.org/wiki/Colossal_Cave_Adventure)

原始作者的代码的最后一个版本是在 1995 年写的，如果你和我们一样，你听到这个故事并想知道代码现在在哪里。在作者的鼓励和允许下，开源先锋 Eric S. Raymond 已经将代码移植到了他的 GitLab 帐户中。在本文中，我们将向您展示如何构建和运行这些代码。

不过，在我们开始之前，有一个更棒的东西的简短宣传片:这篇文章以及相关的视频和 Docker 图像的灵感来自命令行英雄播客，[第二季，第一集](https://www.redhat.com/en/command-line-heroes/season-2/press-start)。

[![](img/091a95b593870153a676829a77784cf7.png)](https://www.redhat.com/en/command-line-heroes/season-2/press-start)

播客的这一集涵盖了巨大的洞穴探险以及更多关于游戏历史的信息。看看这个节目，可以在你最喜欢的播客上找到。(非常感谢 Saron 和命令行英雄帮告诉我们这个故事，激起我们对代码的兴趣。)

让我们开始吧。

## 克隆回购协议并构建代码

代码住在[https://git lab。com//ESR//开启 - 冒险](https://gitlab.com/esr/open-adventure) :

[![](img/c8ee3794dc9fa0cdffba018bd2ffa657.png)](https://gitlab.com/esr/open-adventure)

当然，从这里开始，您需要克隆回购:

```
git clone https://gitlab.com/esr/open-adventure.git
```

一旦你克隆了回购，切换到该目录并运行`make`...这几乎肯定会失败，因为有几个库可能没有安装在您的机器上。

```
doug@dtidwell-mac:~/Developer/open-adventure $ make
./make_dungeon.py
Traceback (most recent call last):
  File "./make_dungeon.py", line 13, in <module>
    import sys, yaml
ImportError: No module named yaml
make: *** [dungeon.h] Error 1

```

关键的库是 Python YAML 库和`libedit`。更复杂的是，在 Linux、Mac 和 Windows 上安装这些库的方式是不同的。

### Linux 操作系统

我们将首先处理 Linux。使用您的发行版的安装管理器来安装软件包`python-yaml`:

```
[doug@rhel-7 open-adventure]$ sudo yum -y install python-yaml
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
Resolving Dependencies
--> Running transaction check
---> Package PyYAML.x86_64 0:3.10-11.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================================================
 Package                 Arch                    Version                      Repository                           Size
========================================================================================================================
Installing:
 PyYAML                  x86_64                  3.10-11.el7                  rhel-7-server-rpms                  153 k

Transaction Summary
========================================================================================================================
Install  1 Package

Total download size: 153 k
Installed size: 630 k
Downloading packages:
PyYAML-3.10-11.el7.x86_64.rpm                                                                    | 153 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : PyYAML-3.10-11.el7.x86_64                                                                            1/1 
  Verifying  : PyYAML-3.10-11.el7.x86_64                                                                            1/1 

Installed:
  PyYAML.x86_64 0:3.10-11.el7                                                                                           

Complete!

```

接下来，安装软件包`libedit-devel:`

```
[doug@rhel-7 open-adventure]$ sudo yum -y install libedit-devel
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
Resolving Dependencies
--> Running transaction check
---> Package libedit-devel.x86_64 0:3.0-12.20121213cvs.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================================================
 Package                 Arch             Version                           Repository                             Size
========================================================================================================================
Installing:
 libedit-devel           x86_64           3.0-12.20121213cvs.el7            rhel-7-server-optional-rpms            33 k

Transaction Summary
========================================================================================================================
Install  1 Package

Total download size: 33 k
Installed size: 52 k
Downloading packages:
libedit-devel-3.0-12.20121213cvs.el7.x86_64.rpm                                                  |  33 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libedit-devel-3.0-12.20121213cvs.el7.x86_64                                                          1/1 
  Verifying  : libedit-devel-3.0-12.20121213cvs.el7.x86_64                                                          1/1 

Installed:
  libedit-devel.x86_64 0:3.0-12.20121213cvs.el7                                                                         

```

有了这些，您应该能够键入`make`并构建代码:

```
[doug@rhel-7 open-adventure]$ make
./make_dungeon.py
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline    -c main.c
main.c: In function ‘do_command’:
main.c:1152:9: warning: implicit declaration of function ‘strncasecmp’ [-Wimplicit-function-declaration]
         if (strncasecmp(command.word[0].raw, "west", sizeof("west")) == 0) {
         ^
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline    -c init.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline    -c actions.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline    -c score.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline    -c misc.c
misc.c: In function ‘get_motion_vocab_id’:
misc.c:362:13: warning: implicit declaration of function ‘strncasecmp’ [-Wimplicit-function-declaration]
             if (strncasecmp(word, motions[i].words.strs[j], TOKLEN) == 0 && (strlen(word) > 1 ||
             ^
misc.c: In function ‘get_vocab_metadata’:
misc.c:457:5: warning: implicit declaration of function ‘strcasecmp’ [-Wimplicit-function-declaration]
     if (strcasecmp(word->raw, game.zzword) == 0) {
     ^
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline    -c saveresume.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all  -c dungeon.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all  -o advent main.o init.o actions.o score.o misc.o saveresume.o dungeon.o  -ledit -ltinfo  

```

您会得到一些警告，但您可能会忽略它们。(哎，代码编译了吧？)您现在应该有一个名为`advent`的可执行文件。

### 苹果个人计算机

在 Mac 上，情况要复杂一些。对于 Python YAML 库，只需键入`sudo -H pip install PyYAML`:

```
doug@dtidwell-mac:~/Developer/open-adventure $ sudo -H pip install PyYAML
Collecting PyYAML
  Using cached https://files.pythonhosted.org/packages/9e/a3/1d13970c3f36777c583f136c136f804d70f500168edc1edea6daa7200769/PyYAML-3.13.tar.gz
Installing collected packages: PyYAML
  Running setup.py install for PyYAML ... done
Successfully installed PyYAML-3.13

```

对于`libedit`，如果你使用 [MacPorts](https://www.macports.org/) ，`sudo port install libedit`就可以了。忽略出现的任何警告错误消息:

```
doug@dtidwell-mac:~/Developer/open-adventure $ sudo port install libedit
Warning: port definitions are more than two weeks old, consider updating them by running 'port selfupdate'.
Warning: xcodebuild exists but failed to execute
Warning: Xcode does not appear to be installed; most ports will likely fail to build.
--->  Computing dependencies for libedit
--->  Cleaning libedit
--->  Scanning binaries for linking errors
--->  No broken files found.
--->  No broken ports found.

```

如果你使用[自制软件](https://brew.sh/)，输入`brew install libedit`:

```
doug@dtidwell-mac:~/Developer/CLH/S2E1 $ brew install libedit
Updating Homebrew...
Ignoring path homebrew-cask/
To restore the stashed changes to /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask run:
  'cd /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask && git stash pop'
==> Auto-updated Homebrew!
Updated 2 taps (homebrew/core, homebrew/cask).
==> New Formulae
skopeo                                   ucloud
==> Updated Formulae
ghostscript ✔       double-conversion   libphonenumber      pdfsandwich
kubernetes-cli ✔    eslint              libspectre          phpunit
annie               folly               lnav                pqiv
apache-drill        fonttools           logentries          ripgrep
at-spi2-atk         grunt-cli           mdcat               terragrunt
atk                 imagemagick         media-info          webpack
atomicparsley       imagemagick@6       mps-youtube         xmount
bwfmetaedit         kustomize           msgpack             youtube-dl
cmdshelf            libimobiledevice    nano
==> Deleted Formulae
llvm@3.7

==> Downloading https://homebrew.bintray.com/bottles/libedit-20180525-3.1.high_s
Already downloaded: /Users/doug/Library/Caches/Homebrew/downloads/44dfc51536ebc0eede0404570dae31f5d2812eb16b2eaf3ac33ad31eb8f7714f--libedit-20180525-3.1.high_sierra.bottle.tar.gz
==> Pouring libedit-20180525-3.1.high_sierra.bottle.tar.gz
==> Caveats
libedit is keg-only, which means it was not symlinked into /usr/local,
because macOS already provides this software and installing another version in
parallel can cause all kinds of trouble.

For compilers to find libedit you may need to set:
  export LDFLAGS="-L/usr/local/opt/libedit/lib"
  export CPPFLAGS="-I/usr/local/opt/libedit/include"

For pkg-config to find libedit you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/libedit/lib/pkgconfig"

==> Summary
?  /usr/local/Cellar/libedit/20180525-3.1: 53 files, 510KB

```

请注意，无论您使用哪个工具，您都必须将包含`libedit.pc`文件的目录添加到您的`PKG_CONFIG_PATH`环境变量中。在上面的屏幕截图中，Homebrew 给出了使用的命令。简单地剪切并粘贴`export`命令。注意 MacPorts 将`libedit`安装在不同的目录(`/opt/local/lib/pkgconfig`)中，因此要相应地设置变量。

有了这些繁琐的步骤，运行`make`应该可以了。您将得到的警告信息是不同的，但是您忽略它们的疏忽应该是相同的。

```
doug@dtidwell-mac:~/Developer/open-adventure $ make
./make_dungeon.py
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/local/Cellar/libedit/20180525-3.1/include -I/usr/local/Cellar/libedit/20180525-3.1/include/editline  -c main.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/local/Cellar/libedit/20180525-3.1/include -I/usr/local/Cellar/libedit/20180525-3.1/include/editline  -c init.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/local/Cellar/libedit/20180525-3.1/include -I/usr/local/Cellar/libedit/20180525-3.1/include/editline  -c actions.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/local/Cellar/libedit/20180525-3.1/include -I/usr/local/Cellar/libedit/20180525-3.1/include/editline  -c score.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/local/Cellar/libedit/20180525-3.1/include -I/usr/local/Cellar/libedit/20180525-3.1/include/editline  -c misc.c
misc.c:150:18: warning: passing an object that undergoes default argument
      promotion to 'va_start' has undefined behavior [-Wvarargs]
    va_start(ap, blank);
                 ^
misc.c:142:62: note: parameter of type 'bool' is declared here
void pspeak(vocab_t msg, enum speaktype mode, int skip, bool blank, ...)
                                                             ^
1 warning generated.
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/local/Cellar/libedit/20180525-3.1/include -I/usr/local/Cellar/libedit/20180525-3.1/include/editline  -c saveresume.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all  -c dungeon.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all  -o advent main.o init.o actions.o score.o misc.o saveresume.o dungeon.o  -L/usr/local/Cellar/libedit/20180525-3.1/lib -ledit
```

### Windows 操作系统

最后，如果你使用的是 Windows，你需要去 Cygwin 网站点击安装链接:

![The Cygwin website](img/575e90f9ac616f14b46328574fe13e72.png "The Cygwin website")

为了简单起见，只需安装`Devel`和`Python`包中的所有东西。

![Installing Cygwin packages](img/e8477aac4bed70211c7d66c602363fbd.png "Installing Cygwin packages")

完成后，只需输入`make`就可以了:

```
doug@DOUGTIDWELL4497 ~/open-adventure
$ make
./make_dungeon.py
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline  -c main.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline  -c init.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline  -c actions.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline  -c score.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline  -c misc.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all -I/usr/include/editline  -c saveresume.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all  -c dungeon.c
cc -std=c99 -D_DEFAULT_SOURCE -DVERSION=\"1.4\" -O2 -D_FORTIFY_SOURCE=2 -fstack-protector-all  -o advent main.o init.o actions.o score.o misc.o saveresume.o dungeon.o  -ledit -lcurses

```

请注意，`gcc`和`make`的 Windows/Cygwin 端口根本没有生成任何错误。

## 玩游戏

现在你可以开始玩游戏了！振作起来:巨大的洞穴探险将命令行的视觉吸引力与打字的所有兴奋结合在一起。运行`advent`可执行文件开始。首先会问你是否需要指导。键入`yes`获取一些背景信息，看看激励了一代游戏玩家的文字:

```
doug@dtidwell-mac:~/Developer/open-adventure$ advent

Welcome to Adventure!!  Would you like instructions?

> y

Somewhere nearby is Colossal Cave, where others have found fortunes in
treasure and gold, though it is rumored that some who enter are never
seen again. Magic is said to work in the cave. I will be your eyes
and hands. Direct me with commands of 1 or 2 words. I should warn
you that I look at only the first five letters of each word, so you'll
have to enter "northeast" as "ne" to distinguish it from "north".
You can type "help" for some general hints.  For information on how
to end your adventure, scoring, etc., type "info".
                 - - -
This program was originally developed by Willie Crowther.  Most of the
features of the current program were added by Don Woods.

You are standing at the end of a road before a small brick building.
Around you is a forest.  A small stream flows out of the building and
down a gully.

```

探索这座建筑似乎是一个合理的起点。您可以键入`enter building`来使事情更清楚，或者只键入单词`enter`:

```
> enter

You are inside a building, a well house for a large spring.

There are some keys on the ground here.

There is a shiny brass lamp nearby.

There is food here.

There is a bottle of water here.

```

所有这些东西听起来都很有用，所以请尽情享用吧:

```
> get keys

OK

> get lamp

OK

> get food

OK

> get water

OK

```

任何时候，您都可以使用`inventory`命令查看您携带的物品:

```
> inventory

You are currently holding the following:
Set of keys
Brass lantern
Tasty food
Small bottle
Water in the bottle

```

请注意，该命令将瓶子中的水与瓶子本身分开列出。所以也许瓶子不管有没有水都是有用的。嗯。食物，不管是什么，都很好吃。所以我们有这样的优势，这很好。

**专业提示:**按照说明中的解释，巨大洞穴探险只看一个命令的前五个字符。这意味着您可以键入`inven`来做同样的事情:

```
> inven

You are currently holding the following:
Set of keys
Brass lantern
Tasty food
Small bottle
Water in the bottle

```

这个快捷方式让你查看库存的速度提高了 44%。不客气。

**另一个提示:**这么说吧，也许把楼里的东西都捡起来不是个好主意。或许是吧。你必须玩游戏才能找到答案。

现在你已经准备好了食物，输入`leave`回到外面。

```
> leave

You're in front of building.
```

现在你可以开始探索世界了。键入`west`去，嗯，去西方:

```
> west

You have walked up a hill, still in the forest.  The road slopes back
down the other side of the hill.  There is a building in the distance.

```

(也可以只输入`w`。)

这个游戏最大的好处之一就是它需要你活跃的想象力。比如，这里的叙述暗示了远处的建筑就是你刚刚参观过的那个。它令人惊讶地引人注目，特别是考虑到它是一个 40 多年前编写的命令行游戏。为了追踪事物，大多数玩家会抓起一张纸，边走边画一张世界地图。

输入`quit`和`yes`可以让你退出游戏。除非你已经探索了游戏之外的东西，否则你会得到一个关于你可怜分数的刻薄信息:

```
> quit

Do you really want to quit now?

> y

OK

You scored 32 out of a possible 430, using 11 turns.

You are obviously a rank amateur.  Better luck next time.

To achieve the next higher rating, you need 14 more points.

```

## 我们给你的礼物:码头工人形象

作为额外的奖励，我们在 [quay.io](https://quay.io/repository/rhdp/open-adventure) 创建了一个 Docker 图像，其中包含游戏及其源代码:

[![](img/c2cac7f1d034efae77713ead6f206940.png)](https://quay.io/repository/rhdp/open-adventure)

该图像让您无需自己构建代码就可以玩游戏。如果您的机器上安装并运行了 Docker，`docker pull [quay.io/rhdp/open-adventure](https://quay.io/repository/rhdp/open-adventure?tab=tags&tag=latest)`会下载图像。从它创建一个容器，SSH 到容器中，运行`./advent`，然后就可以开始了。

## 摘要

巨大的洞穴探险对玩家和游戏产生了巨大的影响。四十年后，你仍然可以看到所有的大惊小怪是什么。像任何好游戏一样，它很复杂，需要你全神贯注。如果你已经完成了这里的步骤(或者只是调出了 Docker 图片)，你可以像你之前的成千上万的游戏玩家一样，沿着同样危险的道路前进。享受冒险吧！

*Last updated: January 6, 2023*