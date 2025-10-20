# rabbitmq单机安装

##

## 一、rabbitmq单机安装

前置环境安装

```
yum -y install gcc
yum -y install ncurses-devel
yum -y install openssl
yum -y install openssl-devel
```

### 1、下载erlang

erlang官网[https://www.erlang.org/downloads](https://www.erlang.org/downloads)&#x20;

下载src安装包 otp\_src\_24.0.tar.gz

### 2、解压并安装erlang

```
tar -zxvf otp_src_24.0.tar.gz
cd otp_src_24.0
./configure --prefix=/usr/local/erlang
```

**如果报错：**

```
configure: error: in `/root/msmtp-1.4.20':
configure: error: no acceptable C compiler found in $PATH
```

需要安装gcc

```
yum -y install gcc
```

**如果报错：**

```
configure: error: No curses library functions found
ERROR: /app/otp_src_24.0/erts/configure failed!
```

需要安装ncurses-devel

```
yum -y install ncurses-devel
```

**安装erlang**

ssl主要是在使用rabbitmq管理页面时需要

```
./configure --prefix=/usr/local/erlang ---with-ssl
make && make install
```

### 3、配置环境变量

vim /etc/profile 在末尾添加

```
ERLANG_HOME=/usr/local/erlang
export PATH=$PATH:$ERLANG_HOME/bin
export ERLANG_HOME
```

### 4、下载rabbitmq

github网站 [https://github.com/rabbitmq/rabbitmq-server/releases](https://github.com/rabbitmq/rabbitmq-server/releases)&#x20;

下载 rabbitmq-server-generic-unix-3.7.5.tar.xz 这种文件

### 5、解压并安装rabbitmq

```
xz -d rabbitmq-server-generic-unix-3.7.5.tar.xz
tar -xvf rabbitmq-server-generic-unix-3.8.16.tar -C /app/
mv rabbitmq_server-3.8.16 /usr/local/rabbitmq
```

### 6、配置环境变量

vim /etc/profile 在末尾添加

```
export PATH=$PATH:/usr/local/rabbitmq/sbin
export RABBITMQ_HOME=/usr/local/rabbitmq
```

### 7、启动rabbitmq

```
rabbitmq-server -detached
rabbitmqctl status
```

开启管理插件，管理平台端口是15672

```
rabbitmq-plugins enable rabbitmq_management
```

默认账号密码

```
guest
guest
```

默认管理平台账户只允许本地访问，添加添加远程访问用户

```
# root权限
 //添加用户，后面两个参数分别是用户名和密码
rabbitmqctl add_user root 123456

//添加权限
rabbitmqctl set_permissions -p / root ".*" ".*" ".*"  

//修改用户角色,将用户设为管理员rabbitmqctl add_user root 123456
rabbitmqctl set_user_tags root administrator
```

之后使用新的账户登陆管理平台，登陆页面示例如下：&#x20;

![](<../.gitbook/assets/image (14).png>)

### 8、总结

在rabbitmq管理页面上发现的一句很重要的话

The default exchange is implicitly bound to every queue, with a routing key equal to the queue name. It is not possible to explicitly bind to, or unbind from the default exchange. It also cannot be deleted.

翻译过来就是：

默认交换机隐式地绑定到每个队列，路由键等于队列名。不可能显式绑定到默认交换，也不可能取消绑定到默认交换。也不能删除。

