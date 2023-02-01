# Jakarta EE:创建企业 JavaBeans 计时器

> 原文：<https://developers.redhat.com/blog/2019/12/13/jakarta-ee-creating-an-enterprise-javabeans-timer>

Enterprise JavaBeans (EJB)有许多有趣且有用的特性，其中一些我将在本文和后续文章中重点介绍。在本文中，我将向您展示如何通过编程和注释创建一个 [EJB 计时器](https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/undejdev012.htm)。我们走吧！

EJB 计时器功能允许我们根据日历配置安排要执行的任务。这非常有用，因为我们可以使用 [Jakarta](https://developers.redhat.com/blog/2019/09/12/jakarta-ee-8-the-new-era-of-java-ee-explained/) 上下文的能力来执行预定的任务。当我们基于计时器运行任务时，我们需要回答一些关于并发性的问题，任务被调度到哪个节点上(如果是集群中的应用程序)，如果任务没有执行会采取什么行动，等等。当我们使用 EJB 计时器时，我们可以将这些问题委托给 Jakarta 上下文，并更多地关注业务逻辑。很有趣，不是吗？

## 以编程方式创建 EJB 计时器

我们可以使用编程方法，根据业务逻辑安排 EJB 计时器运行。根据传递给流程的参数值，当我们需要动态行为时，可以使用此方法。让我们看一个 EJB 计时器的例子:

```
import javax.annotation.Resource;
import javax.ejb.SessionContext;
import javax.ejb.Stateless;
import javax.ejb.Timeout;
import java.util.logging.Logger;

@Stateless
public class MyTimer {

    private Logger logger = Logger.getLogger(MyTimer.class.getName());
    @Resource
    private SessionContext context;

    public void initTimer(String message){
        context.getTimerService().createTimer(10000, message);
    }

    @Timeout
    public void execute(){
        logger.info("Starting");

        context.getTimerService().getAllTimers().stream().forEach(timer -> logger.info(String.valueOf(timer.getInfo())));

        logger.info("Ending");
    }    
}

```

若要调度此 EJB 计时器，请调用此方法:

```
@Inject
private MyTimer myTimer;
....
```

```
myTimer.initTimer(message);
```

经过 10000 毫秒后，将调用用@Timeout 注释的方法。

## 使用注释调度 EJB 定时器

我们还可以创建一个 EJB 计时器，根据注释配置自动调度运行。看看这个例子:

```
@Singleton
public class MyTimerAutomatic {

    private Logger logger = Logger.getLogger(MyTimerAutomatic.class.getName());

    @Schedule(hour = "*", minute = "*",second = "0,10,20,30,40,50",persistent = false)
    public void execute(){

        logger.info("Automatic timer executing");

    }
}

```

正如您所看到的，要配置自动 EJB 计时器计划，您可以使用@Schedule 对该方法进行注释，并配置日历属性。例如:

```
@Schedule(hour = "*", minute = "*",second = "0,10,20,30,40,50",persistent = false)
```

如您所见，execute 方法被配置为每 10 秒调用一次。您还可以配置计时器是否持久。

## 结论

EJB 定时器是一个很好的 EJB 功能，有助于解决许多问题。使用 EJB 计时器功能，我们可以调度要执行的任务，从而将一些责任委托给 Jakarta context 来为我们解决。此外，我们可以创建持久计时器，控制并发执行，并在集群环境中使用它。如果你想看完整的例子，请访问 GitHub 上的这个[库](https://github.com/rhuan080/sampleejbtime)。

*Last updated: July 1, 2020*