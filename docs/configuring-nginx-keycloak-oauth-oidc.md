# 使用 Keycloak/Red Hat SSO 为 OAuth/OpenID Connect SSO 配置 NGINX

> 原文：<https://developers.redhat.com/blog/2018/10/08/configuring-nginx-keycloak-oauth-oidc>

在本文中，我将介绍如何使用 Keycloak/Red Hat 单点登录为基于 OAuth 的单点登录(SSO)配置 NGINX。这允许对联合身份使用 OpenID Connect (OIDC)。当 NGINX 充当后端应用服务器(例如 Tomcat 或 JBoss)的反向代理服务器时，这种配置很有帮助，在这种情况下，身份验证将由 web 服务器执行。

在这个设置中，Keycloak 将作为基于 OAuth 的 SSO 中的授权服务器，NGINX 将作为中继方。我们将使用 [lua-resty-openidc](https://github.com/zmartzone/lua-resty-openidc) ，它是 NGINX 实现 OpenID 连接依赖方(RP)和/或 OAuth 2.0 资源服务器(RS)功能的库。

这是一个基于 OIDC 的身份验证流程图:

[![OAuth-based authentication flow](img/f6d38896a3daca6c80614fb1f2c6655c.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/basic-oidc.jpg)

为了安装`lua-resty-oidc` **，**你需要在 NGINX 服务器上安装其他几个依赖模块:

*   [ngx_devel_kit](https://github.com/simplresty/ngx_devel_kit)
*   月球
*   [Lua-nginx-模块](https://github.com/openresty/lua-nginx-module)
*   [lua-cjson.php](https://www.kyne.com.au/~mark/software/lua-cjson.php)
*   [lua-resty-string](https://github.com/openresty/lua-resty-string)

## **安装说明**

1.  首先，我们创建一个目录来保存所有需要的包，然后我们将当前的工作目录更改为新创建的目录。在这里，我将作为`root`用户执行所有命令；也可以以非 root 用户的身份执行它们，但是有些命令，例如`yum`对非 root 用户不起作用，需要额外的步骤来执行。

    ```
    # mkdir /tmp/nginx-lua
    # cd /tmp/nginx-lua
    ```

2.  Now, download the packages that are required:
    a. Download NGINX version 1.13.6 and extract it:

    ```
    # wget http://nginx.org/download/nginx-1.13.6.tar.gz
    # tar -zxvf nginx-1.13.6.tar.gz
    ```

    b.下载 OpenSSL 并解压:

    ```
    # wget https://github.com/openssl/openssl/archive/OpenSSL_1_0_2g.tar.gz 
    # wget OpenSSL_1_1_0g.tar.gz
    ```

    c.下载`lua-nginx-module`并解压:

    ```
    # wget https://github.com/openresty/lua-nginx-module/archive/v0.10.13.tar.gz 
    # tar -zxvf v0.10.13.tar.gz
    ```

    d.下载`ngx_devel_kit`并解压:

    ```
    # wget https://github.com/simplresty/ngx_devel_kit/archive/v0.3.0.tar.gz
    # tar -zxvf v0.3.0.tar.gz
    ```

    e.下载 Lua 并解压:

    ```
    # wget http://www.lua.org/ftp/lua-5.1.5.tar.gz
    # tar -zxvf lua-5.1.5.tar.gz
    ```

    f.克隆`luaffib`并使用`luarocks`进行安装:

    ```
    # git clone https://github.com/facebookarchive/luaffifb
    # cd luaffifb
    # luarocks make
    ```

3.  Install the dependencies and packages required for `lua-resty-oidc`:
    a. We will first install Lua, so change the current working directory to `lua-5.1.5` and then execute the installation:

    ```
    # cd lua-5.1.5
    # make linux test
    # make install
    # cd ..
    ```

    b.安装`luarocks`:

    ```
    # yum install luarocks
    ```

    c.使用`luarocks`安装所有 Lua 模块:

    ```
    # luarocks install lua-cjson
    # luarocks install lua-resty-openidc
    ```

    d.Lua 安装完成后，导出`LUA_LIB`和`LUA_INC`的路径:

    ```
    # export LUA_LIB=/usr/local/lib/lua/5.1/
    # export LUA_INC=/usr/local/include/
    ```

    e.现在，我们需要安装开发工具，例如，gcc，c++等。

    ```
    # yum group install "Development Tools"
    # yum install readline-devel
    ```

    f.因为我们要做 NGINX 的二进制安装，所以需要安装`pcre`和`zlib`:

    ```
    # yum install pcre
    # yum install pcre-devel
    # yum install zlib
    # yum install zlib-devel
    ```

4.  现在，我们可以执行 NGINX 的安装，导航到 NGINX 二进制目录:

    ```
    # cd nginx-1.13.6
    # ./configure --prefix=/opt/nginx --with-http_ssl_module --with-ld-opt="-Wl,-rpath,/usr/local/lib/lua/5.1/" --add-module=/tmp/lua/ngx_devel_kit-0.3.0 --add-module=/tmp/lua/lua-nginx-module-0.10.13 --with-openssl=/tmp/lua/openssl-OpenSSL_1_0_2g
    # make -j2
    # make install
    ```

5.  成功执行安装命令后，NGINX 将安装在`/opt/nginx`中。
6.  Create a directory called `ssl` in the directory `/opt/nginx` and generate a self-signed certificate:

    ```
    # mkdir /opt/nginx/ssl
    # cd /opt/nginx/ssl
    # openssl req -nodes -newkey rsa:2048 -keyout private.pem -out certificate.csr -subj "/C=IN/ST=WestBengal/L=Kolkata/O=Red Hat/OU=APS/CN=www.example.com"
    # openssl x509 -req -in certificate.csr -out certificate.pem -signkey private.pem
    ```

    **注意:** certificate.csr 可以提交给 CA 供应商以获得签名证书。

7.  下载[键盘锁](https://downloads.jboss.org/keycloak/4.4.0.Final/keycloak-4.4.0.Final.zip)并解压。Keycloak 将作为身份提供者，NGINX 将作为服务提供者。

    ```
    # wget https://downloads.jboss.org/keycloak/4.4.0.Final/keycloak-4.4.0.Final.zip
    # unzip  keycloak-4.4.0.Final.zip -d /opt/keycloak
    ```

## 配置 Keycloak 和 NGINX

1.  在主领域中创建一个用户并启动 key cloak:

    ```
    # cd /opt/keycloak/keycloak-4.4.0.Final/bin
    # ./add-user-keycloak.sh -u admin -p admin@123 -r master      
    # ./standalone.sh -b www.example.com
    ```

2.  Create a new realm:
    a. Move the cursor near **Master** and click** Add Realm**.[![](img/d4c7b147786f7d03532722f43727b0f3.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Selection_055.png)
    b. Provide a name for your realm and click **Create**.

    | [![](img/7499a071e60695c8b5b59d1131739903.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Selection_056.png)

    **注意:**不需要创建新领域；可以在主领域中创建客户端。

3.  Now, we need to create a client for NGINX. Click **Client** in the left panel and click the **Create** button:[![](img/128c7341fc2145aaae83a2cd2379031d.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Selection_057.png)

    [![](img/9c74d07c1c191bd43cf4683432e95bc7.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Selection_058.png)

4.  选择 **openid-connect** 作为客户端协议，并将 NGINX URL 放入**根 URL** 字段:
    [![](img/af86dde66cebbb7f174a5affd39e6a25.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Selection_059.png)
5.  将**访问类型**设置为**机密**，点击**保存** : [![](img/ee2e07cf041953c50933b5acb02e6629.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Selection_060.png)
6.  点击**凭证**，复制稍后配置 NGINX 的密码: [![](img/e7ff1e0dfe979e3cb5b3002452d2f714.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Selection_061.png)
7.  在`nginx.conf`中的`http`块下添加下面一行:

    ```
    lua_package_path '~/lua/?.lua;;';
    resolver 8.8.8.8;
    # cache for discovery metadata documents
    lua_shared_dict discovery 1m;
    # cache for JWKs
    lua_shared_dict jwks 1m;
    ```

8.  使用如下代码在 NGINX 中创建服务器:

    ```
     server {
           listen     80 default_server;
           server_name  www.example.com;
           root     /opt/nginx/html;
           access_by_lua '
             local opts = {
               redirect_uri_path = "/redirect_uri",
               accept_none_alg = true,
               discovery = "http://www.example.com:8080/auth/realms/NGINX/.well-known/openid-configuration",
               client_id = "nginx",
               client_secret = "62d3b835-e3d1-4cec-a2f2-612f496bc6c3",
               redirect_uri_scheme = "http",
               logout_path = "/logout",
               redirect_after_logout_uri = "http://www.example.com:8080/auth/realms/NGINX/protocol/openid-connect/logout?redirect_uri=http://www.example.com/",
               redirect_after_logout_with_id_token_hint = false,
               session_contents = {id_token=true}
             }
             -- call introspect for OAuth 2.0 Bearer Access Token validation
             local res, err = require("resty.openidc").authenticate(opts)
             if err then
               ngx.status = 403
               ngx.say(err)
               ngx.exit(ngx.HTTP_FORBIDDEN)
             end
          ';
          # I disabled caching so the browser won't cache the site.
          expires           0;
          add_header        Cache-Control private;
          location / {
          }
          # redirect server error pages to the static page /40x.html
          #
          error_page 404 /404.html;
              location = /40x.html {
          }
          # redirect server error pages to the static page /50x.html
          #
          error_page 500 502 503 504 /50x.html;
              location = /50x.html {
          }
      }
    ```

9.  验证 NGINX 配置:

    ```
    # cd /opt/nginx/sbin/
    # ./nginx -t
    ```

10.  成功验证 NGINX 配置后，启动 NGINX 服务器:

    ```
    #./nginx
    ```

现在，当您访问受保护的 URL (www.example.com)时，您将被重定向到 http://www.example.com:8080/auth/realms/NGINX/.的 Keycloak。成功认证后，您将被重定向回 NGINX 欢迎页面。

*Last updated: April 1, 2022*