# ASP.NET 核心 Hello World 解释道

> 原文：<https://developers.redhat.com/blog/2017/10/20/asp-net-core-hello-world-explained>

大多数教授 C#的书籍都是从“Hello World”应用程序开始的。这个简单的程序用来解释像*名称空间*、*类*、*主*和*控制台这样的概念。WriteLine* 。当每一行代码都被剖析后，它是如何工作的就很清楚了。

对于 ASP.NET 核心应用程序来说，这就不那么明显了。我们不再调用我们的代码；相反，ASP.NET 核心*框架*正在为我们做这件事。在这篇博文中，我们将看看一个简单的 ASP.NET 核心应用程序，并解释 ASP.NET 核心是如何让它运行的。

这是代码:

```
class Program {
    public static void Main(string[] args) =>
        BuildWebHost(args).Run();

    public static IWebHost BuildWebHost(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup()
            .Build();
}

class Startup {
    private IConfiguration _configuration;

    public Startup(IConfiguration configuration) =>
        _configuration = configuration;

    public void ConfigureServices(IServiceCollection services) {
        services.AddMvc();
        services.AddDbContext(
            options => options.UseSqlServer(_configuration.GetConnectionString("Default")));
    }

    public void Configure(IApplicationBuilder app) {
        app.UseMvc();
        app.Run(context => context.Response.WriteAsync("Hello World!"));
    }
}

public class ToDoItem {
    public long Id { get; set; }
    public string Description { get; set; }
    public bool IsDone { get; set; }
}

public class ToDoDbContext : DbContext {
    public ToDoDbContext(DbContextOptions options) :
        base(options)
    {}

    public DbSet ToDoItems { get; set; }
}

[Route("/api/todo")]
public class ToDoController {
    private readonly ToDoDbContext _dbContext;

    public ToDoController(ToDoDbContext context) =>
        _dbContext = context;

    [HttpGet]
    public IEnumerable GetAll() =>
        _dbContext.ToDoItems.ToList(); 
}
```

该应用程序从 URL '/api/todo '处的 SQL server 数据库中返回一个列表，其中列出了 *ToDoItems* 。在其他网址，“你好，世界！”被退回。

你可能在做你的第一个 ASP.NET 核心教程时认出了你复制的模式。

在*程序中。主，*我们把控制权交给 ASP.NET 核心。我们可以看到 *Main* 如何通过 *BuildWebHost* 找到 *Startup* 类。然而，似乎没有一个代码路径通向 *ToDoController* 。ASP.NET 核心通过使用*反射*找到这个类。反射是在运行时在加载的程序集中查找类型信息的能力。

通常，属性用于标识某些类型或方法。在这种情况下，我们的 *ToDoController* 是通过一个*命名约定*找到的:以“Controller”结尾的公共类被认为是 MVC 控制器。类似的，*启动*的*配置*和*配置服务*也是通过它们的名字找到的。 *Route* 和 *HttpGet* 属性用于确定 *ToDoController 处理的 URL 和 HTTP 方法。GetAll* 。这种反映是由 ASP.NET 核心的 MVC 中间件完成的，这是构建 web APIs 和应用程序的默认方法。我们已经通过 *AddMvc* 和 *UseMvc* 调用在我们的应用程序中启用了这个中间件。

*配置*和*配置服务*方法服务于两个重要的、不同的目标。 *ConfigureServices* 用于配置依赖注入(DI)。而*配置*用于设置请求处理流水线。让我们更深入地了解一下这意味着什么。

*依赖注入*是一种模式，支持高级组件和它们完成工作所需的位之间的松散耦合。例如，我们的 *ToDoController* 需要待办事项的数据源。我们没有在控制器内部建立到数据库的连接，而是将其作为构造函数的参数。因此，这种重要的依赖关系不再隐藏在实现中，而是出现在公共 API 中。它还允许在同一控制器上轻松使用其他数据源。这对测试特别有用。例如，我们可以使用内存中的对象存储，而不是使用真正的数据库。

