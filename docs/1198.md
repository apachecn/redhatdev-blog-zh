# Camel Rest DSL 和 Spring Boot 的单元测试

> 原文：<https://developers.redhat.com/blog/2018/04/04/testing-camel-rest-dsl-spring-boot>

希望到现在为止，您已经知道如何使用 [Spring Boot](http://camel.apache.org/spring-boot.html) 编写您的第一个 [Rest DSL](http://camel.apache.org/rest-dsl.html) Camel Route。如果没有，先查看[这篇文章](https://developers.redhat.com/blog/2018/03/26/camel-spring-boot-rest-dsl/)。现在您已经写好了路线，是时候为它写一个单元测试了。许多人发现 [Apache Camel 单元测试](http://camel.apache.org/testing.html)是一个很大的难题。幸运的是，在 Apache Camel Rest DSL 测试中使用 Spring Boot 时，Rest 路线并不太难。

首先，您将设置您的测试类:

```
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class SampleCamelApplicationTest {
}
```

现在您的测试类已经设置好了，您需要添加所有测试方法中需要的任何变量。由于这个单元测试将测试一个 Rest 服务，我们需要将 TestRestTemplate 注入到测试类中:

```
import org.springframework.boot.test.web.client.TestRestTemplate;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class SampleCamelApplicationTest {
   @Autowired
   private TestRestTemplate restTemplate;
}
```

最后，编写测试类。使用 Spring Boot 进行测试允许您向 Rest 服务发出请求，并验证您是否收到了预期的响应。

```
@Test
public void sayHelloTest() {
   // Call the REST API
   ResponseEntity<String> response = restTemplate.getForEntity("/camel/hello", String.class);
   assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
   String s = response.getBody();
   assertThat(s.equals("Hello World"));
}

```

不要忘记在[红帽峰会](https://www.redhat.com/en/summit/2018)上与 Claus Ibsen 一起参加我的[研讨会，了解更多关于其余 DSL 的信息；如何测试这条路线；部署到](https://agenda.summit.redhat.com/SessionDetail.aspx?id=154680)[红帽 OpenShift](https://developers.redhat.com/products/openshift/overview/)以及如何使用 [3scale by Red Hat](https://www.redhat.com/en/technologies/jboss-middleware/3scale) 来管理您的 API。

*Last updated: January 29, 2019*