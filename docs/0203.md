# cloud events JavaScript SDK 简介

> 原文：<https://developers.redhat.com/blog/2021/03/09/an-introduction-to-javascript-sdk-for-cloudevents>

在今天这个[无服务器](https://developers.redhat.com/topics/serverless-architecture)功能和[微服务](https://developers.redhat.com/topics/microservices)的世界里，事件无处不在。问题是，根据您使用的生产者技术，对它们的描述是不同的。

如果没有一个通用的标准，开发人员就要不断地重新学习如何使用事件。没有一个标准也使得库和工具的作者更难在像 SDK 这样的环境中交付事件数据。最近，创建了一个新项目来帮助这项工作。

[CloudEvents](https://cloudevents.io/) 是一种以通用格式描述事件数据的规范，以提供跨服务、平台和系统的互操作性。事实上，红帽 OpenShift 无服务器功能使用的是 CloudEvents。关于这个新的开发者特性的更多信息，请看 [*用 Red Hat OpenShift 无服务器函数*](/blog/2021/01/04/create-your-first-serverless-function-with-red-hat-openshift-serverless-functions/) 创建你的第一个无服务器函数。

## 云事件规范

该规范的目标不是创建另一种事件格式，并试图强迫每个人都使用它。相反，我们希望为事件定义公共元数据，并确定这些元数据应该出现在被发送的消息中的什么位置。

这是一个带有简单目标的简单规范。事实上，一个 CloudEvent 只需要四块元数据:

*   `type`描述这可能是什么类型的事件(例如，“创建”事件)。
*   `specversion`表示用于创建云事件的规范版本。
*   `source`描述事件的来源。
*   `id`是对德杜平有用的唯一标识符。

还有其他有用的字段，如`subject`，当与`source`结合使用时，可以为事件的起源添加更多的上下文。

正如我提到的，CloudEvents 规范只关心上面列出的公共元数据，以及发送事件时放置这些元数据的位置。

目前，有两种事件格式:二进制(首选格式)和结构化。建议使用二进制，因为它是可加的。也就是说，二进制格式只是在 HTTP 请求中添加了一些头。如果有一个中间件不理解 CloudEvents，它不会破坏任何东西，但如果那个系统更新到支持 CloudEvents，它就开始工作了。

结构化格式是为那些目前还没有定义任何格式，并且正在寻找应该如何构建的指导的人准备的。

下面是这两种事件格式在原始 HTTP 中的简单示例:

```
// Binary

Post  /event HTTP/1.0
Host: example.com
Content-Type: application/json
ce-specversion: 1.0
ce-type: com.nodeshift.create
ce-source: nodeshift.dev
ce-id: 123456
{
  "action": "createThing",
  "item": "2187"
}

// Structured

Post  /event HTTP/1.0
Host: example.com
Content-Type: application/cloudevents+json

{
  "specversion": "1.0"
  "type": "com.nodeshift.create"
  "source": "nodeshift.dev"
  "id": "123456"
  "data": {
    "action": "createThing",
    "item": "2187"
  }
}

```

## 面向云事件的 JavaScript SDK

当然，我们不希望必须手动格式化这些事件。这就是用于 CloudEvents 的 [JavaScript SDK 的用武之地。SDK 应该实现三个主要目标:](https://www.npmjs.com/package/cloudevents)

*   撰写事件。
*   对要发送的事件进行编码。
*   解码输入事件。

安装 JavaScript SDK 就像使用任何其他节点模块一样:

```
$ npm install cloudevents

```

既然我们已经了解了什么是 CloudEvent 以及它是如何有用的，那么让我们来看一个例子。

## 创建新的云事件

首先，我们将创建一个新的 CloudEvent 对象:

```
const { CloudEvent } = require('cloudevents');

// Create a new CloudEvent
const ce = new CloudEvent({
 type: 'com.cloudevent.fun',
 source: 'fun-with-cloud-events',
 data: { key: 'DATA' }
});

```

如果我们用对象的内置`toJSON`方法将其注销，我们可能会看到类似这样的内容:

```
console.log(ce.toJSON());

{
 id: '...',
 type: 'com.cloudevent.fun',
 source: 'fun-with-cloud-events',
 specversion: '1.0',
 time: '...',
 data: { key: 'DATA' }
}

```

### 发送消息

接下来，让我们看看如何使用二进制格式通过 HTTP 发送这个消息。

首先，我们需要创建二进制格式的消息，使用`HTTP.binary`方法可以很容易地做到这一点。我们将使用上一个示例中的 CloudEvent:

```
  const message = HTTP.binary(ce);
  //const message = HTTP.structured(ce); // Showing just for completeness

```

同样，如果我们注销它，它可能看起来像这样:

```
 headers: {
   'content-type': 'application/json;',
   'ce-id': '...',
   'ce-type': 'com.cloudevent.fun',
   'ce-source': 'fun-with-cloud-events',
   'ce-specversion': '1.0',
   'ce-time': '...'
 },
 body: { key: 'DATA' }
}

```

既然消息已经被正确格式化，我们可以通过使用像 [Axios](https://github.com/axios/axios) 这样的库来发送它。

注意，CloudEvents SDK 不处理发送消息；它只处理消息头和消息体的格式化。这允许您使用任何想要的 HTTP 库来发送消息。

```
const axios = require('axios')

axios({
 method: 'post',
 url: 'http://localhost:3000/cloudeventy',
 data: message.body,
 headers: message.headers
}).then((response) => {
 console.log(response.data);
});

```

我们正在向“cloud event-y”REST 端点发送一个 POST 请求。在这个例子中，我使用了一个简单的 Express.js 应用程序，但是您可以使用任何您喜欢的框架。

### 接收消息

一旦我们有了消息，我们可以使用`HTTP.toEvent`方法将其转换回 CloudEvent 对象。

```
const express = require('express');
const { HTTP } = require('cloudevents');

const app = express();

app.post('/cloudeventy', (req, res) => {
  const ce = HTTP.toEvent({
                  headers: req.headers, 
                  body: req.body
  });
 console.log(ce.toJSON());
 res.send({key: 'Event Received'});
});

```

同样，日志输出类似于我们输出 CloudEvent 对象时看到的内容:

```
{
 id: '...',
 type: 'com.cloudevent.fun',
 source: 'fun-with-cloud-events',
 specversion: '1.0',
 time: '...',
 data: { key: 'DATA' }
}

```

## 结论

要了解更多关于 JavaScript SDK for CloudEvents 的信息，[请查看 GitHub 项目](https://github.com/cloudevents/sdk-javascript)。有关该规范背后的历史、发展和设计原理的更多信息，请参见 [CloudEvents 初级读本](https://github.com/cloudevents/spec/blob/master/primer.md)。

*Last updated: March 8, 2021*