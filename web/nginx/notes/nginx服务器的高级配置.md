# nginx服务器的高级配置

```shell
/etc/sysctl.conf
```



## 针对IPv4的内核优化

### net.core.netdev_max_backlog 队列数据包最大数目

表示每个网络接口接收数据报的速率比内核处理这些包的速率快时，**允许发送到队列的数据包的最大数目**

系统默认值128，NGINX服务器中定义的NGX_LISTEN_BACKLOG默认为511，需要调整参数

```shell
net.core.netdev_max_backlog = 262144
```

### net.core.somaxconn 系统同时发起的TCP连接数

默认值128，可能导致链接超时或者重传问题，可以根据实际需要结合并发请求书来调节

```shell
net.core.somaxconn = 262144
```



