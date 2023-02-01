# Red Hat 3scale API 管理，第 2 部分:使用速率限制策略保护 API 中的自定义策略

> 原文：<https://developers.redhat.com/blog/2021/05/04/custom-policies-in-red-hat-3scale-api-management-part-2-securing-the-api-with-rate-limit-policies>

在本系列的[第 1 部分](/blog/2021/02/24/custom-policies-in-red-hat-3scale-api-management-part-1-overview/)中，我们讨论了 [Red Hat 3scale API 管理](/products/3scale/overview)中的策略框架——向 APIcast 网关添加策略以定制 API 请求和响应行为。在本文中，我们将研究向 APIcast 网关添加速率限制、后端 URL 保护和边缘限制策略。我们还将回顾哪些策略适合用于不同的用例。

## 作为反向代理的 API 网关

API 网关的主要职责之一是保护 API 端点。API 网关充当反向代理，所有 API 请求都通过网关流向后端 API。通过 API 网关公开的 API 通过以下方式得到保护:

*   **速率限制**通过强制限制每个 URL 路径、方法或用户和帐户计划限制来控制到达 API 的请求数量。这是 3scale API 管理的标准功能，可以使用 [API 打包和计划](/blog/2021/03/02/packaging-apis-for-consumers-with-red-hat-3scale-api-management/)来实现。您可以配置额外的策略来限制允许的 IP 范围，使用速率限制标头进行响应，并在维护期间关闭所有到后端的流量。
*   **认证**提供了一种唯一识别请求者的方法，并且只允许访问经过认证的账户。这种身份验证可以基于请求者的身份进行。3scale API 管理支持通过 API(用户)密钥、应用标识符和密钥对或基于 OAuth 2.0 的 OpenID Connect (OIDC)进行身份验证。
*   **授权**允许您基于角色管理用户和帐户访问。这超越了身份验证，通过查看用户配置文件来确定用户或组是否应该有权访问所请求的资源。这在 3scale API 管理中通过将用户和帐户分配给特定计划来配置。通过检查身份提供者共享的 JWT (JSON Web Token)并应用角色检查策略，可以为 OIDC 安全服务提供更细粒度的访问控制。

在本文中，我们将主要关注 APIcast 中通过策略提供的不同访问控制和速率限制选项。

### 应用配置示例

要应用本文中列出的任何配置示例，请向 Admin API 端点发送一个 PUT 请求:

```
https://<<3scale admin URL>>/admin/api/services/<<service id>>/proxy/policies.json

```

输入以下信息:

*   指定**3 规模管理门户 URL** ( `<<3scale admin URL>>`)。
*   输入 API 产品的**服务 ID 号** ( `<<service id>>`)，可在 3scale 的产品概述页面上找到。
*   可以在请求体中传递**配置** JSON。
*   管理员访问令牌也可以在请求体中传递。

以下是请求的示例:

```
curl -v  -X PUT   -d 'policies_config=[{"name":"apicast","version":"builtin","configuration":{},"enabled":true}]&access_token=redacted'  https://red-hat-gpte-satya-admin.3scale.net/admin/api/services/18/proxy/policies.json

```

如果请求成功，管理 API 将发送一个 HTTP 200 响应。

## 匿名访问策略

