# 2020 时区数据库(tzdata)更改

> 原文：<https://developers.redhat.com/blog/2020/12/25/2020-time-zone-database-tzdata-changes>

夏令时转换、时区名称更改和一些过时文件的删除:这些是时区数据库(tzdata)包中发生的一些更改，该包为 Red Hat Enterprise Linux (RHEL)和应用程序提供时区信息。

GNU C 库(glibc)利用 tzdata 包来使 strftime()等 API 正常工作，而/usr/bin/date 等应用程序则利用这些信息来打印本地日期。tzdata 包包含记录世界各地不同时区的当前和历史转换的数据文件。此数据代表了当地政府机构或时区边界变化所要求的更改，以及 UTC 偏移量和夏令时(DST)的更改。

2020 年，tzdata 包有四个上游版本:

*   大多数更改与夏令时(DST)开始和结束日期转换有关。其中包括非洲/卡萨布兰卡、南极洲/凯西、太平洋/斐济、亚洲/加沙和亚洲/希伯伦。此外，由美洲/怀特霍斯和美洲/道森时区代表的加拿大育空地区永久使用 UTC-07。
*   区域名称很少改变，但在 2020 年，上游项目更名为 America/Godthab，以更好地反映英语的当前用法，即现在的 America/Nuuk。
*   2020 年删除了一些过时的文件。其中包括 pacificnew、systemv 和 yearistype.sh，但是，我们发现删除 pacificnew 可能会导致某些应用程序失败。在 tzdata 的 rearguard 数据格式版本中添加了 pacificnew 文件的空版本，以防止这些故障。关于后防线数据格式的更多信息，请参见 *[时区数据(tzdata): 2018 数据格式变化和红帽企业 Linux](https://developers.redhat.com/blog/2019/02/22/time-zone-data-tzdata-2018-data-format-changes-and-red-hat-enterprise-linux/) 。*
*   上游项目提供的 zic 版本也有一些变化。然而，这些不会影响 Fedora 或 RHEL 用户，因为我们使用 glibc 提供的 zic 版本。

为了保持时区数据库提供的信息的准确性，上游项目会在新信息可用时频繁更新过去的时间戳。2020 年，对远至 1891 年的过去时间戳进行了几次修正。

有时，时区变更会在没有太多准备时间或信息不完整的情况下宣布。在这些情况下，我们尝试为不同的场景做计划，并依靠上游维护人员根据他们的经验做出决定。一旦一个更新准备好了，我们更新我们的源代码和我们的质量保证，发布工程师帮助尽快推出更新。

上游时区项目在 iana.org 网站上发布更新。您还可以在该网站上找到有关订阅 tzdata 电子邮件列表和访问存档的信息。电子邮件列表包括对提议的更改和更正、发布公告以及时区相关新闻参考的讨论。

有关 Red Hat Enterprise Linux tzdata 更新的更多信息，请参见 [Red Hat Enterprise Linux 时区数据(tzdata) -开发状态页面。](https://access.redhat.com/articles/1187353)

*Last updated: December 21, 2020*