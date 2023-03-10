# 扼杀你的 API:汉堡类比

> 原文：<https://developers.redhat.com/blog/2017/07/28/kill-your-api-the-burger-analogy>

前几天，我和致力于 OpenAPI 计划的人聊天，我解释了一个我们企业中的一些人用来从我们的 API 中获得更高速度和可伸缩性的技巧。起初，对于那些坚持标准的人来说，这似乎是完全的亵渎，但是如果你允许我用一个简单的类比来解释，你会发现这是多么的有用…

**“汉堡类比”**

比方说，你的电脑必须向一家餐馆下订单。您需要以该项目的格式向餐馆发送一个请求。因此，如果在这种情况下，我想要一个“吉士汉堡”，我会发送 **“内容类型:食物/吉士汉堡”** 格式，这样他们就知道我的订单的 ID 是什么，餐厅会发回[面包、生菜、西红柿、肉、酱、面包]。

既然他们知道了我的请求，你认为我还需要发送额外的数据告诉他们返回格式也是' **食物/芝士汉堡** '？当然不是！那没有意义！当然，我期待的是‘食物/芝士汉堡’……这就是我点的菜！实际上，如果他们退回‘食物/玉米卷’，我会很生气(好吧，没那么生气…但你会明白的)。

**拿你点的东西**

这就是我对 OpenAPI 倡议的解释。在 API 中，我们发送两个头:

*   **CONTENT-TYPE:** 一个标题，说明我们请求的是什么数据。
*   **ACCEPT:** 一个报头，说明我们用什么数据来响应。

但是超过 **90%的 API 总是返回请求的内容！** 所以，这是多此一举。为什么我们要做所有这些额外的处理？这是因为我们试图覆盖所有用例的 100%,而不是仅仅自动化 90%,然后在出现异常时编码……这是我们应该做的。

因此，在这些规定格式的头中，只需要一个:内容类型。

但是 GET、OPTIONS 或 DELETE 呢？

你可以捏造和使用内容类型。对所有内容使用一个头使得自动化更容易，编码更容易，处理更容易，请求更容易。这简化了我们的 AJAX 和 API 调用。不，创造一打不同的方式来称呼事物。 **只是单向调用一切** 和单向处理 **。**

所以我们创建一个 AJAX 调用，就像这样

 `$.ajax({  type: POST,  url: '`+window.url+`/v0.1/`+path+`' ,  data: JSON.stringify(jsonData),  headers:{  'Content-Type': 'application/json'  },  success: function(json) {  ...  },  error: function(jqXHR, textStatus, errorThrown) {  ...  },  ));` 

**框架中的用法**

这已经在几个框架中使用，以获得速度优势。一个可以立即找到它的框架是[Grails](https://web.archive.org/web/20170929165200/https:/docs.grails.org/latest/ref/Controllers/withFormat.html):
*Grails 3.0 忽略 HTTP Accept 头，除非您将 Grails . mime . use . Accept . header = true 设置添加到 application.groovy 文件中。换句话说，如果没有该设置，withFormat()将完全不受 Accept 头的影响。*

*在我自己在论坛* *上提出完全不必使用 ACCEPT 头的建议后，这个变化被正式添加到 Grails 3.0 中。*

**返回不同的格式**

如果我需要发送 JSON 来获取一个资源，并将该资源处理成 PDF 格式，该怎么办？简单…从控制器/方法发送接受头。这样就不需要处理任何东西，但是您的客户端可以接收不同的内容类型，您的文档可以反映它。

在下面的例子中，我展示了一个 Spring-Boot 控制器方法，其中我返回了 ACCEPT 头:

@RequestMapping(方法=RequestMethod。GET，"/greeting")

公共问候语问候语(@RequestParam(value="name "，defaultValue="World ")字符串名称){

response.addHeader("Accept "，" application/pdf ")；

...

}

**简单……打破常规**

我知道我会得到 50%的反应；当我展示那些大大提高了速度和规模，却违背常规的东西时，我通常会这样做。但是对于那些总是想从环境中获得最大收益的人来说，这是我几年来一直使用的技巧之一。？

*Last updated: January 8, 2018*