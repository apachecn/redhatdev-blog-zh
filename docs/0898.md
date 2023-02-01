# Quarking Drools:我们如何将一个 13 岁的 Java 项目变成一流的无服务器组件

> 原文：<https://developers.redhat.com/blog/2019/03/14/quarking-drools-how-we-turned-a-13-year-old-java-project-into-a-first-class-serverless-component>

计算机能否思考的问题并不比潜水艇能否游泳的问题更有趣

基于规则的人工智能(AI)经常被忽视，可能是因为人们认为它只在重量级的企业软件产品中有用。然而，那不一定是真的。简单地说，规则引擎只是一个软件，它允许您将领域和特定于业务的约束从主应用程序流中分离出来。我们是开发和维护 [Drools](https://www.drools.org/) 团队的一员，Drools 是世界上最流行的开源规则引擎，也是 Red Hat 的一部分，在本文中，我们将描述我们如何改变 Drools，使其成为云和无服务器革命的一部分。

## 技术概述

我们的主要目标是使规则引擎的核心变得更轻、更独立、易于跨不同平台移植，并且非常适合在容器中运行。在过去的 20 年里，软件开发领域发生了很大的变化。我们正越来越多地走向一个多语言的世界，这也是我们努力让我们的技术在许多不同平台上工作的一个原因。这也是我们开始关注新的 Oracle 实验室多语言虚拟机(VM)生态系统 [GraalVM](https://www.graalvm.org/) 的原因，该生态系统包括:

*   一个多语言 VM 运行时，替代了带有实时(JIT)编译器的 [Java](https://developers.redhat.com/topics/enterprise-java/) 虚拟机(JVM ),与传统的 HotSpot 相比，它提高了应用程序的效率和速度。这也是“适当的”GraalVM。
*   一个框架来编写高效的动态编程语言(例如 JavaScript、Python 和 R)并混合和匹配它们(Truffle)。
*   一个提前将程序(AOT)编译成本机可执行文件的工具。

与此同时，在 Red Hat，另一个团队已经在尝试使用 GraalVM 和原生二进制代码进行应用程序开发。这种努力已经在一个你可能听说过的叫做 [Quarkus](https://developers.redhat.com/blog/2019/03/07/quarkus-next-generation-kubernetes-native-java-framework/) 的新项目中实现了。Quarkus 项目是一个同类最佳的 Java 堆栈，可以在优秀的旧 JVM 上工作，但也是专门为 GraalVM、原生二进制编译和云原生应用程序开发而定制的。

GraalVM 是一个了不起的工具，但它也有一些(可以理解的)限制。因此，Quarkus 旨在与 GraalVM 和原生映像生成无缝集成，并提供有用的实用程序来克服任何相关的限制。特别是，Drools 曾经大量使用动态类生成、类加载和相当多的反射。为了生成快速、高效和小型的本机可执行文件，Graal 执行积极的内联和死代码消除，并且它在**封闭世界假设**下运行:也就是说，编译器删除对代码中不能静态到达的类和方法的任何引用。换句话说，无限制的反射调用和动态类加载是不允许的。虽然这乍听起来像是一个精彩的表演，但在这里我们将详细记录我们是如何修改 Drools 的核心来克服这些限制的，我们将解释为什么这些限制不是邪恶的，而是可以解放的。

## 可执行模型

在规则引擎中，**事实**被插入到**工作记忆中。** **规则**描述当插入工作存储器的事实的某些**约束**变为**真**时**采取的动作**。比如“**当** *太阳落山* **:** *开灯*”这句话表达了对太阳的统治。事实是太阳正在下山。*动作*是开灯。在规则引擎中，我们*在工作记忆中插入*太阳正在下山的事实。当我们*触发*规则时，就会执行*开灯*的动作。

规则定义的形式如下

*约束* → *后果*

*约束*部分，也称为规则的*左侧*，描述了激活规则并使其准备好*触发*的约束；*结果*部分，也称为规则的右侧*部分*，包含规则被触发时规则将采取的动作。

在 Drools 中，使用 Drools 规则语言(简而言之，DRL)编写了一个规则，其形式如下:

```
rule R1 when
   $r : Result()                               // constraints
   $p : Person( age >= 18 )     
then
   $r.setValue( $p.getName() + " can drink");  // consequence
end
```

使用模式匹配的形式对插入工作内存的数据(Java 对象)编写约束。动作基本上是一个 Java 代码块，带有一些 Drools 特有的扩展。

历史上，DRL 曾经是一种动态语言，在运行时由 Drools 引擎解释。特别是，模式匹配语法有一个主要缺点:它大量使用了反射，除非引擎检测到一个约束足够“热”以进行进一步优化；也就是说，如果它已经评估了一定的次数；在这种情况下，引擎会动态地将其编译成字节码。

大约一年前，出于性能原因，我们决定放弃运行时反射和动态代码生成，并完成了我们称之为 [Drools 可执行模型](http://blog.athico.com/2018/02/the-drools-executable-model-is-alive.html)的实现，提供了一个规则集的纯基于 Java 的表示，以及一个方便的 Java DSL 来编程定义这样的模型。

为了给出这个 Java API 的样子，让我们再次考虑上面报道的简单 Drools 规则。如果工作记忆包含年龄字段大于或等于 18 的任何结果实例和任何人实例，则该规则将被触发。结果是将结果对象的值设置为一个字符串，表示这个人可以喝酒。用可执行模型 API 表达的等价规则如下所示(为了可读性，进行了漂亮的印刷):

```
var r = declarationOf(Result.class, "$r");
var p = declarationOf(Person.class, "$p");
var rule =
   rule("com.example", "R1").build(
         pattern(r),
         pattern(p).expr("e", p -> p.getAge() >= 18),
         alphaIndexedBy(int.class, GREATER_OR_EQUAL, 1, this::getAge, 18),
         reactOn("age")),
    on(p, r).execute(($p, $r) ->
         $r.setValue($p.getName() + " can drink")));
```

正如您所看到的，这种表示更加冗长和难以理解，部分原因是因为 Java 语法，但主要是因为它显式地包含了许多细节，例如 Drools 应该如何在内部索引给定的约束，这隐含在相应的 DRL 中。我们故意这样做，因为我们想要一个完全显式的规则表示，不需要任何复杂的推理或反射魔法。然而，我们知道让用户知道所有这些复杂的细节是疯狂的，所以我们写了一个编译器来把 DRL 翻译成等价的 Java 代码。我们使用 [JavaParser](http://javaparser.org/) 实现了这一点，这是一个非常好的开源库，允许通过一个方便的 API 解析、修改和生成任何 Java 源代码。

老实说，当我们设计和实现可执行模型时，我们头脑中并没有严格的 GraalVM。我们只是想要一个中间的、纯 Java 的规则表示，它可以被引擎有效地解释和执行。然而，通过完全避免反射和动态代码生成，可执行模型是允许我们用 Graal 支持本机二进制生成的关键。例如，因为新模型将所有约束表示为 lambda 谓词，所以我们不需要通过字节码生成和动态类加载来优化约束赋值器，这在本机映像生成中是完全禁止的。

在使 Drools 兼容 Graal 的过程中，可执行模型的设计和实现给我们上了重要的一课:任何限制都可以通过足够多的代码生成来克服。我们将在下一节进一步讨论这一点。

## 克服其他重大限制

拥有一个 Drools 规则库的普通 Java 模型是一个非常好的起点，但是要使我们的项目与本机二进制生成兼容，还需要做更多的工作。

可执行模型使得反射在很大程度上是不必要的；然而，我们的引擎仍然需要对最后一个叫做[属性反应](http://docs.jboss.org/drools/release/7.17.0.Final/drools-docs/html_single/#_fine_grained_property_change_listeners)的特性进行反思。我们的计划是完全摆脱反射，但是，因为这种改变是不小的，所以这次我们求助于二进制映像编译器的一个方便的特性。这个特性*确实*支持某种形式的反射，前提是我们可以提前声明我们在运行时需要反射的类。这可以通过向编译器提供一个 JSON 描述符文件来实现，或者，如果你使用 Quarkus，你可以用[注释域类](https://quarkus.io/guides/rest-json-guide)。例如，在上面显示的规则中，我们的域类是 Result 和 Person。然后我们可以写:

```
[JSON descriptor
 {
    "name" : "org.drools.simple.project.Person",
    "allPublicMethods" : true
 },
 {
    "name" : "org.drools.simple.project.Result",
    "allPublicMethods" : true
 }
]
```

然后，我们可以用标志来指示本机二进制编译器

```
-H:ReflectionConfigurationFiles=reflection.json
```

我们将其他多余的反射诡计分离到一个动态模块中，并实现了相同组件的另一个静态版本，用户可以选择将其导入到他们的项目中。这种方法对于二进制图像生成特别有用，但是对于常规用例也有好处。特别是，避免反射和动态加载可以导致更快的启动时间和改进的运行时间。

在启动时，Drools 项目读取一个名为 *kmodule* 的 XML 描述符，其中用户声明性地定义了项目的配置。通常，我们解析这个 XML 文件并将其加载到内存中，但是我们当前基于 XStream 的解析器使用了大量的反射；因此，首先，我们可以用避免反射的替代策略来加载 XML。然而，我们可以走得更远:如果我们可以保证 XML 的内存表示在运行中永远不会改变，并且我们可以在重新打包项目进行部署之前运行一个快速的代码生成阶段，那么我们就可以完全避免在每次启动时加载 XML。事实上，我们现在能够将 XML 文件转换成一个在启动时加载的类文件，就像任何其他手工编码的类一样。下面是 XML 与生成的代码片段的比较(同样，为了可读性，打印得很漂亮)。生成的代码更加详细，因为它明确了所有的配置默认值。

| 

```
<kbase name="simpleKB"
       packages="org.drools.simple.project">
  <ksession name="simpleKS" default="true"/>
</kbase>
```

 | 

```
var m = KieServices.get().newKieModuleModel();
var kb = m.newKieBaseModel("simpleKB");
kb.setEventProcessingMode(CLOUD);
kb.addPackage("org.drools.simple.project");
var ks = kb.newKieSessionModel("simpleKS");
ks.setDefault(true);
ks.setType(STATEFUL);
ks.setClockType(ClockTypeOption.get("realtime"));
```

 |

启动时间的另一个问题是动态类路径扫描。Drools 支持采用基于 DRL 的规则之外的其他方式来做出决策，例如*决策表、*决策模型和符号(DMN) 或*使用*预测模型标记语言* ( *PMML* )。这样的扩展被实现为可动态加载的模块，通过在启动时扫描类路径，这些模块被挂接到核心引擎中。尽管这非常灵活，但并不是必需的:即使在这种情况下，我们也可以避免运行时类路径扫描，并通过在构建时生成代码，或者通过向最终用户提供显式 API 来手动挂钩组件，来提供所需组件的静态*连接。我们求助于提供一个带有最小核心的预建静态模块。**

```
private Map<Class<?>, Object> serviceMap = new HashMap<>();
private void wireServices() {
  serviceMap.put(ServiceInterface.class,
                 Class.forName("org.drools.ServiceImpl").newInstance());
  // … more services here
}
```

请注意，尽管我们在这里使用 Class.forName()，但编译器足够聪明，能够识别该常量并用实际的构造函数替换它。当然，可以通过生成一系列的 **if** 语句来进一步简化这个过程。

最后，我们通过去除最后几个可执行前的遗留模型:遗留的 Drools 类加载器，将所有东西联系在一起。这是以下明显晦涩的错误消息背后的罪魁祸首:

```
Error: unsupported features in 2 methods
Detailed message:
Error: com.oracle.graal.pointsto.constraints.UnsupportedFeatureException: Unsupported method java.lang.ClassLoader.defineClass(String, byte[], int, int, ProtectionDomain) is reachable: The declaring class of this element has been substituted, but this element is not present in the substitution class
To diagnose the issue, you can add the option --report-unsupported-elements-at-runtime. The unsupported element is then reported at run time when it is accessed the first time.
Trace:
       at parsing org.drools.dynamic.common.DynamicComponentsSupplier$DefaultByteArrayClassLoader.defineClass(DynamicComponentsSupplier.java:49)
Call path from entry point to org.drools.dynamic.common.DynamicComponentsSupplier$DefaultByteArrayClassLoader.defineClass(String, byte[], ProtectionDomain):
```

然而，实际上，这个信息非常清楚:我们的定制类加载器能够动态地*定义*一个类，这在运行时*生成字节码时非常有用。*但是，如果代码库完全依赖于可执行模型，我们可以完全避免这种情况，所以我们将遗留类加载器隔离到了*动态*模块中。

这是成功生成我们的简单测试项目的本机映像所必需的最后一步，结果超出了我们的预期，从而证实了我们在这个实验中花费的时间和精力是值得的。实际上，用普通 JVM 执行我们测试用例的主类需要 43 毫秒，占用 73M 内存。Graal lasted 生成的相应本机映像的时间不到 1 毫秒，并且仅使用 21M 的内存。

## 与 quartus 集成

一旦我们有了兼容 Graal 原生二进制生成的 Drools 的第一个版本，下一步自然是开始利用 Quarkus 提供的特性，并尝试用它创建一个简单的 web 服务。我们注意到，Quarkus 提供了一种不同的、更简单的机制，让编译器知道我们需要对特定的类进行反射。事实上，不必像以前一样在 JSON 文件中声明，您可以如下注释您的域模型的类:

```
@RegisterForReflection
public class Person { … }
```

我们还决定用我们的代码生成机制向前迈进一小步。特别是，我们为 Drools 代码添加了一个小接口

```
public interface KieRuntimeBuilder {
    KieSession newKieSession();
    KieSession newKieSession(String sessionName);
}
```

因此，当 Drools 编译器从 DRL 文件创建可执行模型时，它也会生成该类的一个实现。这个实现的目的是提供一个 Drools 会话，该会话自动配置了用户定义的规则和参数。

之后，我们准备好将 Quarkus 提供的依赖注入和 REST 支持投入使用，并且开发了一个简单的 web 服务来运行 Drools 运行时。

```
@Path("/candrink/{name}/{age}")
public class CanDrinkResource {

    @Inject
    KieRuntimeBuilder runtimeBuilder;

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String canDrink( @PathParam("name") String name,
                            @PathParam("age") int age ) {

       KieSession ksession = runtimeBuilder.newKieSession();

       Result result = new Result();
       ksession.insert(result);
       ksession.insert(new Person( name, age ));
       ksession.fireAllRules();

       return result.toString();
    }
}
```

这个例子非常简单，不需要任何进一步的解释，并且完全可以部署为 OpenShift 集群中的微服务。由于极低的启动时间——由于我们在 Drools 上所做的工作和 Quarkus 的低开销——这个微服务足够快，可以部署在一个 [KNative](https://cloud.google.com/knative/) 云中。你可以在 [GitHub](https://github.com/kiegroup/submarine-examples) 上找到完整的源代码。

## 介绍潜艇

如今，规则引擎很少成为讨论的话题。这是因为他们*只是工作*。规则引擎不一定与云环境对立，但是可能需要工作来适应新的范式。这就是我们旅行的故事。我们从勇气和好奇心开始。在接下来的几个月里，我们将推进这项工作，使其不仅仅是一个简单的原型，而是实现一整套为云做好准备的业务自动化工具。倡议的名字是**号潜艇，**是从著名的 Dijkstra 引用来的。所以，坐稳了，准备好开始吧。

*Last updated: January 4, 2022*