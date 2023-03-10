# 应用服务器在 Kubernetes 时代已经死亡了吗？(第二部分)

> 原文：<https://developers.redhat.com/blog/2018/10/02/are-app-servers-dead-in-the-age-of-kubernetes-part-2>

欢迎来到关于 Kubernetes、应用服务器和未来的系列文章的第二篇。第 1 部分， [Kubernetes 是新的应用程序操作环境](https://developers.redhat.com/blog/2018/09/05/kubernetes-new-operating-environment)，讨论了 Kubernetes 及其在应用程序开发中的地位。在这一部分中，我们将探讨应用服务器及其在 Kubernetes 中的作用。

您可能还记得第 1 部分的[中，我们探讨了在](https://developers.redhat.com/blog/2018/09/05/kubernetes-new-operating-environment)[中提出的观点，即为什么 Kubernetes 是新的应用服务器](https://developers.redhat.com/blog/2018/06/28/why-kubernetes-is-the-new-application-server/)，并思考这些观点对 Java EE、 [Jakarta EE](https://developers.redhat.com/blog/2018/04/24/jakarta-ee-is-officially-out/) 、Eclipse MicroProfile 和应用服务器意味着什么。这是应用服务器的谢幕吗？我们是否看到了他们的受欢迎程度和使用率即将下降的开始？

在回答这个问题之前，我们需要讨论应用服务器的用例。然后我们才能决定它是否仍然是一个有效的用例。

## 应用服务器有什么用？

为什么我们有应用服务器？他们的目的是什么？这些是我们在思考他们的问题空间时可以问的一些问题。

应用服务器不会无缘无故地突然出现。它们是 CORBA 和 DCOM 模型的发展，为安全、交易、消息传递等提供独立的服务。所有这些被分离的服务需要它们之间的大量通信。

应用服务器诞生于将所有这些服务置于一个单一流程下的需求。除了协同定位服务之外，它们还在顶层提供了一个框架，使得组合服务的功能变得更加容易。

听起来熟悉吗？的确如此。CORBA 和 DCOM 在架构上类似于我们今天所说的[微服务](https://developers.redhat.com/topics/microservices/)。

## 现在是应用服务器的死亡吗？

Kubernetes 并不是我们今天所知的应用服务器的丧钟。随着硬件和软件的改进，应用服务器一直在发展，并将继续发展。开发人员的生产力在不断提高。Kubernetes、Docker 和现在的 [service mesh](https://developers.redhat.com/topics/service-mesh/) 是应用服务器转变的另一个必要步骤。这并不意味着它们无关紧要。

## 应用服务器重生

如果有什么不同的话，Kubernetes 作为操作环境的出现将导致应用服务器的另一次变革。就像凤凰涅槃一样，应用服务器将会自我改造，这在过去已经发生过很多次了。

第一篇文章谈到了考虑 Kubernetes 提供的内容以及我们的应用或微服务的需求。它指出，Kubernetes 是一个很好的应用服务器，适合不与许多其他服务交互的部署。相反，大多数 Java EE 和 MicroProfile 应用程序并没有隔离到可以用这种方式处理的程度。

让我们来看看为什么 Java EE 和 MicroProfile 应用程序通常不那么孤立。无论应用程序是在单个进程中，还是分布在整个网络中，它们都是由许多服务组成的，这些服务共同致力于一个业务目标——尽管它们很可能不是以这种方式开始的。

随着业务需求的变化，具有单一业务目标的应用程序会快速增长。或者一个人开发的应用程序需要修改以适应更广泛的用途。应用程序需要增长和调整的原因有很多。复杂的是，在开始时，我们通常不知道应用程序会增长。

## 作为应用服务器的 Kubernetes

使用 Kubernetes 作为应用程序服务器来规划一个简单的应用程序很快就会出现问题，因为应用程序的扩展超出了最初的目标，需要框架来集成和提供超出 Kubernetes 本身所提供的功能。

应用程序通常与其他应用程序、服务和系统交互，或者与自身以外的任何东西交互。这样做可以保证应用程序需要诸如以下内容:

*   与异步或离线处理的消息传递系统集成
*   自身内部和跨其他调用的事务
*   细粒度的安全控制，与粗粒度的安全控制相反

无论您是使用 WildFly 还是使用 [Thorntail](https://developers.redhat.com/blog/2018/08/23/eclipse-microprofile-and-red-hat-update-thorntail-and-smallrye/) 部署到，这些都是应用服务器和胖罐子为当今的应用和微服务提供的关键东西。这些担忧不会消失，Kubernetes 或其他建立在此基础上的项目也不会提供这些担忧，比如 [Red Hat OpenShift](http://openshift.com/) 和 [Istio](https://developers.redhat.com/topics/service-mesh/) 。

## 在第 3 部分中

本系列的最后一部分将结束我们的分析。它将回答 Kubernetes 和应用服务器的以下问题:它们能共存吗？

*Last updated: March 23, 2022*