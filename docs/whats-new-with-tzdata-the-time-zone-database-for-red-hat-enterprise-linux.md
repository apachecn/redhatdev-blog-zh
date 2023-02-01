# tzdata 的新特性:Red Hat Enterprise Linux 的时区数据库

> 原文：<https://developers.redhat.com/blog/2020/04/03/whats-new-with-tzdata-the-time-zone-database-for-red-hat-enterprise-linux>

[时区数据库](https://www.iana.org/time-zones) ( `tzdata`)为[Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview/)(RHEL)提供特定于当地时区的数据。Linux 操作系统中的应用程序将这些数据用于各种目的。例如，GNU C 库(`glibc`)使用`tzdata`来确保像`strftime()`这样的 API 正确工作，而像`/usr/bin/date`这样的应用程序使用它来打印本地日期。

`tzdata`包包含记录全球不同时区的当前和历史转换的数据文件。此数据代表了当地政府机构或时区边界变更所要求的变更，以及协调世界时(UTC)偏移和夏令时(DST)的变更。

这篇文章是关于 2019 年`tzdata`包的变化的快速更新，以及我们正在监控的 2020 年包更新的可能时区变化。

**注**:偶尔会有`tzdata`变更在没有太多交付时间或信息不完整的情况下宣布。在这些情况下，我们尝试为不同的场景做计划，并依靠上游维护人员根据他们的经验做出决定。一旦更新准备好了，我们就更新我们的源代码，我们的质量保证和发布工程师会帮助尽快推出更新。

## 2019 年和 2020 年对 tzdata 包的更改

我们在 2019 年发布了三个对`tzdata`包的更新，大部分变化与 DST 开始和结束日期转换有关。斐济、诺福克岛、巴勒斯坦和阿拉斯加凯奇坎的 Metlakatla 印第安人社区都更改了夏令时的开始和结束日期。此外，巴西不再采用夏令时。

为了保持时区数据库提供的信息的准确性，上游项目会在新信息可用时频繁更新过去的时间戳。2019 年，我们对早在 1866 年的时间戳进行了修正。

2020 年，我们将关注与英国脱离欧盟相关的变化。

上游时区项目在[iana.org](http://www.iana.org/time-zones)发布更新，在那里你也可以找到关于订阅`tzdata`电子邮件列表和访问档案的信息。电子邮件列表包括对提议的更改和更正、发布公告以及时区相关新闻参考的讨论。

有关红帽企业版 Linux 的具体信息，请参见[红帽企业版 Linux 时区数据(tzdata)开发状态页面](https://access.redhat.com/articles/1187353)。

*Last updated: June 29, 2020*