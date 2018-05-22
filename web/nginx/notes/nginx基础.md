# nginx

nginx官网：http://www.nginx.org  
nginx wiki：https://www.nginx.com/resources/wiki/

## nginx配置文件

### nginx.conf示例

```shell
# 全局块：默认配置文件到event块之间的内容，主要设置一些影响nginx服务器整体运行的配置命令，包括nginx的用户，用户组，允许生成的worker process数，nginx进程PID存放路径，日志存放路径和类型、配置文件引入
user  nobody;	# 用户nobody
worker_processes  1;	# 

error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;

pid        logs/nginx.pid;

# events块：events块涉及的指令主要影响nginx服务器与用户的网络连接，常用到的设置包括是否开启对多worker process下的网络连接进行序列化，是否允许同事接受多个网络连接，怄那种时间驱动模型处理连接请求，每个worker process可以同事支持的最大连接数等
events {
    worker_connections  1024;
}

# http块是nginx服务器配置中的重要部分，代理、缓存和日志定义等绝大多数的功能和第三方模块的配置都可以放在这个模块中。
# http块中可以包含自己的全局快，也可以包含server块，server块中又可以金亦波包含location块，在本书中我们使用"http全局块"来表示http中自己的全局块，即http块中不包含在server块中的部分
# http全局块中可以包含文件引入、MIME-Type定义、日志自定义、是否使用sendfile传输文件、连接超时时间、单连接请求数上限等
http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    sendfile        on;
    tcp_nopush     on;

    keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;
# server块与虚拟主机的概念有密切联系。
# 虚拟主机，又称虚拟服务器、主机空间或者网页空间，他是一种技术，为了节互联网服务器硬件成本而出现。将服务内容逻辑划分为多个服务单位，对外表现为多个服务器，充分利用服务器硬件资源
# server块可以包含多个server块，而每个server块就相当于一台虚拟主机，他内部可有多台主机联合提供服务
    server {
        listen       80;
        server_name  localhost;

        charset koi8-r;

        access_log  logs/host.access.log  main;
# 每个server块可以包含多个location块，主要作用于给予nginx服务器接收到的请求字符串对除虚拟主机名称之外的字符串进行匹配，对特定的请求进行处理，地址定向、数据缓存和应答控制等功能都是在这部分实现，许多第三方模块的配置也是在location块中提供功能
        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page  404              /404.html;

         redirect server error pages to the static page /50x.html
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

         proxy the PHP scripts to Apache listening on 127.0.0.1:80
        
        location ~ \.php$ {
            proxy_pass   http://127.0.0.1;
        }

         pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            include        fastcgi_params;
        }

         deny access to .htaccess files, if Apache's document root
         concurs with nginx's one
        
        location ~ /\.ht {
            deny  all;
        }
    }


     another virtual host using mix of IP-, name-, and port-based configuration
    
    server {
        listen       8000;
        listen       somename:8080;
        server_name  somename  alias  another.alias;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }


     HTTPS server
    
    server {
        listen       443 ssl;
        server_name  localhost;

        ssl_certificate      cert.pem;
        ssl_certificate_key  cert.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }

}
```

### user

```shell
user user [group];
```

* user：制定运行nginx服务器的用户
* group：可选项制定可以运行nginx服务器的用户组

如果想要所有用户都可以启动nginx，则把user注释掉，或者设置为

```shell
user nobody nobody;
```

### worker process

nginx服务器实现并发处理服务的关键所在，理论上worker process越大支持的并发越大，但实际受限于服务器本身资源

```shell
worker process number|auto;
```

* number：制定nginx进程最多可以产生多少个worker process
* auto：nginx自动检测生成相应的worker process（一般等同于CPU线程数量）

### pid

主进程号，编译安装默认在logs目录下  

尽量写绝对路径，如：/opt/nginx/logs/nginx.pid

### error_log

全局块、HTTP块和server块都可以对你滚下服务器的日志进行相关配置，语法结构如下：

```shell
error_log  file | stderr  [debug | info | notice | warn | error | crit | alert |emerg];
```

从语法结构上看，nginx服务器错误日志支持输出到固定文件file或者输出到标准错误输出stderr；日志的错误级别可选，（debug、info、notice、warn、error、crit、alert、emerg）。**需要注意：设置某一级别后，比这一级别高的日志也会被记录下来**  

```shell
reeor_log logs/error.log error;
```

日志等级高于error的日志会被记录在logs/error.log下



