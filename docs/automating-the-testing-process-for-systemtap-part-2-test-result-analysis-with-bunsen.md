# SystemTap 的自动化测试过程，第 2 部分:使用 Bunsen 进行测试结果分析

> 原文：<https://developers.redhat.com/blog/2021/05/10/automating-the-testing-process-for-systemtap-part-2-test-result-analysis-with-bunsen>

这是由两部分组成的系列文章的第二部分，在这个系列文章中，我描述了我为 SystemTap 项目开发的自动化测试基础设施。第一篇文章“自动化 SystemTap 的测试过程，第 1 部分:用 libvirt 和 Buildbot 进行自动化测试”描述了我管理测试机器和产生 SystemTap 测试结果的解决方案。这篇后续文章继续描述 Bunsen，这是我为存储和分析测试结果而开发的工具包。

图 1 总结了测试基础设施的组件以及它们如何交互。

[![](img/d157f68a9f3b980f66a06830c8bd9403.png "bunsen-2020-12-workflow")](/sites/default/files/blog/2020/12/bunsen-2020-12-workflow.png)

Figure 1: Components of the SystemTap testing infrastructure and their interactions.

## 重温成功测试的七个步骤

在第 1 部分中，我列出了成功测试软件项目的七个步骤:

*   步骤 1:供应测试机和虚拟机(VM)。
*   步骤 2:安装 SystemTap 项目并运行测试套件。
*   步骤 3:将测试结果发送到一个中心位置。
*   步骤 4:以紧凑的格式接收和存储测试结果。
*   步骤 5:查询测试结果。
*   步骤 6:分析测试结果。
*   步骤 7:以可读的格式报告分析。

前三个步骤与测试项目和收集测试结果有关。我在第 1 部分中讨论了这些步骤。Bunsen 工具包实现了接下来的四个步骤，这四个步骤与存储收集到的测试结果并分析它们有关。我将描述每个步骤的实现，最后总结我的关键设计思想并概述未来可能的改进。

## 步骤 4:以紧凑的格式接收和存储测试结果

