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

### user 启动用户

```shell
user user [group];
```

* user：制定运行nginx服务器的用户
* group：可选项制定可以运行nginx服务器的用户组

如果想要所有用户都可以启动nginx，则把user注释掉，或者设置为

```shell
user nobody nobody;
```

### worker process  进程数

nginx服务器实现并发处理服务的关键所在，理论上worker process越大支持的并发越大，但实际受限于服务器本身资源

```shell
worker process number|auto;
```

* number：制定nginx进程最多可以产生多少个worker process
* auto：nginx自动检测生成相应的worker process（一般等同于CPU线程数量）

### pid 进程ID

主进程号，编译安装默认在logs目录下  

尽量写绝对路径，如：/opt/nginx/logs/nginx.pid

### error_log 错误日志

全局块、HTTP块和server块都可以对你滚下服务器的日志进行相关配置，语法结构如下：

```shell
error_log  file | stderr  [debug | info | notice | warn | error | crit | alert |emerg];
```

从语法结构上看，nginx服务器错误日志支持输出到固定文件file或者输出到标准错误输出stderr；日志的错误级别可选，（debug、info、notice、warn、error、crit、alert、emerg）。**需要注意：设置某一级别后，比这一级别高的日志也会被记录下来**  

```shell
reeor_log logs/error.log error;
```

日志等级高于error的日志会被记录在logs/error.log下

### include 包含配置文件

用于将其他的nginx配置或者第三方模块的配置引用到当前的主配置文件中。语法为：

```shell
include file;
```

**注意：** 新引用进来的文件同样要求运行nginx进程的用户对其具有写权限，并且符合nginx配置文件规定的相关语法和结构  

该配置命令可以放在配置文件的任意地方

### accept_mutex 网络连接序列化

惊群问题：当某一时刻只有一个网络连接到来时，多个睡眠进程会被同时叫醒，但只有一个进程可获得连接，如果每次还行的进程数目太多，会影响一部分系统性能。  

为了解决这样的问题，nginx配置中包含了accept_mutex，当accept_mutex开启时，将对多个nginx进程接收连接进行序列化，防止多个进程对连接的争抢，其语法结构为：

```shell
accept_mutex on | off ; 
```

该指令默认开启，只能在events块中进行配置

### multi_accept 多网络连接

nginx服务器的worker process是否同时接收多个新到达的网络连接，由multi_accept控制：

```shell
multi_accept on | off ;
```

### use 事件驱动模型

nginx服务器提供多宗事件驱动模型来处理网络消息，指令use:

```shell
use method;
```

其中，method可选择的内容：select、poll、kqueue、epoll、rtsig、/dev/poll以及eventport

**注意：** 可以在编译是使用`--with-select_module`和`--without-select_module`设置是否强制编译select模块到nginx内核；使用`--with-poll_module`和`--without-poll_module`设置是否强制编译poll模块到nginx内核。  

此指令只能在events块中进行配置。

### work_connections 最大连接数

每一个worker process同时开启的最大连接数

```shell
worker_connections number;
```

默认值512

**注意：这里的number不仅仅包括和前端用户简历的连接数，而是包括所有可能的连接数。另外，number值不能大于操作系统支持打开的最大我呢见句柄数量**

只能在events块中进行配置

### MIME-Type 文件类型

MIME-Type是网络子u按的媒体类型，包括HTML，XML，GIF，及Flash等文本、媒体资源

```
include mine.types;
default_type application/octet-stram;
```

```shell
[root@test nginx]# cat conf/mime.types

types {
    text/html                                        html htm shtml;
...

    text/mathml                                      mml;
...

    image/png                                        png;
...
    image/x-ms-bmp                                   bmp;

    application/font-woff                            woff;
    application/java-archive                         jar war ear;
...
    application/vnd.openxmlformats-officedocument.presentationml.presentation
                                                     pptx;
    application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
                                                     xlsx;
    application/vnd.openxmlformats-officedocument.wordprocessingml.document
                                                     docx;
...

    application/octet-stream                         bin exe dll;
    application/octet-stream                         deb;
...

    audio/midi                                       mid midi kar;
    audio/mpeg                                       mp3;
...
    video/3gpp                                       3gpp 3gp;
    video/mp2t                                       ts;
    video/mp4                                        mp4;
...
}
```

type中包含了浏览器能够识别的MIME类型以及对应相应类型的文件啊后缀名，由于mime——types文件是猪配置文件应用的第三方文件  

```shell
default——type mime-type;
```

### access_log 服务日志

```shell
access_log path [format [buffer=size]]
```

* path: 日志存放的路径和名称
* format： 可选项，自定义服务日志的格式字符串
* size： 配置临时存放日志的内存缓存区大小

access_log可以在HTTP、server、或者location块进行设置，默认的配置为：

```shell
access_log logs/access.log combined;
```

combined 为log_format指令默认定义的日志格式字符串的名称

#### 取消日志

```shell
access_log off;	
```

### log_format 服务日志格式

只能在HTTP块中进行配置

* name 格式民称，默认为combined
* string 服务日志的格式字符串，可以使用nginx预设的变量获取相关内容，变量的名称使用双引号，string整体使用单引号括起来。

```shell
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```

###  sendfile  文件传输模式

sendfile可以放nginx在传输文件是直接在磁盘和TCP socket之间传输数据，如果这个参数不开启，会先在用户空间(nginx进程空间)申请一个buffer，用read函数把数据从磁盘读到cache，再从cache读取到用户空间的buffer，再用writer函数把数据从用户空间的buffer写入到内核的buffer，最后到TCP socket，开启这个参数可以让数据不用经过用户buffer。

```shell
sendfile on | off ;
sendfile_max_chunk size;
```

* sendfile默认为off，可以在HTTP、server、location中进行配置

* sendfile_max_chunk默认值为0，可以在HTTP、server、location中配置

  size值如果大于0，则nginx每个worker process每次调用sendfile传输的数据量最大不能超过这个值；size值设置为0，则无限制。

### keepalive_timeout 连接超时时间

与用户简历会话连接后，nginx服务器可以保持的会话时长，可以配置在server和location中

```shell
keepalive_timeout timeout [header_timeout];
```

* timeout 服务器端对连接的保持时间，默认75S
* header_timeout，可选项，在应答报文头部的keep-alive域设置超时时间：“Keep-Alive：timeout=header_timeout”，保温中的指令可以被Mozilla或者konqueror识别

```shell
# 在服务端保持连接的时间设置为120s，发送给用户端的应答报文头部中Keep-Alive域的超时时间设置为100s
keepalive_timeout 120s 100s;
```

### keepalive_requests 单连接请求数上限

nginx服务端与用户建立会话连接后，用户端通过此连接发送请求。keepalive用于限制用户通过一个连接向nginx服务器发送请求的次数，默认为100

```shell
keepalive_requests number;
```

### listen 网络监听

#### 三种监听模式

##### 监听地址

```shell
listen address [:port] [default_server] [setfib=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [deferred] [accepy_filter=filter] [bind] [ssl];
```

##### 监听端口

```shell
listen port [default_server] [setfib=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [ssl];
```

##### Unix domain socket

```shell
listen unix:path [default_server] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ssl];
```

#### 参数

未完成

123













