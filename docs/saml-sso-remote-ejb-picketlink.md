# 通过 Picketlink 启用基于 SAML 的 SSO 和远程 EJB

> 原文：<https://developers.redhat.com/blog/2018/01/03/saml-sso-remote-ejb-picketlink>

假设您有一个远程企业 JavaBeans (EJB)应用程序，其中 EJB 客户端是安全断言标记语言(SAML)架构中的服务包(SP)应用程序。您希望使用用于 SP 的相同断言来验证您的远程 EJB。

在继续本教程之前，你应该对 [EJB](http://www.oracle.com/technetwork/java/javaee/ejb/index.html) 和[皮特林克](http://picketlink.org/about/)有个基本的了解。

I have developed a POC based on Picketlink. Below are the things I have done to achieve it.

*   将 Picketlink 设置为在身份提供者(IDP)应用程序中对响应和声明进行签名。
    在 IDP 应用的 picketlink.xml 中配置:
    ` SAML 2 signaturecenthandler `如下。

```
    <Handler class="org.picketlink.identity.federation.web.handlers.saml2.SAML2SignatureGenerationHandler">
         <Option Key="SIGN_RESPONSE_AND_ASSERTION" Value="true"/>
     </Handler>
```

*   设置 picketlink 以在 SP 应用程序的 http-session 中存储 SAML 断言。
    在 SP 应用的 picketlink.xml 中配置:
    `SAML2AuthenticationHandler `如下。

```
    <Handler class="org.picketlink.identity.federation.web.handlers.saml2.SAML2AuthenticationHandler">
         <Option Key="ASSERTION_SESSION_ATTRIBUTE_NAME" Value="org.picketlink.sp.assertion"/>
     </Handler>
```

*   配置 sp 安全域，如下所示，为登录模块和 IDP 应用程序设置标志“足够”。您可以配置自己定制的登录模块，也可以使用 picketbox 中提供的任何登录模块。

```
      <security-domain name="sp" cache-type="default">
         <authentication>
            <login-module code="org.picketlink.identity.federation.bindings.jboss.auth.SAML2LoginModule" flag="sufficient" />
            <login-module code="org.picketlink.identity.federation.bindings.jboss.auth.SAML2STSLoginModule" flag="sufficient">
                 <module-option name="roleKey" value="Role"/>
                 <module-option name="localValidation" value="true"/>
                 <module-option name="localValidationSecurityDomain" value="sp"/>
            </login-module>
          </authentication>
       </security-domain>
```

*   在您的 servlet (EJB 客户端/SP 应用程序)中，您需要获得已签名的断言，并将其与 EJB 调用一起发送以进行验证，如下所示。这里，我使用了快速入门[1]中提供的 ejb-remote 应用程序。

```
//Getting Signed SAML Assertion
public String getSignedAssertion(HttpServletRequest httpRequest) throws Exception {
         HttpSession session = httpRequest.getSession();
         String cachedSignedAssertion = (String) session.getAttribute("org.picketlink.sp.assertion.signed");
         if (cachedSignedAssertion == null) {
             Document assertion = (Document) session.getAttribute("org.picketlink.sp.assertion");
             String stringSignedAssertion = DocumentUtil.asString(assertion);
             System.out.println(stringSignedAssertion);
             return stringSignedAssertion;
         } else {
             System.out.println("...cached assertion...");
             return cachedSignedAssertion;
         }
     }

//EJB invocation
public void getInitialContext(String assertion, String username) throws Exception, NamingException {

         Properties props = new Properties();
         props.put("remote.connectionprovider.create.options.org.xnio.Options.SSL_ENABLED", "false");
         props.put(Context.URL_PKG_PREFIXES, "org.jboss.ejb.client.naming");
         props.put("remote.connections", "default");
         props.put("remote.connection.default.port", "4447");
         props.put("remote.connection.default.host", "10.10.10.10");
         System.out.println("Connecting...");
         props.put("remote.connection.default.username", username);
         props.put("remote.connection.default.password", assertion);
         props.put("remote.connection.default.connect.options.org.xnio.Options.SASL_POLICY_NOPLAINTEXT","false");
         props.put("remote.connection.default.connect.options.org.xnio.Options.SASL_POLICY_NOANONYMOUS","false");
         Context context = new InitialContext(props);
         RemoteCounter aa = (RemoteCounter) context.lookup("ejb:/jboss-ejb-remote-server-side//CounterBean!org.jboss.as.quickstarts.ejb.remote.stateful.RemoteCounter?stateful");
         System.out.println(aa.getCount());
         aa.increment();
         System.out.println(aa.getCount());
         System.out.println("EJB Executed... using SAML assertion");
     }
```

You can get the IDP and SP sample of Picketlink at [2] below:

1.  [https://github . com/JBoss-developer/JBoss-EAP-quick starts/tree/6.4 . x/EJ b-remote](https://github.com/jboss-developer/jboss-eap-quickstarts/tree/6.4.x/ejb-remote)
2.  [https://github . com/JBoss-developer/JBoss-picket link-quick starts](https://github.com/jboss-developer/jboss-picketlink-quickstarts)

今天到此为止！

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: December 29, 2017*