一旦测试结果被测试结果服务器接收，它们必须以紧凑的格式存储，以便以后分析。为此，我开发了一个名为 [Bunsen](https://sourceware.org/git/?p=bunsen.git;a=summary) 的测试结果存储和分析工具包。

目前，Bunsen 可以解析、存储和分析那些测试套件基于 [DejaGnu](https://www.gnu.org/software/dejagnu/) 测试框架的项目的测试结果日志文件。已经用 SystemTap 测试结果对 Bunsen 进行了广泛的测试，并且还在进行工作以使 Bunsen 适应 GDB ( [GNU 调试器](https://www.gnu.org/software/gdb/))项目的需求。

Bunsen 的测试结果存储和索引引擎是基于 Git 版本控制系统的。根据以下方案，Bunsen 在 Git 存储库的几个分支中存储一组测试结果:

*   首先，Bunsen 将原始的测试结果日志文件存储在 Git 存储库中。文件存储在根据方案`*projectname*/testlogs-*year*-*month*`命名的 Git 分支中。例如，2020 年 8 月获得的 SystemTap 测试的日志文件将存储在`systemtap/testlogs-2020-08`下。这个分支中的每个修订存储一组不同的测试结果日志文件。当为一组新的测试结果创建新的修订版时，先前添加的测试结果的日志文件保留在同一分支的先前修订版中，但是从新的修订版中移除。这最小化了每个`testlogs`分支的 Git 工作副本的大小。因为一组测试结果日志文件可能包含大约 50MB 的明文数据，所以包含测试套件多次运行的测试结果日志文件的工作副本可能会变得非常大。为每个修订版本存储一组测试结果，可以检出只包含一组测试结果的工作副本。
*   Second, Bunsen parses the test result log files and produces a representation of the test results in JSON format. The JSON representation contains the results for each test case, as well as a summary of the test machine configuration. The JSON representation also contains a reference to the commit ID of the `testlogs` branch revision that stores the original test result log files. A simplified example follows showing how test results are converted to a JSON representation. Suppose the following results are contained in a `systemtap.log` file:

    ```
      ...
      Running /opt/stap-checkout/testsuite/systemtap.base/cast-scope.exp ...
      doing compile
      Executing on host: g++ /opt/stap-checkout/testsuite/systemtap.base/cast-scope.cxx  -g -isystem/opt/stap-chec\
      kout/testsuite -isystem/opt/stap-checkout/stap_install/include  -lm   -o cast-scope-m32.exe    (timeout = 300)
      spawn -ignore SIGHUP g++ /opt/stap-checkout/testsuite/systemtap.base/cast-scope.cxx -g -isystem/opt/stap-che\
      ckout/testsuite -isystem/opt/stap-checkout/stap_install/include -lm -o cast-scope-m32.exe
      pid is 26539 -26539
      output is
      PASS: cast-scope-m32 compile
      executing: stap /opt/stap-checkout/testsuite/systemtap.base/cast-scope.stp -c ./cast-scope-m32.exe
      PASS: cast-scope-m32
      doing compile
      Executing on host: g++ /opt/stap-checkout/testsuite/systemtap.base/cast-scope.cxx  -g -isystem/opt/stap-chec\
      kout/testsuite -isystem/opt/stap-checkout/stap_install/include  -lm   -o cast-scope-m32.exe    (timeout = 300)
      spawn -ignore SIGHUP g++ /opt/stap-checkout/testsuite/systemtap.base/cast-scope.cxx -g -isystem/opt/stap-che\
      ckout/testsuite -isystem/opt/stap-checkout/stap_install/include -lm -o cast-scope-m32.exe
      pid is 26909 -26909
      output is
      PASS: cast-scope-m32 compile
      executing: stap /opt/stap-checkout/testsuite/systemtap.base/cast-scope.stp -c ./cast-scope-m32.exe
      PASS: cast-scope-m32
      doing compile
      Executing on host: g++ /opt/stap-checkout/testsuite/systemtap.base/cast-scope.cxx  -g -isystem/opt/stap-chec\
      kout/testsuite -isystem/opt/stap-checkout/stap_install/include -O  -lm   -o cast-scope-m32-O.exe    (timeout = 300)
      spawn -ignore SIGHUP g++ /opt/stap-checkout/testsuite/systemtap.base/cast-scope.cxx -g -isystem/opt/stap-che\
      ckout/testsuite -isystem/opt/stap-checkout/stap_install/include -O -lm -o cast-scope-m32-O.exe
      pid is 27279 -27279
      output is
      PASS: cast-scope-m32-O compile
      executing: stap /opt/stap-checkout/testsuite/systemtap.base/cast-scope.stp -c ./cast-scope-m32-O.exe
      PASS: cast-scope-m32-O
      testcase /opt/stap-checkout/testsuite/systemtap.base/cast-scope.exp completed in 20 seconds
      Running /opt/stap-checkout/testsuite/systemtap.base/cast-user.exp ...
      doing compile
      Executing on host: gcc /opt/stap-checkout/testsuite/systemtap.base/cast-user.c  -g  -lm   -o /opt/stap-check\
      out/stap_build/testsuite/cast-user.exe    (timeout = 300)
      spawn -ignore SIGHUP gcc /opt/stap-checkout/testsuite/systemtap.base/cast-user.c -g -lm -o /opt/stap-checkou\
      t/stap_build/testsuite/cast-user.exe
      pid is 27649 -27649
      output is
      PASS: cast-user compile
      executing: stap /opt/stap-checkout/testsuite/systemtap.base/cast-user.stp /opt/stap-checkout/stap_build/test\
      suite/cast-user.exe -c /opt/stap-checkout/stap_build/testsuite/cast-user.exe
      FAIL: cast-user
      line 5: expected ""
      Got "WARNING: Potential type mismatch in reassignment: identifier 'cast_family' at /opt/stap-checkout/testsuite/systemta\
      p.base/cast-user.stp:25:5"
      " source:     cast_family = @cast(sa, "sockaddr", "<sys/socket.h>")->sa_family"
      "             ^"
      "Number of similar warning messages suppressed: 15."
      "Rerun with -v to see them."
      testcase /opt/stap-checkout/testsuite/systemtap.base/cast-user.exp completed in 6 seconds
      Running /opt/stap-checkout/testsuite/systemtap.base/cast.exp ...
      executing: stap /opt/stap-checkout/testsuite/systemtap.base/cast.stp
      FAIL: systemtap.base/cast.stp
      line 6: expected ""
      Got "WARNING: Potential type mismatch in reassignment: identifier 'cast_pid' at /opt/stap-checkout/testsuite/systemtap.b\
      ase/cast.stp:14:5"
      " source:     cast_pid = @cast(curr, "task_struct", "kernel<linux/sched.h>")->tgid"
      "             ^"
      "Number of similar warning messages suppressed: 4."
      "Rerun with -v to see them."
      testcase /opt/stap-checkout/testsuite/systemtap.base/cast.exp completed in 8 seconds
      ...

    ```

    Bunsen 生成了这些测试结果的以下 JSON 表示:

    ```
      ...
      {"name": "systemtap.base/cast-scope.exp", "outcome": "PASS",
       "origin_sum": "systemtap.sum.autotest.2.6.32-754.33.1.el6.i686.2020-12-05T03:17-0500:383",
       "origin_log": "systemtap.log.autotest.2.6.32-754.33.1.el6.i686.2020-12-05T03:17-0500:3091-3116"},
      {"name": "systemtap.base/cast-user.exp", "outcome": "FAIL", "subtest": "FAIL: cast-user\n",
       "origin_sum": "systemtap.sum.autotest.2.6.32-754.33.1.el6.i686.2020-12-05T03:17-0500:386"},
      {"name": "systemtap.base/cast.exp", "outcome": "FAIL", "subtest": "FAIL: systemtap.base/cast.stp\n",
       "origin_sum": "systemtap.sum.autotest.2.6.32-754.33.1.el6.i686.2020-12-05T03:17-0500:388"},
      ...

    ```

    JSON 表示存储在根据方案`*projectname*/testruns-*year*-*month*`命名的分支中，例如`systemtap/testruns-2020-08`。

    因为 JSON 表示比原始的测试套件日志文件更紧凑，所以在`testruns`分支中的最新修订可以保留存储在该分支中的所有测试套件运行。这使得检查总结一个月所有测试结果的工作副本成为可能。

*   最后，Bunsen 存储了包含测试机器配置的 JSON 表示的概要，但没有详细的测试结果。该表示被附加到分支`index`中的文件`*projectname*-*year*-*month*.json`中。因此，要获得 Bunsen 存储库中所有测试结果的摘要，只需检查这个`index`分支的工作副本就足够了。

图 2 总结了 Bunsen 测试结果存储库的布局。

[![](img/bda75763551db616b77ebb034deaf15f.png "bunsen-2020-12-layout")](/sites/default/files/blog/2020/12/bunsen-2020-12-layout.png)

Figure 2: Layout of a Bunsen test result repository.

Git 存储库格式为存储测试结果日志文件提供了显著的空间优势，因为不同测试结果集之间有很大程度的相似性。在大量的测试套件运行中，相同的测试用例结果和相同的测试用例输出倾向于重复出现，只有微小的变化。Git 的内部表示法[可以将多个测试结果打包成一个压缩的表示法](https://git-scm.com/book/en/v2/Git-Internals-Packfiles)，其中重复出现的测试用例结果被去重。

例如，从测试套件的 1，562 次运行中收集的 SystemTap 测试结果日志在未压缩存储时占用了 103GB 的磁盘空间。Bunsen 生成的相同测试结果的去重复 Git 存储库仅占用 2.7GB。该存储库包括原始测试结果日志和测试结果的 JSON 表示。

## 步骤 5:查询测试结果

除了存储和索引引擎之外，Bunsen 还包含了一个分析脚本的集合，可以用来从测试结果存储库中提取信息。分析脚本是 Python 程序，它使用 Bunsen 的存储引擎来访问存储库。

Bunsen 包含的一些分析脚本旨在查询关于特定测试运行内容的信息。下面的命令示例演示了如何定位和检查最近一次测试运行的结果，以及如何将它们与以前运行的结果进行比较。

[list_runs](https://sourceware.org/git/?p=bunsen.git;a=blob;f=scripts-main/list_runs.py) 和 [list_commits](https://sourceware.org/git/?p=bunsen.git;a=blob;f=scripts-main/list_commits.py) 脚本列出存储在测试结果存储库中的测试运行，并显示测试结果日志文件的 Git commit ID。`list_runs`脚本简单地列出了所有的测试运行；`list_commits`获取 SystemTap Git 存储库的检出位置，并列出测试存储库主分支中每个提交的测试运行:

```
$ ./bunsen.py +list_commits source_repo=/opt/stap-checkout sort=recent restrict=3
commit_id: 1120422c2822be9e00d8d11cab3fb381d2ce0cce 
 summary: PR27067 <<< corrected bug# for previous commit
* 2020-12 5d53f76... pass_count=8767 fail_count=1381 arch=x86_64     osver=rhel-6
* 2020-12 c88267a... pass_count=8913 fail_count=980 arch=i686 osver=rhel-6
* 2020-12 299d9b4... pass_count=7034 fail_count=2153 arch=x86_64     osver=ubuntu-18-04
* 2020-12 53a18f0... pass_count=9187 fail_count=1748 arch=x86_64     osver=rhel-8
* 2020-12 7b37e97... pass_count=8752 fail_count=1398 arch=x86_64     osver=rhel-6
* 2020-12 8cd3b77... pass_count=8909 fail_count=1023 arch=i686 osver=rhel-6
* 2020-12 a450958... pass_count=10714 fail_count=2457 arch=x86_64     osver=rhel-7
* 2020-12 50dc14a... pass_count=10019 fail_count=3590 arch=x86_64     osver=fedora-31
* 2020-12 6011ae2... pass_count=9590 fail_count=2111 arch=i686     osver=fedora-30
* 2020-12 771bf86... pass_count=9976 fail_count=2617 arch=x86_64     osver=fedora-32
* 2020-12 1335891... pass_count=10013 fail_count=2824 arch=x86_64     osver=fedora-34

commit_id: 341bf33f14062269c52bcebaa309518d9972ca00 
 summary: staprun: handle more and fewer cpus better
* 2020-12 368ee6f... pass_count=8781 fail_count=1372 arch=x86_64     osver=rhel-6
* 2020-12 239bbe9... pass_count=8904 fail_count=992 arch=i686 osver=rhel-6
* 2020-12 b138c0a... pass_count=6912 fail_count=2276 arch=x86_64     osver=ubuntu-18-04
* 2020-12 bf893d8... pass_count=9199 fail_count=1760 arch=x86_64     osver=rhel-8
* 2020-12 8be8643... pass_count=10741 fail_count=2450 arch=x86_64     osver=rhel-7
* 2020-12 94c84ab... pass_count=9996 fail_count=3662 arch=x86_64     osver=fedora-31
* 2020-12 a06bc5f... pass_count=10139 fail_count=2712 arch=x86_64     osver=fedora-34
* 2020-12 8d4ad0e... pass_count=10042 fail_count=2518 arch=x86_64     osver=fedora-32
* 2020-12 b6388de... pass_count=9591 fail_count=2146 arch=i686     osver=fedora-30

commit_id: a26bf7890196395d73ac193b23e271398731745d 
 summary: relay transport: comment on STP_BULK message
* 2020-12 1b91c6f... pass_count=8779 fail_count=1371 arch=x86_64     osver=rhel-6
* 2020-12 227ff2b... pass_count=8912 fail_count=983 arch=i686 osver=rhel-6
...

```

给定一个测试运行的 Git 提交 ID， [show_logs](https://sourceware.org/git/?p=bunsen.git;a=blob;f=scripts-main/show_logs.py) 脚本显示该测试运行的测试结果日志文件的内容。例如，下面的`show_logs`调用检查 Fedora 32 上最新 SystemTap 提交的测试结果。在 Bunsen 存储库中，这些测试结果有一个 Git 提交 ID`771bf86`:

```
$ ./bunsen.py +show_logs testrun=771bf86 "key=systemtap.log*" | less
...
Running /opt/stap-checkout/testsuite/systemtap.maps/map_hash.exp ...
executing: stap /opt/stap-checkout/testsuite/systemtap.maps/map_hash_II.stp     -DMAPHASHBIAS=-9999
FAIL: systemtap.maps/map_hash_II.stp -DMAPHASHBIAS=-9999
line 1: expected "array[2048]=0x1"
Got "ERROR: Couldn't write to output 0 for cpu 1, exiting.: Bad file     descriptor"
executing: stap /opt/stap-checkout/testsuite/systemtap.maps/map_hash_II.stp     -DMAPHASHBIAS=-9999 --runtime=dyninst
FAIL: systemtap.maps/map_hash_II.stp -DMAPHASHBIAS=-9999 --runtime=dyninst
line 1: expected "array[2048]=0x1"
Got "array[2048]=1"
    "array[2047]=1"
...

```

除了直接查看测试结果之外，我们通常想要将它们与来自项目的已知基线版本的测试结果进行比较。对于 SystemTap 项目，合适的基线版本应该是 SystemTap 的最新版本。为了找到基线，`list_commits`脚本可以再次用于列出 SystemTap 提交在 4.4 版前后的测试结果:

```
$ pushd /opt/stap-checkout
$ git log --oneline
...
931e0870a releng: update-po
1f608d213 PR26665: relayfs-on-procfs megapatch, rhel6 tweaks
988f439af (tag: release-4.4) pre-release: version timestamping, NEWS tweaks
f3cc67f98 pre-release: regenerate example index
1a4c75501 pre-release: update-docs
...
$ popd
$ ./bunsen.py +list_commits source_repo=/notnfs/staplogs/upstream-systemtap sort=recent | less
...
commit_id: 931e0870ae203ddc80d04c5c1425a6e3def49c38
 summary: releng: update-po
* 2020-11 6a3e8b1... pass_count=9391 fail_count=760 arch=x86_64 osver=rhel-6
* 2020-11 349cdc9... pass_count=9096 fail_count=800 arch=i686 osver=rhel-6
* 2020-11 136ebbb... pass_count=8385 fail_count=818 arch=x86_64     osver=ubuntu-18-04
* 2020-11 f1f93dc... pass_count=9764 fail_count=1226 arch=x86_64     osver=rhel-8
* 2020-11 2a70a1d... pass_count=10094 fail_count=1510 arch=x86_64     osver=fedora-33
* 2020-11 45aabfd... pass_count=11817 fail_count=1357 arch=x86_64     osver=rhel-7
* 2020-11 cfeee53... pass_count=9958 fail_count=1785 arch=i686     osver=fedora-30
* 2020-11 838831d... pass_count=11085 fail_count=2569 arch=x86_64     osver=fedora-31
* 2020-11 e07dd16... pass_count=10894 fail_count=1734 arch=x86_64     osver=fedora-32

commit_id: 988f439af39a359b4387963ca4633649866d8275
summary: pre-release: version timestamping, NEWS tweaks
* 2020-11 2f928a5... pass_count=9877 fail_count=1270 arch=aarch64     osver=rhel-8
* 2020-11 815dfc0... pass_count=11831 fail_count=1345 arch=x86_64     osver=rhel-7
* 2020-11 f2ca771... pass_count=11086 fail_count=2562 arch=x86_64     osver=fedora-31
* 2020-11 5103bb5... pass_count=8400 fail_count=804 arch=x86_64     osver=ubuntu-18-04
* 2020-11 a5745c3... pass_count=9717 fail_count=1216 arch=x86_64     osver=rhel-8
* 2020-11 64de18e... pass_count=10094 fail_count=1464 arch=x86_64     osver=fedora-33
* 2020-11 b23e63d... pass_count=10895 fail_count=1733 arch=x86_64     osver=fedora-32

commit_id: 1a4c75501e873d1281d6c6c0fcf66c0f2fc1104e
summary: pre-release: update-docs
* 2020-11 78a1959... pass_count=8383 fail_count=819 arch=x86_64     osver=ubuntu-18-04
* 2020-11 7fd7908... pass_count=9753 fail_count=1214 arch=x86_64     osver=rhel-8
* 2020-11 935b961... pass_count=10092 fail_count=1509 arch=x86_64     osver=fedora-33
...

```

[diff_runs](https://sourceware.org/git/?p=bunsen.git;a=blob;f=scripts-main/diff_runs.py) 脚本比较来自两个不同测试运行的测试结果。例如，下面的`diff_runs`调用将 Fedora 32 x86_64 上的最新测试结果与 4.4 版前后同一系统上的测试结果进行了比较:

```
$ ./bunsen.py +diff_runs baseline=b23e63d latest=771bf86 | less
...
* PASS=>FAIL systemtap.maps/map_hash.exp FAIL:     systemtap.maps/map_hash_stat_II.stp
* PASS=>FAIL systemtap.maps/map_hash.exp FAIL:     systemtap.maps/map_hash_stat_SI.stp
* PASS=>FAIL systemtap.maps/map_hash.exp FAIL:     systemtap.maps/map_hash_stat_SSI.stp -DMAPHASHBIAS=2
* PASS=>FAIL systemtap.maps/map_wrap.exp FAIL: systemtap.maps/map_wrap2.stp
...

```

最后， [diff_commits](https://sourceware.org/git/?p=bunsen.git;a=blob;f=scripts-main/diff_commits.py) 脚本比较对应于两个不同修订的所有测试结果。`diff_commits`试图选择匹配的系统配置对进行比较，而不是为每对测试运行提供单独的比较。

`diff_commits`的输出列出了每一个被改变的测试用例一次，根据它们被改变的比较集合对测试用例进行分组。

例如，下面的`diff_commits`调用将最近 SystemTap commit 341bf33 的所有测试运行与 4.4 版 commit 988f439 的所有测试运行进行了比较:

```
$ ./bunsen.py +diff_commits source_repo=/notnfs/staplogs/upstream-systemtap baseline=988f439     latest=1120422 | less
...
Regressions by version

Found 9 regressions for:
(arch=x86_64 osver=rhel-7) -> (arch=x86_64 osver=rhel-6)
(arch=x86_64 osver=rhel-7) -> (arch=i686 osver=rhel-6)
(arch=x86_64 osver=rhel-8) -> (arch=x86_64 osver=rhel-8)
(arch=x86_64 osver=rhel-7) -> (arch=x86_64 osver=rhel-7)
(arch=x86_64 osver=fedora-31) -> (arch=x86_64 osver=fedora-31)
(arch=x86_64 osver=rhel-7) -> (arch=i686 osver=fedora-30)
(arch=x86_64 osver=fedora-32) -> (arch=x86_64 osver=fedora-32)
(arch=x86_64 osver=rhel-7) -> (arch=x86_64 osver=fedora-34)
* PASS=>ERROR systemtap.apps/busybox.exp ERROR: tcl error sourcing     /notnfs/smakarov/stap-checkout/testsuite/systemtap.apps/busybox.exp.
* PASS=>FAIL systemtap.onthefly/kprobes_onthefly.exp FAIL: kprobes_onthefly     - otf_start_disabled_iter_4 (stap)
* PASS=>KFAIL systemtap.onthefly/tracepoint_onthefly.exp KFAIL:     tracepoint_onthefly - otf_timer_10ms (stap) (PRMS: 17256)
* KFAIL=>PASS systemtap.onthefly/tracepoint_onthefly.exp KFAIL:     tracepoint_onthefly - otf_timer_10ms (invalid output) (PRMS: 17256)
* PASS=>FAIL systemtap.printf/end1b.exp FAIL: systemtap.printf/end1b
* PASS=>FAIL systemtap.printf/mixed_outb.exp FAIL:     systemtap.printf/mixed_outb
* PASS=>FAIL systemtap.printf/out1b.exp FAIL: systemtap.printf/out1b
* PASS=>FAIL systemtap.printf/out2b.exp FAIL: systemtap.printf/out2b
* PASS=>FAIL systemtap.printf/out3b.exp FAIL: systemtap.printf/out3b

Found 3 regressions for:
(arch=x86_64 osver=rhel-7) -> (arch=x86_64 osver=rhel-6)
(arch=x86_64 osver=fedora-31) -> (arch=x86_64 osver=fedora-31)
* FAIL=>PASS systemtap.apps/java.exp FAIL: multiparams (0)
* FAIL=>PASS systemtap.apps/java.exp FAIL: multiparams 3.0 (0)
* PASS=>FAIL systemtap.base/ret-uprobe-var.exp FAIL: ret-uprobe-var: TEST 1: @var in return probes should not be stale (4.1+) (kernel): stderr: string should be "", but got "ERROR: Couldn't write to output 0 for cpu 2, exiting.: Bad file descriptor
...

```

因为测试结果以压缩格式存储在 Git 存储库中，所以在调用分析脚本和接收结果之间存在明显的延迟。对`+list_runs`、`+list_commits`或`+diff_runs`的调用大约需要 1 秒钟才能完成。调用`+diff_commits`脚本必须提取和比较多组测试结果，大约需要 10 秒钟才能完成。

## 步骤 6:分析测试结果

当我们查看最近的一组测试结果时，并不总是清楚哪些测试失败是新的，哪些测试失败在项目的历史中重复出现。如果测试失败在许多组测试结果中重复出现，了解失败在项目测试历史中第一次出现的时间是很有帮助的。

作为过滤掉重复测试失败的初始解决方案，我开发了一个名为 [find_regressions](https://sourceware.org/git/?p=bunsen.git;a=blob;f=scripts-main/find_regressions.py) 的分析脚本。这个脚本按照时间顺序遍历项目的 Git 历史，并检查每个修订的测试结果，以识别测试结果中的新变化。

如果在可配置数量的先前提交内(例如，在最后 10 次先前提交内)已经发生了至少一次变更，则认为它是已经发生的变更。否则，它被认为是新发生的变化。

这种分析过滤掉不确定的测试用例，这些测试用例的结果经常在“通过”和“失败”结果之间来回变化。`find_regressions`的最终输出列出了与每次提交相关联的新发生的变更，以及每次变更在后续提交后出现的次数。

例如，[这个通过调用`find_regressions`生成的 HTML 文件](https://people.redhat.com/smakarov/2021-bunsen-blog-1/find-regressions-bpf.html)总结了 SystemTap 项目中 25 个最新修订的 [bpf.exp](https://sourceware.org/git/?p=systemtap.git;a=blob;f=testsuite/systemtap.bpf/bpf.exp) 测试用例中新出现的回归。

当然，测试结果分析的理想目标应该是精确地识别由每个提交或者测试环境中的每个变化引起的测试结果变化。由于 SystemTap 测试套件的性质，准确识别测试结果中每个变化的原因是一个困难的问题。过滤掉重复的失败可以被认为是解决这个问题的第一步。

## 步骤 7:以可读的格式报告分析

上一节中的示例演示了如何通过登录到测试结果服务器并从命令行调用分析脚本来查询 Bunsen 测试结果存储库。然而，这不是检查测试结果的最方便的方式。最好是通过 web 界面远程访问测试结果。

Bunsen 包含的分析脚本包括一个以 HTML 格式生成测试结果的选项。此外，Bunsen toolkit 包括一个 CGI 脚本， [bunsen-cgi.py](https://sourceware.org/git/?p=bunsen.git;a=blob;f=server/bunsen-cgi.py) ，它接受运行 Bunsen 分析脚本的请求，并返回 HTML 版本的分析结果。

例如，[这个 HTML 文件](https://people.redhat.com/smakarov/2021-bunsen-blog-1/list-commits.html)是通过调用`list_commits`脚本生成的，它总结了 SystemTap 项目中 25 个最新修订的测试结果。

在为 Bunsen 分析输出开发 HTML 选项时，我选择保持简单的格式有两个原因。首先，我想避免与更复杂的交互式 web 控制台(如 Grafana)相关的开发和维护开销。第二，我希望 Bunsen 生成的 HTML 文件是独立的、可查看的，而不必向测试结果服务器发出额外的请求，这样分析输出就可以保存到磁盘，通过电子邮件共享，并附加到错误报告中。

## 第二部分的结论

通过仔细分析 SystemTap 项目的测试需求，我能够消除产生和存储测试结果所需的大部分人工工作。在这个过程中，我不得不为许多困难开发解决方案。

首先，我需要在各种 Linux 环境中测试 SystemTap。为了解决这个问题，我使用了简单的 shell 脚本和 [libvirt](https://libvirt.org) 框架的广泛自动化支持来自动执行以前必须手动完成的步骤，以设置 SystemTap 测试机器。

第二，因为 SystemTap 的测试结果日志文件很大，所以我需要一种压缩系数很大的存储格式。为了解决这个问题，我设计了 Bunsen toolkit 来利用 [Git 的去重复存储格式](https://git-scm.com/book/en/v2/Git-Internals-Packfiles)。当查询收集的测试结果时，这种格式以明显的延迟为代价实现了很大程度的压缩。将来，可以通过在压缩的 Git 存储库和访问它的分析脚本之间添加一个缓存层来减少延迟。

第三，大量的测试结果和大量的不确定性测试用例意味着即使是仔细检查收集的测试结果以发现新的失败也是一项耗时的手工任务。我决定开发一个基于改进的测试结果分析的解决方案，而不是重做整个 SystemTap 测试套件来消除不确定的和依赖于环境的测试用例。在未来，我希望改进我的分析脚本，并找到更有效的方法来过滤和报告项目测试结果中的新变化。

测试结果分析的难题并不是 SystemTap 项目所独有的。许多其他开源程序都有非常大的测试套件，并且包含不可预测行为的测试用例。返工测试套件以删除这些测试用例将占用开发人员大量宝贵的时间。因此，开发工具使跨各种项目分析收集的测试结果变得更容易是值得的。Bunsen 工具包中的测试结果存储和分析功能是朝着这个方向迈出的第一步。

*Last updated: May 7, 2021*