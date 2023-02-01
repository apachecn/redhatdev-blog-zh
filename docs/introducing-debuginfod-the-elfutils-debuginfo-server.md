# 介绍 debuginfod，elfutils debuginfo 服务器

> 原文：<https://developers.redhat.com/blog/2019/10/14/introducing-debuginfod-the-elfutils-debuginfo-server>

因为 bug 是不可避免的，开发人员需要快速方便地访问调试工具如 [Systemtap](https://sourceware.org/systemtap/) 和 [GDB](https://www.gnu.org/software/gdb/) 所依赖的工件，这些工件通常是 [DWARF](http://dwarfstd.org/) (使用属性化记录格式进行调试)debuginfo 或源文件。在调试您自己的本地构建树时，访问这些资源应该不成问题，但是它们经常不容易获得。

例如，您的发行版可能[将 debuginfo](https://access.redhat.com/solutions/9907) 和源文件与您试图调试的可执行文件分开打包，而您可能缺少安装这些包的权限。或者，也许您正在一个不是用这些资源构建的容器中进行调试，或者您只是不想让这些文件占用您机器上的空间。

Debuginfo 文件因占用大量空间而臭名昭著，它们的大小是相应的可执行文件的五到十五倍并不罕见。`debuginfod`旨在解决这些问题。

## 什么是 debuginfod，它是如何工作的？

`debuginfod`是一个 HTTP 文件服务器，为类似调试器的工具提供调试资源。服务器定期扫描目录树和 RPM 档案，以提取找到的任何可执行文件和 debuginfo 文件的构建 id。它包含一个 SQLite 数据库，该数据库将构建 id 索引到文件名或(包、内容)元组。

构建 ID 是作为 ELF 注释嵌入到目标文件中的唯一散列代码:

```
$ eu-readelf -n foo | grep Build.ID
    Build ID: be1743a97c6afb4a066a93c57499e9fba41cbcd5

```

这些 IDs 在 GCC 中已经默认启用了 10 年，并且得到了 LLVM 的支持。更多信息，请参见 [Fedora wiki](https://fedoraproject.org/wiki/Releases/FeatureBuildId) 。

`debuginfod`还提供与特定构建 ID 相关联的源文件。因为源文件本身不包含构建 ID，`debuginfod`依赖于包含在相应的 debuginfo 或可执行文件中的构建 ID 和源路径信息来确定源文件的位置。

## 我如何使用 debuginfod？

这里有一个例子:

```
$ debuginfod -vv -R /var/tmp/rpmbuild -F /usr/lib/debug
[...] Opened database /home/amerey/.debuginfod.sqlite
[...] Searching for ELF/DWARF files under /usr/lib/debug
[...] Searching for RPMs under /var/tmp/rpmbuild
[...] fts/rpm traversed /var/tmp/rpmbuild [...] debuginfo=386, executable=0, sourcerefs=34119, sourcedefs=3398
[...] fts traversed /usr/lib/debug [...] debuginfo=8071, executables=2, source=1958339
[...] Started http server on IPv4 IPv6 port=8002

```

要查询服务器，我们可以使用与`debuginfod`打包在一起的`debuginfod-find`命令行工具。`debuginfod-find`命令查询包含在`$DEBUGINFOD_URLS`环境变量中的 URL，以获取特定的 debuginfo、可执行文件或源文件。如果成功检索到文件，该命令会将其存储在本地缓存中，并打印文件的路径:

```
$ export DEBUGINFOD_URLS="http://buildhost:8002/"
$ debuginfod-find source BUILD-ID /path/to/foo.c
/home/amerey/.debuginfod_client_cache/source#path#to#foo#c
$ debuginfod-find debuginfo BUILD-ID
/home/amerey/.debuginfod_client_cache/debuginfo

```

`debuginfod`还包括一个共享库`libdebuginfod`，它使工具能够只使用一个构建 ID(和路径，如果试图获取一个源文件的话)来查询`debuginfod`服务器的 debuginfo、可执行文件或源文件。与`debuginfod-find`一样，文件从服务器下载到本地缓存中，并提供给工具使用，不需要任何特殊许可。

将`debuginfod`客户端支持添加到工具的一个选项是在代码中添加一个后备，当工具不能定位文件时，向服务器查询文件。这个实现允许添加`debuginfod`功能，而不必改变工具的通常行为(当然，虽然`libdebuginfod`可以以开发者认为合适的任何方式集成)。

我们已经为 elfutils 和 GDB 开发了原型`debuginfod`客户端(参见“我如何获得 debuginfod？”一节了解更多信息)。以下代码块基于一个补丁，该补丁为 GDB 的 debuginfo 查找例程添加了`debuginfod`功能。这个原型展示了`debuginfod`客户端功能只需要少量代码:

```
#if HAVE_LIBDEBUGINFOD 
  if ([separate debuginfo should exist but was not found])
    {
      const struct bfd_build_id *build_id;
      char *debugfile_path;

      build_id = build_id_bfd_get (objfile->obfd);
      int fd = debuginfod_find_debuginfo (build_id->data,
                                          build_id->size,
                                          &debugfile_path);

      if (fd >= 0)
        {
          /* debuginfo successfully retrieved from server.  */
          gdb_bfd_ref_ptr debug_bfd (symfile_bfd_open (debugfile_path));
          symbol_file_add_separate (debug_bfd.get (), debugfile_path,
                                    symfile_flags, objfile);
          close (fd);
          free (debugfile_path);
        }
    }
#endif /* LIBDEBUGINFOD */

```

我们将目标 debuginfo 文件的构建 ID 传递给`debuginfod_find_debuginfo()`。该函数向`debuginfod`服务器查询文件，如果成功检索到，文件描述符和文件本地副本的路径对 GDB 可用，它使用[二进制文件描述符(BFD)](https://en.wikipedia.org/wiki/Binary_File_Descriptor_library) 库打开文件，并将其与我们试图调试的相应目标文件相关联。

此外，基于 elfutils 的工具(如 Systemtap)会自动从 elfutils `debuginfod`客户端继承`debuginfod`功能。假设我们试图用一个可执行文件`foo`来运行`stap`和`gdb`命令，对于这个可执行文件，我们在本地没有 debuginfo 或源代码。我们还缺少 Glibc debuginfo 和源文件:

```
$ stap -e 'probe process("/path/to/foo").function("*") { [...] }'
semantic error: while resolving probe point: identifier 'process' at t.stp:1:7
            source: probe process("/path/to/foo").function("*") {
semantic error: no match
$ gdb /path/to/foo
[...]
Reading symbols from /path/to/foo...
(No debugging symbols found in /path/to/foo)
```

我们可以用`foo`的构建树和 Glibc debuginfo RPM 在远程机器上启动一个`debuginfod`服务器，并让它们对本地机器上的`stap`和`gdb`实例可用:

```
$ debuginfod -p PORT -F foo_build/ -R debug_rpms/
[...]
[...] Started http server on IPv6 IPv6 port=PORT
```

```
$ export DEBUGINFOD_URLS="http://foobuildhost:PORT/"
$ stap -v -e probe process("/path/to/foo").function("*") { [...] }'
[...]
Pass 5: starting run
^C
$ gdb /path/to/foo
[...]
Reading symbols from /home/amerey/.debuginfod_client_cache/debuginfo...
(gdb) break printf
[...]
(gdb) run
[...]
Breakpoint 1, __printf (format=0x40201e, "main\n") at printf.c:28
28        {
(gdb) list
[...]
26        int
27        __printf (const char *format, ...)
28        {
29          va_list arg;
30          int done;
[...]
```

现在我们能够从一个`debuginfod`服务器获取必要的文件，我们能够成功地用`stap`探测`foo`(如“第五关:开始运行”所示)，用`gdb`我们能够调试和查看`foo`的源代码，以及它对 C 库函数的任何调用。如果服务器找不到目标文件，`debuginfod`可以很容易地配置为将请求委托给其他`debuginfod`服务器。为此，只需在`$DEBUGINFOD_URL`环境变量中包含其他`debuginfod`服务器的 URL。

## 我如何得到 debuginfod？

目前，`debuginfod`服务器、命令行界面和共享库的原型——以及文档——可以从 [elfutils Git 库](https://sourceware.org/git/?p=elfutils.git;a=shortlog;h=refs/heads/debuginfod)获得。`debuginfod`计划包含在未来的 elfutils 版本中。GDB Git 库的一个实验分支[上有一个原型 GDB 客户端。`debuginfod`二进制文件也可以在](https://sourceware.org/git/?p=binutils-gdb.git;a=shortlog;h=refs/heads/users/fche/dbgserver) [Fedora COPR](https://copr.fedorainfracloud.org/coprs/fche/elfutils/) 上获得，可以从命令行下载，如下所示:

```
yum copr enable fche/elfutils
yum -y install elfutils-debuginfod
```

## debuginfod 的下一步是什么？

我们正在努力将`debuginfod`客户端功能添加到其他工具中，例如 [binutils](https://www.gnu.org/software/binutils/) 和 [LLDB](https://lldb.llvm.org/) 。我们还计划支持 [Debian 包格式](https://en.wikipedia.org/wiki/Deb_(file_format))，在 Fedora 的 [Koji 构建系统](https://koji.fedoraproject.org/koji/)上运行`debuginfod`服务器，并扩展`debuginfod`的 web API 以支持 DWARF 内容查询。

总是感谢帮助或反馈。请通过 elfutils-devel@sourceware.org 或 irc.freenode.net 的`#elfutils`频道与我们联系。

*Last updated: July 1, 2020*