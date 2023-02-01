# 使用 GitHub API 开始使用 GraphQL

> 原文：<https://developers.redhat.com/blog/2017/11/13/getting-started-graphql-using-github-api>

GraphQL 是一种用于 API 的查询语言。它为 API 中的数据提供了一个完整的、可理解的描述，让客户能够准确地要求他们所需要的，仅此而已。

向您的 API 发送一个 GraphQL 查询，并获得您所需要的内容。这些查询总是使用 GraphQL 返回可预测的结果。它又快又稳。

打开 GitHub 提供的 GraphQL explorer 后，您会注意到它会在左侧面板中显示一个示例查询，如下所示:

```
# We'll get a sample query showing your username!
query {
  viewer {
    login
  }
}

```

通过使用 GraphQL，我们可以创建有用的查询，获取查询的响应，还可以创建变异。

GraphQL 可以用来检查数据库 id。例如:

```
{
    user(login: "yourUsername") {
        databaseId
    }
}
```

将会生成如下响应:

```
{
    "data": {
        "user": {
            "databaseId": 12345678
        } 
    }
}
```

接下来，我将结合 GitHub 解释使用 GraphQL 的突变示例。假设有人必须对某个问题做出反应。我们将通过使用 GraphQL 来实现这一点。

```
{
 repository(owner: "yourName", name: "repository-name") {
   issues(last: 5) {
     edges {
       node {
         title
         id
       }
     }
   }
 }
}
```

它将生成如下响应:

```
{
 "data": {
   "repository": {
     "issues": {
       "edges": [
         {
           "node": {
             "title": "Title of the 1st issue",
             "id": "MDU6SRXNXzdWUyMTg2NTcwMzk="
           }
         },
         {
           "node": {
             "title": "Another issue",
             "id": "MDU6SRXXNzdWUyMTg2NjA4OTQ="
           }
         }
       ]
     }
   }
 }
}
```

它会自动为每个不同类型的问题生成唯一的 id。现在，我们将使用突变的概念，通过使用其唯一的 ID 来添加对任何一个问题的反应。

```
mutation {
 addReaction(input: {subjectId: "MDU6SRXXNzdWUyMTg2NjA4OTQ=", content: LAUGH}) {
   reaction {
     content
     id
   }
 }
}
```

因此，这将增加一个“笑”的问题。也可以通过检查我们得到的响应来观察:

```
{
 "data": {
   "addReaction": {
     "reaction": {
       "content": "LAUGH",
       "id": "MDg6UmVhY3Rpb24xNTQyNTUxMQ=="
     }
   }
 }
}
```

* * *

[**加入红帽开发者计划**](https://developers.redhat.com/?intcmp=70160000000xZNgAAM) **(免费)并获得相关的备忘单、书籍和产品下载。**

*Last updated: November 9, 2017*