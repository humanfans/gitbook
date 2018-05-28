# nginx服务器的高级配置

```shell
/etc/sysctl.conf
```



## 针对IPv4的内核优化

### net.core.netdev_max_backlog 

队列数据包最大数目，表示每个网络接口接收数据报的速率比内核处理这些包的速率快时，**允许发送到队列的数据包的最大数目**

系统默认值128，NGINX服务器中定义的NGX_LISTEN_BACKLOG默认为511，需要调整参数

```shell
net.core.netdev_max_backlog = 262144
```

### net.core.somaxconn 

系统同时发起的TCP连接数，默认值128，可能导致链接超时或者重传问题，可以根据实际需要结合并发请求书来调节

```shell
net.core.somaxconn = 262144
```

### net.ipv4.tcp_max_orphans 

设置系统中最多允许存在多少TCP套接字不被关联到任何一个用户文件句柄，超过这个数字，米有与用户文件句柄关联的TCP套接字将被立即复位，同时给出警告信息

```shell
net.ipv4.tcp_max_orphans = 262144
```

### net.ipv4.tcp_max_syn_backlog

未收到客户端确认信息的连接请求的最大值。大于128MB内存的系统默认值为1024，小内存则是128。可以增大该参数：

```shell
net.ipv4.tcp_max_syn_backlog = 262144
```

### net.ipv4.tcp_timestamps

用于设置时间戳，避免序列号的卷绕。赋值为0时表示禁止TCP时间戳的支持。

```shell
net.ipv4.tcp_timestamps = 0
```

### net.ipv4.tcp_synack_retries

用于内核放弃TCP连接之前向客户端发送SYN+ACK包的数量。服务器与客户端要通过三次握手建立连接，在第二次握手期间，内核需要发送SYN并附带一个回应前一个SYN的ACK，主要影响这里，建议赋值为1，即内核放弃连接之前发送一次SYN+ACK包：

```shell
net.ipv4.tcp_synack_retries = 1
```

### net.ipv4.tcp_syn_retries

与net.ipv4.tcp_synack_retries类似，设置内核放弃建立连接之前发送SYN包的数量，建议1

```shell 
net.ipv4.tcp_syn_retries = 1
```



