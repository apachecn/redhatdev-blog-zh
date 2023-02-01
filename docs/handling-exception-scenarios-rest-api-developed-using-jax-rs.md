# 在使用 JAX-RS 开发的 REST API 中处理异常场景

> 原文：<https://developers.redhat.com/blog/2017/10/02/handling-exception-scenarios-rest-api-developed-using-jax-rs>

***先决条件*** :使用 JAX-RS 开发 REST API 的动手知识。

几年来，REST 服务已经成为复杂企业应用程序不可或缺的一部分。开发人员通常更喜欢下面列出的两种 API，用于在他们的企业应用程序中构建 REST API。

1.  ***【JAX-RS】**–**JEE*规范的一部分，有不同的实现，如 *RestEasy* 、 *Jersey* 、 *Restlet* 等。
2.  **–**一个开源的 Spring 社区项目，最适合基于微服务的应用。**

**在本文中，我们将学习如何在使用 *JAX-RS 构建的 Rest APIs 中优雅地处理异常。***

 **### **例子**

让我们以 Twitter 这样的社交媒体应用程序为例，它部署在 Tomcat 这样的应用程序/Web 服务器上。Twitter API 中的 REST 端点示例如下:

```
/tweet/{tweetID}
// Gives Client a particular tweet based on tweet ID provided.
```

现在，如果客户端发送了一个数据库中没有可用 tweet 的 tweetID。例如:

```
/tweet/25
// no content(as Response)
```

这不是 Rest 服务的客户端想要看到的。在这种情况下，我们可以创建自己的自定义异常。这样，如果有人试图访问服务器上没有的 tweet，我们只需抛出一个异常，而不是返回空内容。

**第一步:创建自定义异常**

```
public class INFONotFoundException extends RuntimeException {
    private static final long serialVersionUID = 1L;

    public INFONotFoundException(String message) {
        super(message);
    }
}
```

现在，如果我们访问`/tweet/25`但是特定的 tweet ID 不可用，我们会得到 Tomcat 错误页面，因为我们在我们的业务类中抛出了 *INFONotFoundException* ，但是没有捕捉到它。它出现在 JAX_RS 中，它也不知道如何处理这个异常，所以它使用 servlet 容器，该容器有一个默认行为，在服务器端代码发生错误时显示标准错误页面。

但是，我们不希望异常的 HTML 错误页面作为 Rest API 端点的响应。我们需要一个 JSON 有效负载作为响应，这样它对使用 REST API 的客户端就很有用，这样他们就可以相应地解析它。为此，我们希望 JAX-RS 框架能够捕捉到它，并在它到达 servlet 容器之前返回一个 JSON 有效载荷。

**步骤 2:使用 JAX-RS *异常映射器*将定制异常映射到 JSON 负载(例如 ErrorResponse)**

```
public class ErrorResponse {
  String errorMessage;
  String errorCode;
  String documentationLink;

  public ErrorResponse(String errorMessage, String errorCode, String documentationLink) {
    super();
    this.errorMessage = errorMessage;
    this.errorCode = errorCode;
    this.documentationLink = documentationLink;
  }

  public ErrorResponse() {

  }
  // Getters and Setters
}
```

```
@Provider
public class INFONotFoundExceptionMapper implements ExceptionMapper<INFONotFoundException> {
  @Override
  public Response toResponse(INFONotFoundException ex) {
    ErrorResponse response = new ErrorResponse(ex.getMessage(), "503",
      "https://docs.oracle.com/javaee/7/api/javax/ws/rs/ext/ExceptionMapper.html");
    return Response.ok().entity(response).build();
  }
}
```

现在，当 JAX-RS 在应用程序中看到任何 *INFONotFoundException* 时，它会搜索应用程序中所有用`@Provider`注释的 *ExceptionMapper* ，并试图找到一个可以将 *INFONotFoundException* 映射到 *ErrorResponse* 的映射器。一旦找到合适的 *ExceptionMapper* ，它会将抛出的异常传递给 *ExceptionMapper* 中*to response(Exception ex)*的方法参数，并通过调用 ExceptionMapper 的 *toResponse(Exception e)* 方法构建一个 JSON 负载。

现在，客户端再也看不到 HTML 错误页面，JAX-RS 给出了一个很好的 JSON 有效载荷，可以帮助客户端进行故障排除。

***注:***

1.我们可以在 REST API 中包含 *ExceptionMapper* ,以便处理特定的异常，从而对客户端更有帮助。

2.如果开发人员不想为应用程序中抛出的每个异常创建多个映射器，那么创建一个通用的异常映射器总是一个好的做法。这样，无论我们的服务器端 REST API 出了什么问题(例如，NullPointerException、DataNotFoundException 等等)。)，JAX-RS 总是试图用 *GenericExceptionMapper* 来映射它，我们从来没有看到 HTML 错误页面，而是看到一个 JSON 有效载荷，这对服务客户端很有帮助。

```
@Provider
public class GenericExceptionMapper implements ExceptionMapper<Throwable> {

  @Override
  public Response toResponse(Throwable arg0) {
    // TODO Auto-generated method stub
    return null;
  }
}
```

*Last updated: June 8, 2021***