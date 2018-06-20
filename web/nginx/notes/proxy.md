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

### proxy传递URI注意事项

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
        proxy_pass http://proxy/; #proxu_pass带URL，则替换原有URI，访问server/*全部代理为proxy/;
    }
}
```



