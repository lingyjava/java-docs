# 11·Nginx

- [11·Nginx](#11nginx)
  - [正向代理](#正向代理)
  - [Nginx](#nginx)
  - [配置文件](#配置文件)
  - [Ajax跨域请求](#ajax跨域请求)
  - [session共享](#session共享)

## 正向代理
![图18-正向代理流程](/docs/images/图18-正向代理流程.jpeg)

## Nginx
Nginx是一个实现反向代理的轻量级服务器。代理服务器端。
通常一个项目会同时部署在多个服务器上，那么用户在访问时都会访问一个地址，那么就需要使用反向代理将用户的请求根据负载均衡策略转发到各个服务器上。通过访问反向代理服务器，由反向代理服务器访问真实的服务器。为了实现集群。把所有的请求交给Nginx，Nginx根据内部的策略选择要访问哪个服务器。

## 配置文件
- [nginx配置文件nginx.conf超详细讲解](https://www.cnblogs.com/liang-wei/p/5849771.html)

配置完成后访问的网址应是：http://[配置的ip]:[nginx端口号]/[项目名]/....jsp
```nginx

#user  nobody;
# 进程数,常与cpu核数一致
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
  worker_connections  1024;
}


http {
  include       mime.types;
  default_type  application/octet-stream;

  #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
  #                  '$status $body_bytes_sent "$http_referer" '
  #                  '"$http_user_agent" "$http_x_forwarded_for"';

  #access_log  logs/access.log  main;

  sendfile        on;
  #tcp_nopush     on;

  #keepalive_timeout  0;
  keepalive_timeout  65;

  # 配置集群
  # 此处名字不能带下划线,tomcat9.0开始不支持,运行时出错
  upstream tomcatpool{
    server [ip]:[端口号] weight=5;
    server [ip]:[端口号] weight=5;
  }

  #gzip  on;

  # 一个server相当于一个代理服务器的配置
  server {
    # 反向代理服务器nginx的端口
    listen       8090;
    # 反向代理服务器名,建议用本机ip
    server_name  localhost;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    # 用来过滤请求,一个location就是一个规则
    # 配置nginx代理的真实服务器,以及静态资源路径
    location / {
      root   html;
      index  index.html index.htm;

      # 配置真实服务器(tomcat...)
      # proxy_pass http://[ip]:[端口];
      # 配置集群,设置代理服务器
      # proxy_pass http://tomcat pool;
    }

    # 用来过滤静态请求,一个location就是一个规则
    # 配置nginx代理的真实服务器,以及静态资源路径
    location ~*\.(jpg|css|js|png|ioc)$ {
      root   html;
      index  index.html index.htm;

      # 配置真实服务器(tomcat...)
      # proxy_pass http://[ip]:[端口];
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
      root   html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    # 实现动静分离
    # 实现正则表达式匹配请求路径,如果使用正则表达式必须以~开头
    #location ~ \.php$ {
    # 用来配置真实服务器
    # proxy_pass [ip]:[端口]
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    # 配置静态资源的路径
    # root d:\static
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
  }


  # another virtual host using mix of IP-, name-, and port-based configuration
  #
  #server {
  #    listen       8000;
  #    listen       somename:8080;
  #    server_name  somename  alias  another.alias;

  #    location / {
  #        root   html;
  #        index  index.html index.htm;
  #    }
  #}


  # HTTPS server
  #
  #server {
  #    listen       443 ssl;
  #    server_name  localhost;

  #    ssl_certificate      cert.pem;
  #    ssl_certificate_key  cert.key;

  #    ssl_session_cache    shared:SSL:1m;
  #    ssl_session_timeout  5m;

  #    ssl_ciphers  HIGH:!aNULL:!MD5;
  #    ssl_prefer_server_ciphers  on;

  #    location / {
  #        root   html;
  #        index  index.html index.htm;
  #    }
  #}

}
```

## Ajax跨域请求
当使用Nginx反向代理后,可能会出现Aajax请求失败。这是因为Ajax不能跨域请求（服务器名或端口号不一致的ajax请求）。可以将项目都放在Nginx上，所有请求的地址都是Nginx地址，欺骗浏览器，实现跨域请求。
也可以将Jsp模板改为：
```
<%
		//设置项目根路径
    String basePath = http://[本机IP]:[反向代理服务器端口]/项目名;
%>
<head>
    <base href="<%=basePath%>">
</head>
```

## session共享
使用nginx代理服务器集群后，可能导致session作用域的数据取不到。
解决方法（通过redis数据库）：

1. 把session共享的jar包放到tomcat的lib目录下
2. 在tomcat中配置
   1. 导入session共享jar到tomcat里面
   2. 在tomcat的context.xml里面配置redis相关的信息
```xml
<Valve className="com.radiadesign.catalina.session.RedisSessionHandlerValve" />  
<Manager className="com.radiadesign.catalina.session.RedisSessionManager"  
             host="10.10.10.154"  
             port="6379"  
             database="0"  
             maxInactiveInterval="60" />
```

3. 放到session中的实体类必须实现可序列化接口。

结果：tomcat自动将session域的数据存到redis数据库，再从redis取数据。
