# Rewrite

## Upstream命令-负载

### upstream

```shell
upstream name {...} ; 
```

name为服务器组的组名（别名），花括号中可以列出后端服务器组中包含的服务器

默认轮询（Round-Robin），轮流处理请求，如果某个服务器处理时出现错误，则切换下一个服务器，直到所有服务器报错，返回报错内容

### server

```shell
server address [patameters];
```

* address，服务器地址，可以是IP地址，主机名，域名，或者unix Domain Socket 
* parameters
  * weight=number，服务器权重，权重值高的优先处理请求
  * max_fails=number，设置请求失败次数，一定时间范围，服务器请求失败的次数搞过限定值，认为服务器down，默认值为1，设置为0则失效
  * fail_timeout，设置尝试请求某服务器的时间，即max_fails的事件范围。也用于检查服务器时候有效，如果一台服务器down，则该时间段内不会再次检查服务器状态；默认10Ss
  * backup,标记某个服务器为备用服务器，只有其他服务器down或者busy时才处理客户端请求
  * down，永久标记服务器为无效，通常与ip_hash配合使用

配置示例：

```shell
upstream backend
{
    server backend1.example.com weight=5;
    server 127.0.0.1:8080 max_fails=3 fail_timeout=30s;
    server unix:/tmp/backend3;    
}
```

### ip_hash

基于IP地址将客户端的请求定向到同一台服务器，保证客户端与服务器间会话稳定

局限：

* 不能用server中的weight变量一起使用
* nginx服务器必须是最前端的服务器，才能获取到客户端真实IP
* 客户端IP地址必须是C类地址

```shell
upstream backend {
    ip_hash;
    server myback1.proxy.com;
    server myback2.proxy.com;
}
```

### keepalive

```shell
keepalive connections;
```

connections为nginx服务器每个worker process允许该服务器组保持的空闲网络连接数上线，超过后哦，将采用最近最少使用的策略关闭网络连接

### least_conn

配置nginx服务器使用负载均衡策，选择最少连接负载的服务器分配连接，如果有多台符合的服务器，则采用加权轮询选择这几台服务器

## Rewrite

rewrite依赖PCRE（peal compatible regular expressions）

### 地址重写 & 地址转发

* 地址重写：实现地址标准化，如google.com，www,google.com，google.cn，都跳向google首页
* 地址转发：将一个域名知道另一个以后站点的过程

区别：

* 地址转发后客户端地址显示不变，地址红鞋后客户端浏览器地址栏中的地址改变为服务器选择确定的地址
* 地址转发过程只产生一次网络请求；地址重写一般会产生两次请求
* 地址转发一般发生在同一站点项目内，地址重写没有该限制
* 地址转发的页面可以不用全路径表示，但地址重写到的页面必须使用完整的路径名表示
* 地址转发过程中，可以将客户端请求的request传递给新的页面，地址重写不可以
* 地址转发的速度较地址重定向快

### if

根据条件判断结果选择不同的nginx配置，可以在server块或者location块中配置该指令

```shell
if （condition） {...}
```