在 3scale API 管理中，您必须使用三种[身份验证方法](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/administering_the_api_gateway/index#supported_authentication_patterns)中的一种来访问 API。一个[匿名访问策略](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/administering_the_api_gateway/index#anonymous_access)让您绕过这个要求，这样就可以在请求中不提供认证的情况下发出 API 请求。

只有当服务被设置为使用 API 密钥方法或`App_ID`和`App_Key Pair`认证机制时，才可以使用该策略。该策略不适用于需要 OIDC 认证的 API。

对于该策略，提供商需要创建一个应用程序计划和一个具有有效凭据的应用程序。使用该策略，如果请求不提供任何凭证，则可以在请求期间将这些凭证提供给 API 端点。

### 何时使用匿名访问

在下列情况下，您可以考虑使用此策略:

*   当 API 消费者不能传递认证凭证时，因为它们是遗留的或者不能支持它们。
*   出于测试目的，您可以在网关中重用现有的凭证。
*   在开发或登台环境中，使面向客户的应用程序的开发人员更容易获得 API 访问。

请注意以下注意事项:

*   该请求是匿名的，因此将该策略与 IP 检查策略(将在本文后面讨论)结合起来，以确保 API 端点受到保护。
*   确保提供速率限制并拒绝对创建、更新和删除操作的访问，以避免误用。
*   避免在生产环境中使用此策略。如有必要，将其用作战术解决方案，直到消费者可以迁移到使用经过身份验证的端点。

### 如何配置匿名访问

在策略链中，匿名访问策略需要在 APIcast 策略之前配置。以下是完整配置的示例:

```
[

      {

        "name": "default_credentials",

        "version": "builtin",

        "configuration": {

          "auth_type": "user_key",

          "user_key": "16e66c9c2eee1adb3786221ccffa1e23"

        }

      },

      {

        "name": "apicast",

        "version": "builtin",

        "configuration": {}

      }

 ]
```

## 维护模式策略

因为 APIcast 网关充当反向代理，所以所有经过身份验证的请求都被路由到后端 API。在某些情况下，可以通过使用[维护模式](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/administering_the_api_gateway/index#maintenance-mode)策略来阻止请求到达后端。当 API 由于定期维护而不接受请求时，或者当后端 API 关闭时，这很有帮助。

### 何时使用维护模式

在以下情况下，考虑使用此策略:

*   后端 API 因维护而关闭。
*   当 API 不可用或返回内部服务器错误时，您希望向消费者提供更有意义的响应。
*   API 正在更新，或者 API 策略正在更改。

### 如何配置维护模式

在策略链中，维护策略需要在 APIcast 策略之前配置。以下是完整配置的示例:

```
[

      {

        "name": "maintenance_mode",

        "version": "builtin",

        "configuration": {

          "message_content_type": "text/plain; charset=utf-8",

          "status": 503,

          "message": "Service Unavailable - Maintenance. Please try again later."

        }

      },

      {

        "name": "apicast",

        "version": "builtin",

        "configuration": {}

      }

    ]

```

## IP 检查策略

API 提供者需要能够只允许来自某一组预先配置的 IP 的 API 调用，或者拒绝来自一组 IP 的调用，以防止滥用 API。默认情况下，APIcast 网关将 API 公开为 HTTP/HTTPS 端点，并且不拒绝或允许基于请求者 IP 的调用。将 [IP 检查](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/administering_the_api_gateway/index#ip_check)策略添加到策略链允许您将此功能配置到策略链。

### 何时使用 IP 检查

以下是您可能需要使用此策略的一些示例:

*   API 仅允许内部客户使用，因此 API 端点仅允许用于网络中的一组 IP 或 IP 块。
*   有一组已知的 IP 因滥用 API 或提出恶意请求、DDoS 攻击等而被提供商阻止。网关可以阻止此类 IP 来拒绝请求，而无需到达 API。
*   API 提供者希望提供一个 IP 块来允许一组合作伙伴 IP 访问 API，同时防止网络外部的请求者访问它。

请注意，APIcast IP 检查是第 7 层 OSI 7 层模型，适用于 TCP/IP 层。IP 检查发生在应用层。对于更健壮的策略，可以在 MAC 地址(第 2 层)或为第 3 层和第 4 层配置的防火墙策略上使用 LAN 网络白名单或黑名单策略。

### 如何配置 IP 检查

在策略链中，IP 检查策略需要在 APIcast 策略之前配置。

使用以下配置，并在 POST 请求中向管理 API 发出请求:

```
[

      {

        "name": "ip_check",

        "version": "builtin",

        "configuration": {

          "error_msg": "IP address not allowed",

          "client_ip_sources": [

            "last_caller",

            "X-Forwarded-For",

            "X-Real-IP"

          ],

          "ips": [

            "1.2.3.4",

            "1.2.3.0/4"

          ],

          "check_type": "whitelist"

        }

      },

      {

        "name": "apicast",

        "version": "builtin",

        "configuration": {}

      }

    ]
```

## 边缘限制策略

在 3scale 中，您可以通过在应用计划中设置限制来控制对后端 API 的 API 调用数量。费率限制可根据计划类型、定价模式或访问控制要求进行设置。然而，这个速率限制没有考虑后端 API 的吞吐量。如果应用程序和请求的数量随着 API 订户数量的增加而增加，这可能会导致 API 不堪重负。一个[边缘限制策略](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/administering_the_api_gateway/index#edge_limiting)在一个给定的时间段内对所有应用的后端 API 强制执行请求总数，从而后端 API 得到保护。您可以设置速率限制来强制执行每秒并发请求数或允许的并发连接数。

应用程序计划仅对每个应用程序实施费率限制。特定帐户的所有用户共享应用程序，因此您可以使用边缘限制策略，通过 JWT 索赔检查或请求参数来唯一识别用户，从而允许特定于用户的速率限制。这确保了没有单个帐户用户可以独占为应用程序设置的呼叫限制。使用 [liquid templates](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/administering_the_api_gateway/index#using_variables_and_filters) ，你可以根据远程 IP 地址、报头、JWT 变量或 URL 路径参数等变量来设置速率限制。

边缘限制策略使用 open resty[Lua-resty-limit-traffic](https://github.com/openresty/lua-resty-limit-traffic)库。该政策允许设置以下限制:

*   `leaky_bucket_limiters`，基于漏桶算法，建立在平均请求数加上最大突发大小的基础上。
*   `fixed_window_limiters`，基于固定的时间窗口:持续 n 秒。
*   `connection_limiters`，基于并发连接数。

策略通常应用于网关级别，但是边缘限制策略可以是服务级别的，并且应用于对后端的请求总数，而不管部署了多少个网关。在多个网关共享边缘限制的情况下，可以使用和配置外部 Redis 存储数据库。

### 何时使用边缘限制

以下是您可能考虑使用此策略的一些示例:

*   您希望对所有用户、帐户和应用程序的后端 API 设置一个总体限制。
*   您希望控制拥有数百个消费者的流行 API 的吞吐量。例如，如果 API 可以处理 100 个并发连接，那么可以相应地设置边缘限制策略，它将适用于所有应用程序。
*   一个应用程序计划可能有每分钟 10 个请求的速率限制，但是使用该计划的应用程序的数量取决于消费者的数量。如果有 1，000 个应用程序，那么理论上，每分钟最多可以允许 10，000 个请求。设置边缘限制会根据 API 容量强制实施同时使用限制。
*   一个应用程序计划允许每分钟 100 个请求，但是一个客户端 IP 发出 100%的请求，而具有相同帐户的其他请求者无法访问。设置每个客户端 IP 每分钟 10 个请求的限制可确保该计划在整个帐户中公平使用。

### 如何配置边缘限制

需要在策略链中的 APIcast 策略之前配置边缘限制策略。以下示例演示了一些示例配置。

要使用外部 Redis 存储数据库全局设置并发连接数率限制，请执行以下操作:

```
[

      {

        "name": "rate_limit",

        "version": "builtin",

        "configuration": {

          "limits_exceeded_error": {

            "status_code": 429,

            "error_handling": "exit"

          },

          "configuration_error": {

            "status_code": 500,

            "error_handling": "exit"

          },

          "fixed_window_limiters": [],

          "connection_limiters": [

            {

              "condition": {

                "combine_op": "and"

              },

              "key": {

                "scope": "service",

                "name_type": "plain"

              },

              "conn": 100,

              "burst": 50,

              "delay": 1

            }

          ],

          "redis_url": "redis://gateway-redis:6379/1"

        }

      },

      {

        "name": "apicast",

        "version": "builtin",

        "configuration": {}

      }

    ]
```

要为客户端 IP 地址设置漏桶限制器:

```
[

      {

        "name": "rate_limit",

        "version": "builtin",

        "configuration": {

          "limits_exceeded_error": {

            "status_code": 429,

            "error_handling": "exit"

          },

          "configuration_error": {

            "status_code": 500,

            "error_handling": "exit"

          },

          "fixed_window_limiters": [],

          "connection_limiters": [],

          "redis_url": "redis://gateway-redis:6379/1",

          "leaky_bucket_limiters": [

            {

              "condition": {

                "combine_op": "and"

              },

              "key": {

                "scope": "service",

                "name_type": "liquid",

                "name": "{{ remote_addr }}"

              },

              "rate": 100,

              "burst": 50

            }

          ]

        }

      },

      {

        "name": "apicast",

        "version": "builtin",

        "configuration": {}

      }

    ]
```

若要为标题匹配设定固定窗口限制器:

```
[

      {

        "name": "rate_limit",

        "version": "builtin",

        "configuration": {

          "limits_exceeded_error": {

            "status_code": 429,

            "error_handling": "exit"

          },

          "configuration_error": {

            "status_code": 500,

            "error_handling": "exit"

          },

          "fixed_window_limiters": [

            {

              "window": 10,

              "condition": {

                "combine_op": "and"

              },

              "key": {

                "scope": "service",

                "name_type": "liquid",

                "name": "{{ jwt.sub }}"

              },

              "count": 100

            }
r
          ]

        }

      },

      {

        "name": "apicast",

        "version": "builtin",

        "configuration": {}

      }

    ]
```

## 结论

在本文中，我们看到了如何使用策略来微调为 APIcast 网关设置的速率限制和访问控制。在下一篇文章中，我们将探讨如何使用高级安全策略与 OIDC 一起提供授权控制。

您可以试用 3scale API 管理平台并创建策略，如本文所示，只需免费注册。

*Last updated: October 14, 2022*