# Quarkus 0.17.0 现已推出

> 原文：<https://developers.redhat.com/blog/2019/06/25/quarkus-0-17-0-now-available>

Quarkus 继续其每 2-3 周发布一次的节奏。这个最新版本(0.17.0)包含了 [125 个以上的变化](https://github.com/quarkusio/quarkus/releases/tag/0.17.0)，其中包括新特性、错误修复和文档更新。

值得注意的主要变化包括:

*   **新 AMQP 支线**。 [AMQP 指南](https://quarkus.io/guides/amqp-guide)和[快速入门](https://github.com/quarkusio/quarkus-quickstarts/tree/master/amqp-quickstart)涵盖了如何在 AMQP 使用 MicroProfile 反应式消息传递。
*   **新阿帕奇 Kafka Streams 扩展**。使用 Kafka Streams API 创建流查询。查看[快速入门](https://github.com/quarkusio/quarkus-quickstarts/tree/master/kafka-streams-quickstart)。
*   **新的 Kogito(业务自动化)扩展**。[指南](https://quarkus.io/guides/kogito-guide)展示了如何使用 Kogito([jBPM](http://www.jbpm.org)+[Drools](http://drools.org/))向应用程序添加业务自动化。
*   **集成 Hibernate ORM 和 Hibernate Validator** 。[改进了](https://github.com/quarkusio/quarkus/issues/1889) Hibernate Validator 约束与 SQL 生成的集成。
*   **SmallRye MicroProfile 健康检查扩展 2.0 支持**。MicroProfile Health Check 2.0(参见[规范 PDF)](https://github.com/eclipse/microprofile-health/releases/tag/2.0) 增加了@Liveness 和@Readiness 注释，同时取消了@Health 注释。

体验[开发者的喜悦](https://quarkus.io/vision/developer-joy)同时尝试新的扩展，当然，编译成本地可执行文件以实现超音速、亚原子 Java。

*Last updated: July 1, 2019*