ASP.NET 核心中的依赖注入是基于类型系统的。在 DI 中，*容器*是负责提供依赖关系的类的名称。当被要求提供某种类型的依赖时，容器会查看该类型的公共构造函数。容器将试图找到一个它知道如何创建参数类型的构造函数。例如，当我们向容器请求一种类型的 *ToDoController* 时，基于公共构造函数，它将首先试图弄清楚如何创建一个 *ToDoDbContext* 实例。这就是*启动的地方。ConfigureServices* 开始发挥作用。在这个方法中，我们用 DI 容器注册了额外的类型。 *IServiceCollection* 表示容器已知的类型。通过调用*服务。AddDbContext* 容器了解构建 *ToDoController* 所需的 *DbContextOptions* 和 *ToDoDbContext* 类型。

类型可以用 3 个作用域注册:单例、作用域或瞬态。Singleton 意味着在应用程序的生命周期中使用同一个实例。限定范围意味着每个 HTTP 请求使用同一个实例，瞬态意味着每次创建一个新实例。因此，当两个构造函数参数类型依赖于同一个类型时，它们将在限定范围时获得同一个实例，在瞬态时获得唯一的实例。 *AddDbContext* 用作用域生存期注册 *DbContextOptions* 和 *ToDoDbContext* 。当一个类型被注册时，可以告诉容器它应该创建一个更特殊类型的实例。例如，我们可以注册对 *ISomeFeature* 的请求，应该实例化 *ActualFeature* 。这使我们能够通过 DI 容器改变我们的依赖类所使用的实现(例如，使用 *ActualFeatureV2* )，而不必改变那些类(它们依赖于 *ISomeFeature* )。 *IServiceCollection* 为注册类型提供了一个简单的接口。在 *ConfigureServices* 中调用的大多数方法(比如 *AddDbContext* )都是注册了许多类型的扩展方法。当我们查看名称“ConfigureServices”时，“Configure”反映了我们正在进行设置和配置，“Services”指的是我们正在向 DI 注册的类型。

注意 *Startup* 类正在使用构造函数注入来获取应用程序配置。作为 *WebHost 的一部分，*图标配置*被添加到 DI 容器中。CreateDefaultBuilder* 。默认构建器将从 *appsettings.json* 、 *appsettings 中填充配置。{环境}。json* 和环境变量。我们使用配置来检索 SQL server 连接字符串，这意味着它可以通过环境变量“ConnectionStrings__Default”进行设置，或者作为一个 appsettings json 文件的“ConnectionStrings”对象中的 json 字符串属性“Default”进行设置。请注意，设置查找不区分大小写，“CONNECTIONSTRINGS__DEFAULT”也适用。

现在我们来看看*启动。配置*方法。这个方法配置我们的应用程序如何处理请求。请求处理被设置为一系列处理程序(*管道*)。在这个方法中，我们添加东西的顺序很重要。如果我们反转*应用程序。运行*和 *app。在 *ConfigureServices* 中使用 Mvc* ，每个请求都会得到“Hello World！”“/api/todo”处理程序变得不可访问。每个处理程序控制调用它后面的处理程序(*下一个*)。这意味着处理程序可以自己决定是否要首先调用自己的逻辑，然后调用下一个的*；或者它可以首先调用 *next* ，这样做通常是为了包装其行为(例如，捕捉异常并返回错误页面)。 *UseMvc* 试图找到一个合适的处理程序，如果找不到，它将请求委托给下一个*的*。 *app。跑*不理身后任何人，总是回“Hello World！”。在*配置*中调用的大多数方法都是扩展方法，在管道中添加一个请求处理器(*中间件*)。*

我们的博文到此结束。我们已经了解了 ASP.NET 核心应用程序的基本结构，以及如何使用反射和命名约定将事物联系在一起。我们还学习了 *Startup* 类的角色及其 *Configure* 和 *ConfigureServices* 方法。要了解更多，请查看[ASP.NET 官方核心文档](https://docs.microsoft.com/en-us/aspnet/core/)。

* * *

**了解更多关于。NET Core 上红帽平台，拜访[RedHatLoves.NET](http://redhatloves.net/)。**

*Last updated: September 3, 2019*