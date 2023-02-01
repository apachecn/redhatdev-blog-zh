# 放心测试 REST APIs

> 原文：<https://developers.redhat.com/blog/2017/07/20/testing-rest-apis-with-rest-assured>

**注:**这是几年前我为我的私人博客写的一篇文章[的更新版本。](http://pilhuhn.blogspot.de/2013/01/testing-rest-apis-with-rest-assured.html)

很久以前，当我在研究 RHQ 的 REST API 时，我已经开始针对它编写一些集成测试。通过纯粹的 HTTP 调用来实现这一点是非常繁琐和脆弱的。因此，我在寻找一个测试框架来帮助我，并且找到了一个我用了一段时间的框架。我试图增强它，以更好地满足我的需求，但并没有真正让它工作。

我再次开始搜索，这次找到了[放心](http://rest-assured.io)，它几乎是完美的，因为它提供了一个高级的流畅 Java API 来编写测试。可以放心地与 JUnit 或 TestNG 等经典测试程序一起使用。

让我们看一个非常简单的例子:

```
@Test
public void testStatusPage()
{
  expect()
     .statusCode(200)
     .log().ifError()
  .when()
     .get("/status/server");
}
```

正如你所看到的，这个流畅的 API 非常有表现力，所以我真的不需要解释上面是要做什么。该示例还显示了放心使用默认的目标服务器 IP 和端口。

在下一个例子中，我将添加一些身份验证。

```
given()
     .auth().basic("user","name23")
  .expect()
     .statusCode(401)
  .when()
     .get("/rest/status");
```

在这里，我们将身份验证“参数”添加到调用中，在我们的例子中，这些参数是基本身份验证的信息，我们希望调用失败时会出现“错误的身份验证”响应，以确保拥有错误凭证的用户无法登录。您可以看到，我们不需要知道身份验证实际上是如何转换成 HTTP 报头的，而是可以专注于逻辑。

由于在所有测试中始终提供身份验证位是很繁琐的，因此可以在测试设置中告诉放心，始终提供一组默认的凭证，这些凭证仍然可以被覆盖，如下所示:

```
@Before
public void setUp() {
   RestAssured.authentication = basic("rhqadmin","rhqadmin");
}
```

还有很多选项可以设置为默认值，比如基本 URI、端口、基本路径等等。

现在让我们看看如何提供其他参数并从 POST 请求中检索结果。

```
AlertDefinition alertDefinition = new AlertDefinition(….);
  Header acceptJson = new Header("Accept", "application/json")

  AlertDefinition result =
  given()
     .contentType(ContentType.JSON)
     .header(acceptJson)
     .body(alertDefinition)
     .log().everything()
     .queryParam("resourceId", 10001)
  .expect()
     .statusCode(201)
     .log().ifError()
  .when()
     .post("/alert/definitions")
  .as(AlertDefinition.class);
```

我们首先创建一个 Java 对象`AlertDefinition`,用于 POST 请求的主体。我们定义它应该通过传递一个 ContentType 作为 JSON 发送，并且我们期望 JSON 通过我们之前定义的`acceptJson`返回。对于 URL，应该附加一个名为“resourceId”且值为“10001”的查询参数。

我们还期望调用返回一个 201 - created 并想知道详细情况，但事实并非如此。最后但同样重要的是，我们告诉放心，它应该将答案转换回一个类型为`AlertDefinition`的对象，然后我们可以用它来检查约束或进一步处理它。

### **JSON 路径**

放心提供了另一种有趣的内置方法来借助 XmlPath 或它的 JSON 对等 JSON 路径检查约束，这允许像 XPath 对 XML 那样对 JSON 数据进行公式化查询。

假设我们的 REST 调用返回以下 JSON 有效负载:

```
{
  "vendor": [
    {
      "name": "msc-loaded-modules",
      "displayName": "msc-loaded-modules",
      "description": "Number of loaded modules",
      "type": "gauge",
      "unit": "none",
      "tags": "tier=\"integration\"",
      "multi": false
    },
    {
      "name": "BufferPool_used_memory_mapped",
      "displayName": "BufferPool_used_memory_mapped",
      "description": "The memory used by the pool: mapped",
      "type": "gauge",
      "unit": "byte",
      "tags": "tier=\"integration\"",
      "multi": false
    },
    {
      "name": "BufferPool_used_memory_direct",
      "displayName": "BufferPool_used_memory_direct",
      "description": "The memory used by the pool: direct",
      "type": "gauge",
      "unit": "byte",
      "tags": "tier=\"integration\"",
      "multi": false
    }
  ]
}
```

然后可以运行下面的查询(前面有一个'> ')并获得它下面的结果。

```
# Get the names of all the objects below vendor
> vendor.name
[msc-loaded-modules, BufferPool_used_memory_mapped, BufferPool_used_memory_direct]
```

```
# Find the name of all objects below 'vendor' that have a unit of 'none'
> vendor.findAll { vendor -> vendor.unit == 'none' }.name
[msc-loaded-modules]
```

```
# Find objects below vendor that have a name of 'msc-loaded-modules' 
vendor.find { it.name = 'msc-loaded-modules' }
{unit=none, displayName=msc-loaded-modules, name=msc-loaded-modules, description=Number of loaded modules, type=gauge, tags=tier="integration", multi=false}
```

在前面的例子中，有一个神秘的`it`，它代表了`find`关键字之前的表达式。

然后，您可以在代码中使用这些查询来执行如下操作，以检查返回的数据是否确实是预期的数据。

```
// Do an OPTIONS call to the remote and then translate into JsonPath
JsonPath jsonPath =
    given()
      .header("Accept",APPLICATION_JSON)
    .options("http://localhost:8080/metrics/vendor")
      .extract().body().jsonPath();

// Now find the buffer direct pool
Map<String,String> aMap = jsonPath.getMap("find {it.name == 'BufferPool_used_memory_direct'}");
// And make sure its display name is correct
assert directPool.get("displayName").equals("BufferPool_used_memory_direct");
```

注意，JSONPath 表达式遵循 [Groovy GPath 语法](http://groovy-lang.org/processing-xml.html#_gpath)。我已经创建了一个小的 [JsonPath explorer](https://github.com/pilhuhn/jsonPathExplorer) ，它可以用来针对给定的 JSON 输入文档测试查询。

### **匹配者**

放心的另一个优点是可以使用所谓的匹配器，

```
when()
   .get("http://localhost:8080/metrics/base")
.then()
   .header("ETag",notNullValue())
   .body("scheduleId", hasItem( numericScheduleId)) 
   .body(containsString("total-started-thread-count"));
```

这里的`containsString()`、`notNullValue()`和`hasItem()`方法就是这样一种匹配器，它们在调用 REST API 时获取的所有消息体中寻找传递的表达式。再次使用匹配使得测试代码非常有表现力，并确保良好的可读性。

### **结论**

放心是一个非常强大的框架，可以根据 REST/超媒体 API 编写测试。凭借其流畅的方法和富于表现力的方法名称，它可以很容易地理解某个调用应该做什么和返回什么。JSONPath 和 Matchers 都进一步增加了功能和表现力。

### **延伸阅读**

*   放心吧，网站上有更多的例子和 T2 文档。
*   提到的 RHQ 项目有大量的[测试](https://github.com/rhq-project/rhq/tree/master/modules/integration-tests/rest-api/src/test/java/org/rhq/modules/integrationTests/restApi)可以作为例子。
*   这里通过 Groovy GPath 很好地展示了 [JSONPath 的功能。](http://james-willett.com/2017/05/rest-assured-gpath-json/)

* * *

要构建您的 Java EE 微服务 **请访问 WildFly Swarm 并下载备忘单。**

*Last updated: October 14, 2021*