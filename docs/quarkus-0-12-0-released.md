# Quarkus 0.12.0 发布

> 原文：<https://developers.redhat.com/blog/2019/03/20/quarkus-0-12-0-released>

下一代 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 原生 [Java](https://developers.redhat.com/topics/enterprise-java/) 框架 Quarkus 于 3 月初发布，现在 Quarkus 0.12.0 已经发布，可以从 Maven 资源库获得。[快速入门](https://github.com/quarkusio/quarkus-quickstarts)、[指南](https://quarkus.io/guides/)和[网站](https://quarkus.io)也已更新，并且 [213 期和 PRs](https://github.com/quarkusio/quarkus/releases/tag/0.12.0) 包含在此版本中。这是相当多的更新，但特别是[查看](https://quarkus.io/guides/)新的指标、健康检查和 Kafka 指南。此外，该版本需要 [GraalVM 1.0.0-RC13](https://github.com/oracle/graal/releases/tag/vm-1.0.0-rc13) 来 *[构建本地可执行文件](https://quarkus.io/guides/building-native-image-guide)* 。

**BOM 依赖**

```
  <dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-bom</artifactId>
    <version>0.12.0</version>
  </dependency>
```

**重大变化**

*   [1579]提供《卡夫卡快速入门指南》
*   [1557]从 bom 中删除测试范围
*   [1554]允许在没有@Inject 的字段上使用@ConfigProperty
*   [1548]添加微轮廓度量指南
*   [1521]介绍 JSON-B 和 JSON-P 扩展
*   [1506]改善梯度任务之间的相关性
*   [1492]将 gradle 任务重命名为 camelCase
*   [1483]添加微档案健康指南
*   [1426]支持 Microsoft SQL Server [JDBC]
*   [1417]测试改进
*   [1412]支持更广泛的字符集
*   [1395]切换到 GraalVM 1.0.0-rc13
*   [1358]土著 S2I 建筑者形象
*   [1318]添加微配置文件容错指南
*   [1]添加 Vertx 网络扩展

已经有了相当多的反馈、bug 修复请求和特性请求。夸尔库斯 0.13.0 应该在大约两周内被削减，因为夸尔库斯在大约两周的冲刺中前进。

不断收到反馈！想在这个版本中看到一个特性或错误修复吗？如果是，那么[将 0.13.0 分配给 PR 或发布](https://github.com/quarkusio/quarkus/issues)。

*Last updated: September 3, 2019*