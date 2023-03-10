# CodeMirror 中的 Apache Camel 语言支持

> 原文：<https://developers.redhat.com/blog/2019/10/02/apache-camel-language-support-in-codemirror>

在之前的一篇文章中，[我提到了](https://developers.redhat.com/blog/2019/09/17/a-look-at-development-environments-with-specific-tooling-for-apache-camel-language/)越来越多的支持 [Apache Camel](https://camel.apache.org/) 语言的 IDEs 编辑器。我很高兴地宣布，这一套再次增长。阿帕奇骆驼现在可以使用[代码镜像](https://codemirror.net/)。CodeMirror 是一个轻量级、可嵌入的 web 浏览器编辑器。

这个[库](https://www.youtube.com/redirect?q=https%3A%2F%2Fgithub.com%2Fcamel-tooling%2Fcamel-lsp-client-codemirror&redir_token=bOjTCSEyHPMub--ozWPfv8w6w0x8MTU2OTQ4MzY3OUAxNTY5Mzk3Mjc5&event=video_description&v=JOJW5liL22g)展示了如何使用 CodeMirror。基于这个存储库示例，您可以在此视频中观看这些功能的运行情况:

[https://www.youtube.com/embed/JOJW5liL22g?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/JOJW5liL22g?autoplay=0&start=0&rel=0)

## 在幕后

使用[语言服务器协议(LSP)](https://microsoft.github.io/language-server-protocol/) ，可以通过标准输入/输出或 WebSocket 进行通信。CodeMirror 仅支持 WebSocket 连接。Camel 语言服务器仅支持标准输入/输出。然后，Camel 语言服务器需要支持 WebSocket 连接。这个添加是使用新的 [LSP4J API](https://github.com/camel-tooling/camel-language-server/blob/727f7a88a140041f452151da45df7b9740c5c34e/src/main/java/com/github/cameltooling/lsp/internal/websocket/CamelLSPWebSocketEndpoint.java#L28-L43) 和 Java 中 WebSocket 的参考实现 [Tyrus](https://tyrus-project.github.io/) 完成的。想法是尽可能保持实现的简单和轻量级。

## 支持另一个 IDE/编辑器？

既然 Camel 语言服务器支持 WebSocket 连接，这就为将 Apache Camel 集成到其他只支持 WebSocket 的客户端提供了可能性，比如 [MS Monaco 编辑器。](https://github.com/Microsoft/monaco-editor)

添加对 CodeMirror 的支持无疑是最困难的任务之一，因为它需要在 Camel 语言服务器端实现 WebSocket 支持，而我只花了几天时间。因此，我鼓励其他 IDEs 编辑器的用户测试 Camel LSP 的性能，然后分享使用它们的步骤。你可以在这里找到潜在客户的列表[(我建议搜索你喜欢的 IDE，因为没有全部列出)。为](https://langserver.org/#implementations-client) [Vim](https://github.com/camel-tooling/camel-lsp-client-spacevim/issues/1) 或 [Emacs](https://github.com/camel-tooling/camel-lsp-client-emacs/issues) 提供一些指针。如果在 [Camel 工具组织](https://github.com/camel-tooling)中没有针对您的 IDE 的特定 Git 库，您可以在 [Camel 语言服务器 Git 发布库](https://github.com/camel-tooling/camel-language-server)上提出请求或提供反馈。

*Last updated: July 1, 2020*