# OAuth 访问委托中代理背后的基于角色的访问控制

> 原文：<https://developers.redhat.com/blog/2019/12/27/role-based-access-control-behind-a-proxy-in-an-oauth-access-delegation>

在我之前的[文章中，](https://developers.redhat.com/blog/2018/10/08/configuring-nginx-keycloak-oauth-oidc/)我用 [Keycloak](https://www.keycloak.org/index.html) 演示了在 NGINX 中启用基于 OAuth 的授权的完整实现，其中 NGINX 充当授权代码授权的中继方。NGNIX 还可以充当后端应用程序(如 Tomcat、Open Liberty、WildFly 等)的反向代理服务器。)，它可以托管在企业应用服务器上。

有时，托管在 NGINX 后面的后端应用程序需要有基于角色的访问控制(RBAC)，因为 RBAC 有助于根据用户的角色来限制资源。在 OAuth 中，与用户相关联的角色在 JSON Web 令牌(JWT)中可用，因此用户可以从 ID 或访问令牌中捕获声明，但同样的内容应该由 NGINX 通过头共享:

```
ngx.req.set_header("X-USER", res.id_token.sub)
ngx.req.set_header("X-ROLE", res.id_token.role)

```

这里，我基于主题声明设置主体，基于 ID 令牌中可用的角色声明设置角色。通常，角色声明在 ID 令牌中是不可用的，但是可以通过在 Keycloak 中配置带有相关客户端的映射器来添加此信息。

因为我需要基于 HTTP 头中捕获的用户名创建主体，所以我扩展了`HttpServletRequestWrapper`来设置普通应用程序流中的主体:

```
public class ConfigPrincipal extends HttpServletRequestWrapper {
    String user;
    List<String> roles;
    HttpServletRequest realRequest;
    public ConfigPrincipal(String user, List<String> roles, HttpServletRequest request) {
      super(request);
      this.user = user;
      this.roles = roles;
      this.realRequest = request;
   }

   @Override
   public boolean isUserInRole(String role) {
    if (roles == null) {
      return this.realRequest.isUserInRole(role);
    }
    return roles.contains(role);
   }

   @Override
   public Principal getUserPrincipal() {
     if (this.user == null) {
       return realRequest.getUserPrincipal();
     }
     //make an anonymous implementation to just return our user
     return new Principal() {
     @Override
       public String getName() {
         return user;
       }
     };
   }

   public boolean authenticate(){
     return true;
   }
}
```

接下来，我定义了一个附加到应用程序的 servlet 过滤器，以便它在应用程序的业务逻辑之前执行:

```
  @Override
  public void doFilter(ServletRequest req, ServletResponse response,
           FilterChain next) throws IOException, ServletException {

      HttpServletRequest request = (HttpServletRequest) req;
      String user = request.getHeader("X-USER");
      List<String> roles = new ArrayList<String>();

      //Capturing roles from the request header. 
      if (request.getHeader("role") != null) {
         String[] substrrole = request.getHeader("X-SSBRoleLevel").split(",");
         for (int i = 0; i < substrrole.length; i++) {
            roles.add(substrrole[i]);
         }
      }
      System.err.println("sidde roles:" + roles);
     //call the request wrapper , which overrides getUserPrincipal and is UserInRole
     next.doFilter(new ConfigPrincipal(user, roles, request), response);
   }
```

现在，`request.getUserPrincipal()`可以在应用程序中的任何地方执行来捕获主体，而`request.isUserInRole(role)`可以用来捕获相关联的角色以拥有基于角色的访问控制。

*Last updated: July 1, 2020*