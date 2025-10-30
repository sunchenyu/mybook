# 高并发Nginx和Linux系统配置

## Linux系统配置

### 文件打开数

文件打开数的意思是对一个进程，允许同时打开的句柄数是多少。对于网络程序，也就是能够支持的最大并发连接数了。系统默认的值是1024，我们可以通过下面的命令来查看

```
ulimit -a
```

如下图显示的，就是默认的Linux系统的一些信息。

<div align="left"><figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

修改/etc/security/limits.conf文件中的配置信息

```
# End of file
*	hard	nproc	65535
*	soft	nproc	65535
*	hard	nofile	65535
*	soft	nofile	65535
```

改的是nofile这两个配置，hard表示当前可以设置的最大值，soft表示当前的值

可以将这两个值改成500000，因为单机的话，50万的并发连接基本也到nginx的极限了

<mark style="color:red;">注意：nofile的最大值不能超过/proc/sys/fs/nr\_open这个值</mark>

完成后再次执行ulimit -a

<mark style="color:red;">注意：有些服务器因为系统配置原因，可能通过ssh登录时无法看到生效信息，可以通过配置/etc/ssh/sshd\_config中的UsePAM 和 UseLogin参数来。 在文件中修改/添加UsePAM yes和UseLogin yes，然后重新登录ssh即可。</mark>

## TCP连接配置

### 配置内容

打开/etc/sysctl.conf文件

在文件末尾添加如下参数

```
# It enables fast recycling of TIME_WAIT sokcets. Known to cause some issues with hoststated(NAT and load balancing) if enabled, should be used with caution.
net.ipv4.tcp_tw_recycle = 0
# This allows resuing sockets in TIME_WAIT state for new connections when it is safe from protocol viewpoint. It is generally a safer alternative to tcp_tw_recycle
net.ipv4.tcp_tw_reuse = 1
# Determines the time that must elapse before TCP/IP can release a closed connection and reuse its resource. During this TIME_WAIT state, reopening the connection to
# the client costs less then establishing a new connection. By reducing the value of the entry, TCP/IP can release closed connections faster, making more resources
# available for new connections.
net.ipv4.tcp_fin_timeout = 5
# The wait time between isAlive interval probes(seconds)
net.ipv4.tcp_keepalive_intvl = 15
# The number of probes before timing out.
net.ipv4.tcp_keepalive_probes = 3
# The time of keepalive remined time(seconds)
net.ipv4.tcp_keepalive_time = 300
# TCP syn cookies
net.ipv4.tcp_syncookies = 1
# The length of the syn quene
net.ipv4.tcp_max_syn_backlog = 655350
# The length of the tcp accept queue
net.core.somaxconn = 65535
# port range
net.ipv4.ip_local_port_range = 1024 65000
```

然后执行

```
sysctl -p
```

### 配置详解

#### net.ipv4.tcp\_tw\_recycle

net.ipv4.tcp\_tw\_recycle 是 Linux 内核中的一个 TCP 参数，用于控制 TIME\_WAIT 状态的连接回收机制。不过它现在已经 在 Linux 4.12 之后被移除，因为它会导致严重的网络连接问题（尤其是在 NAT 环境下）。

#### net.ipv4.tcp\_tw\_reuse

允许将处于 TIME\_WAIT 状态的套接字重新用于新的连接。

当本机作为 TCP 客户端（主动发起连接的一方） 时，如果开启了此选项，Linux 可以在满足一定条件的情况下，重用 TIME\_WAIT 状态的 socket（源端口） 来发起新连接。

#### net.ipv4.tcp\_fin\_timeout

对于本端断开的socket连接，TCP保持在FIN-WAIT-2状态的时间。

对方可能会断开连接或一直不结束连接或不可预料的进程死亡。默认值为 60 秒。降低这个值可以一定程度上防范DDOS攻击。

#### net.ipv4.tcp\_keepalive\_intvl

两次检测tcp连接活动状态的间隔时间，默认75秒。

TCP 的 Keepalive 是一种空闲连接检测机制。它用于判断长时间无数据交互的连接是否还“活着”。

#### net.ipv4.tcp\_keepalive\_probes

最大多少次连接检测tcp非活动状态后断开连接，默认9次。

#### net.ipv4.tcp\_keepalive\_time

连接空闲多久后开始发送 Keepalive 探测包，默认7200秒。

#### net.core.somaxconn

限制全连接队列（accept 队列）的最大长度上限。

#### net.ipv4.tcp\_max\_syn\_backlog

控制半连接队列（SYN 队列）的最大长度。

#### net.ipv4.tcp\_syncookies

控制是否启用TCP SYN Cookies 机制，用于防御SYN Flood攻击。通过一个加密算法，将必要状态信息（源IP、端口、MSS 等）编码到 TCP 序列号中。当客户端发回 ACK 时，内核从 ACK 序列号中解析出信息。

#### net.ipv4.ip\_local\_port\_range

控制本机可用的临时端口号范围（用于主动连接的源端口）。

## Nginx配置

### worker多线程

```
worker_processes auto; 
worker_cpu_affinity auto; 
worker_rlimit_nofile 409600;
```

启用多线程处理，默认情况下，nginx只有一个master线程来处理，这段配置告诉nginx尽可能多地启用worker线程来帮助处理高并发的任务。&#x20;

### 主配置

```
events {
    use                 epoll;
    worker_connections  409600;
    multi_accept        on;
    accept_mutex        off;
}

http {
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    access_log          /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;

    keepalive_timeout   300;
    keepalive_requests  20000000;

    include /usr/local/nginx/conf.d/*.conf;
}
```

events表示启用epoll模型来处理高并发的请求，epoll模型是linux底层的高性能、高并发处理模型，nginx也是基于此实现的。&#x20;

http的主配置中，开启sendfile、tcp\_nopush和tcp\_nodelay。

keepalive\_timeout表示超时时间、keepalive\_requests表示同一个连接接收多少请求后断开。

