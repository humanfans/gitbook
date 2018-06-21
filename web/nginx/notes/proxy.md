# proxy

## 反向代理常用命令

### proxy_pass 

```nginx
server {
    location / {
        proxy_pass http://1.1.1.1:8080;
    }
}
```

#### upstream时关于http协议的写法

proxy_pass带协议，则upstream不带协议

```nginx
upstream api {
    server 1.1.1.1:8080；
    server 2.2.2.2:8080;
}

server {
    location / {
        proxy_pass http://api;
    }
}
```

upstream带协议，则proxy_pass不带协议

```nginx
upstream api {
    server http://1.1.1.1:8080；
    server http://2.2.2.2:8080;
}

server {
    location / {
        proxy_pass api;
    }
}
```

#### proxy传递URI注意事项

想要改变URI，则在proxy中配置URI

不想改变URI，则不再proxy中配置URI

```nginx
server {
    server_name server
    location / {
        proxy_pass http://proxy; # proxy_pass不带URI，则传递原本请求的URI,即访问server/aaa代理为proxy/aaa
    }
    location /aaa {
        proxy_pass http://proxy; # proxy_pass不带URI，则传递URI，即访问server/aaa代理为proxy/aaa
    }
    location /aaa {
        proxy_pass http://proxy/bbb; # proxy_pass带URI，则替换原有URI，访问server/*全部代理为proxy/bbb
    }
    location /ccc {
        proxy_pass http://proxy/; # proxu_pass带URL，则替换原有URI，访问server/*全部代理为proxy/;
    }
}
```

### proxy_hide_header

隐藏头域信息

```nginx
proxy_hide_header field;
```

### proxy_pass_header

发送部分头域信息，默认情况下nginx服务器头部不包含date，server，x-accel等来自被**代理服务器的头域信息**

```nginx 
proxy_pass_header field ; 
```

### proxy_pass_request_body

用于配置是否将客户端的请求体（请求的参数）发送给代理服务器，默认开启

```nginx 
proxy_pass_request_body on | off ; 
```

### proxy_pass_request_headers

是否将客户端请求头发送给代理服务器，默认on

```nginx
proxy_pass_request_headers on | off ; 
```

### proxy_set_header 

更改客户端的请求头，将新的请求头发送给被代理的服务器

```nginx
proxy_set_header field value ; 
```

```nginx 
proxy_set_header Host $proxy_host ; 
proxy_set_header Connection close ;
proxy_set_header Host $http_host ;
proxy_set_header Host $host:$proxy_port ; 
```

### proxy_set_body

修改客户端的请求体，并发送给代理服务器

```nginx
proxy_set_body value ; 
```

### proxy_bind  

在配置了多个被代理服务器时，指定某一台服务器处理请求

```nginx
proxy_bind address ; 
```

### proxy_connect_timeout 

nginx与被代理服务器尝试建立连接的超时时间，默认60s

```nginx
proxy_connect_timeout time ; 
```

### proxy_read_timeout

nginx向后端服务器发送read请求后等待响应的事件，默认60s

```nginx
proxy_read_timeout time ; 
```

### proxy_send_timeout

nginx向后端服务器发送write请求后等待响应的超时时间

```nginx
proxy_send_timeout time ; 
```

### proxy_http_version 

指定nginx服务器提供代理服务的http协议版本，默认1.0，1.1版本支持upstream服务器组设置中的Keepalive指令

```nginx
proxy_http_version 1.0 | 1.1 ;
```

### proxy_method 

用于设置nginx服务器请求被代理服务器时的请求方法，一般为POST或者GET，设置了该命令后，客户端的请求方法就被忽略了

```nginx 
proxy_method method ； 
```

### proxy_ignore_client_abort  

忽略客户端中断，用于设置在客户端中断网络请求时，nginx服务器是否中断对后端服务器的请求，默认off

```nginx
proxy_ignore_client_abort on | off ; 
```

### proxy_ignore_headers

是否忽略头域信息，field为忽略的头域字段 

```nginx
proxy_ignore_headers field ...;
```

### proxy_redirect

用于修改被代理服务器返回的响应头中的location和refresh

