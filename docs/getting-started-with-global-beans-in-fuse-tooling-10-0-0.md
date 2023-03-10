# Fuse 工具 10.0.0 中的全局 Beans 入门

> 原文：<https://developers.redhat.com/blog/2017/08/23/getting-started-with-global-beans-in-fuse-tooling-10-0-0>

[Red Hat JBoss Fuse](http://developers.redhat.com/products/fuse/overview/) 提供了一个开源、轻量级、模块化的平台，使您能够连接应用环境中的各种服务和系统。并且，[Red Hat JBoss Developer Studio](http://developers.redhat.com/products/devstudio/overview/)中包含的 Fuse 工具可以帮助您利用这个平台。

路由编辑器最初关注于路由或 Camel 上下文元素中的 Camel 配置部分，但在 8.0.0 版中，我们开始在 Configurations 选项卡上添加对全局元素(如数据格式和端点)的支持。在 10.0 版本中，我们增加了对路由之外的 beans 的支持。

本教程展示了如何创建一个使用全局 Java bean 和 Camel XML 的可定制服务。最后，您将拥有一个简单的 Fuse 集成项目，其中包括一个验证 bean 逻辑的 Camel 测试。

本教程中的主要步骤是:

1.  创建一个 Fuse 集成项目。
2.  在项目中创建一个 bean。
3.  完成 bean 类。
4.  创建一条路线。
5.  创建一个测试。
6.  完成生成的测试。

先决条件:需要月食氧气。在安装 Red Hat Developer Studio 11.0 的过程中，您应该已经选择了 JBoss Fuse 工具的安装。

## 步骤 1:创建一个项目

首先，我们必须创建一个放置代码的项目。

1.  在 Developer Studio 菜单栏，点击**文件- >新建- > Fuse 集成项目**。
2.  在新建项目向导中，在**项目名称**字段中，输入`global-bean-tutorial`。
3.  接受所有默认设置，创建一个新的空的 Fuse 项目。
4.  点击**完成**。

Fuse tooling 然后将构建一个新的 Blueprint Camel 项目，这可能需要一些时间。

## 步骤 2:创建我们的全局 Bean

![](img/3034655dec0822efc58310cbcd43c82c.png)创建一个简单的 Java bean 用于 Camel route:

1.  在路线编辑器中，点击**配置**选项卡，然后点击**添加**。
2.  在“创建新的全局元素...”中对话框中，点击 **Bean** 然后点击 **OK** 。
3.  在“添加 Bean”对话框的 **Id** 字段中，输入`GreetingBean`。
4.  在**类**字段的右边，点击 **+** 按钮创建一个新类。
5.  在“新建 Java 类”向导中:

    1.  在**包**字段中，输入`org.example`。
    2.  在**名称**字段中，输入`Greeting`。
    3.  点击**完成**。
6.  在“添加 Bean”对话框中，点击**完成**。

## 步骤 3:完成 Bean 类

上一步为 Java bean 创建了一个粗略的轮廓。要完成 bean 类并让它做一些事情，请将代码改为如下所示:

```
package org.example;

import java.util.ArrayList;
import java.util.List;

public class Greeting {

   private List messageCache = new ArrayList<>();
   private String greetingText = "Hello";

   public Greeting() {
      // empty
   }

   public Greeting(String newGreeting) {
      greetingText = newGreeting;
   }

   public String hello(String msg) {
      String helloMsg = greetingText + " " + msg;
      messageCache.add(helloMsg);
      return helloMsg;
   }

   public String toString() {
      return messageCache.toString();
   }
}
```

这个简单的 bean 有一个方法，`hello(String)`, that``；

1.  将问候语字符串附加到方法参数的开头。
2.  缓存修改后的字符串。
3.  返回修改后的字符串。

`hello(String)`方法提供了一种在构造函数中定制问候语和检索缓存的方法。Camel route 可以使用这些不同的 bean 代码入口点。

## 步骤 4:创建路线

![](img/4e2ff149e10b028547a859479adc8fb3.png)创建将使用 bean 的路由:

1.  在路线编辑器中，点击**设计**选项卡。
2.  在**面板**中，在**组件**下，点击**直接**到 **Route_route1** 容器上。本教程使用直接组件来保持示例的简单性。当然，您可以使用文件组件、JMS 组件或其他组件。
3.  选择航路中新的“直接”组件:

    1.  打开**属性**视图。
    2.  点击**详细信息**选项卡。
    3.  在 **Uri** 字段中，将值更改为`direct:beanTutorial`。
4.  在**组件面板**中，在【T2 组件】下，点击 **Bean** 组件并拖拽到直接组件上。
5.  选择 Bean_bean1 组件，在**属性**视图中:

    1.  选择**详细信息**选项卡。
    2.  在 **Ref** 字段中，显示下拉列表。
    3.  选择*欢迎 Bean* ，它之所以出现在这里，是因为您之前已经将其添加为全局 Bean。这将创建对全局 bean 的引用，并将其插入到路由中。
6.  在**面板**中，将另一个**直接**组件拖到路线中的 **Bean** 上。
7.  选择路线中的第二个“直接”组件，在**属性**视图中:
    *   在 **Uri** 字段中，将值更改为 *direct:beanTutorialOut* 。这将创建路线的最终目的地。
8.  保存路线。

## 步骤 5:创建一个测试

![](img/578163f70da25b5f15d7a7a86d8fc398.png)在您在步骤 1 中创建的项目中，创建一个测试:

1.  在**项目浏览器**视图中，在 **src** 文件夹中，创建一个名为`test`的新文件夹。
2.  在**测试**文件夹中，创建一个名为`java`的新文件夹。
3.  在 Developer Studio 菜单栏，选择**文件- >新建- > Camel 测试用例**。
4.  在**新 Camel JUnit 测试用例向导** :

    1.  确保**源文件夹**路径为`global-bean-tutorial/src/test/java`。
    2.  在**包**字段，输入`org.example`。
    3.  在 **Camel XML 文件测试**字段的右边，点击**浏览**。
    4.  选择 *blueprint.xml* 文件，点击**确定**。
    5.  在**名称**字段中，输入`GreetingTest`。
    6.  点击**完成**。

## 步骤 6:完成生成的测试

Camel 测试用例向导生成一个测试类，它扩展了`CamelBlueprintTestSupport`并提供了一些模板化的代码。为了完成生成的测试，提供一些样本输入和输出消息，然后将它们发送到项目的 Camel route。

更改您的类，如下所示:

```
package org.example;

import org.apache.camel.EndpointInject;
import org.apache.camel.Produce;
import org.apache.camel.ProducerTemplate;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.component.mock.MockEndpoint;
import org.apache.camel.test.blueprint.CamelBlueprintTestSupport;
import org.junit.Test;

public class GreetingTest extends CamelBlueprintTestSupport {

  // Input message bodies to test with
  protected Object[] inputBodies = 
    { "This is one bean example.",
      "This is another bean example." };

  // Expected message bodies
  protected Object[] expectedBodies = 
    { "Hello This is one bean example.",
      "Hello This is another bean example."};

  // Templates to send to input endpoints
  @Produce(uri = "direct:beanTutorial")
  protected ProducerTemplate inputEndpoint;

  // Mock endpoints used to consume messages from the output 
  // endpoints and then perform assertions
  @EndpointInject(uri = "mock:output")
  protected MockEndpoint outputEndpoint;

  @Test
  public void testCamelRoute() throws Exception {
     // Create routes from the output endpoints to our mock 
     // endpoints so we can assert expectations
     context.addRoutes(new RouteBuilder() {
          @Override
          public void configure() throws Exception {
               from("direct:beanTutorialOut").to(outputEndpoint);
          }
     });

     // Define some expectations
     outputEndpoint.expectedBodiesReceivedInAnyOrder(expectedBodies);

     // Send some messages to input endpoints
     for (Object inputBody : inputBodies) {
          inputEndpoint.sendBody(inputBody);
     }

     // Validate our expectations
     assertMockEndpointsSatisfied();
  }

  @Override
  protected String getBlueprintDescriptor() {
     return "OSGI-INF/blueprint/blueprint.xml";
  }
}
```

有了这些修改，在**项目浏览器**视图中，右键单击`GreetingTest`类并选择**运行方式- > JUnit 测试**。你的测试应该会通过。

![](img/cdace38e6bc3402fe78aa2d7e0c1292f.png)

本系列的下一篇路由编辑器 bean 教程文章将展示如何:

*   更新路由，更具体地说明要调用哪个 bean 方法。
*   通过传递构造函数参数来自定义问候语。
*   根据路由和 bean 更新来更新测试。

* * *

**点击此处查看 [Red Hat JBoss Fuse](https://developers.redhat.com/products/fuse/overview/?intcmp=7016000000124gGAAQ) 的概述，这是一个轻量级的模块化集成平台。**

*Last updated: August 18, 2017*