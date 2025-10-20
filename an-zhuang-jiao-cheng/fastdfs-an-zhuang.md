# fastdfs安装

## 前置条件

**安装环境：**&#x672C;人安装环境：centos7 **环境依赖：**&#x6240;有服务器都需要

```
yum install git gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel wget vim -y
```

**新建目录：**&#x6240;有服务器都需要（我这里将存储路径配置成了 /home/dfs）

```
mkdir /home/dfs
```

所有的IP要根据自己的服务器的IP去配置。 我测试是没问题的，如果安装步骤遇到问题，请及时联系我，进行及时更正，以免影响更多人。

## 一、单机搭建

### 1、下载源码，编译安装

（1）安装libfastcommon

```
cd /usr/local/src #切换到安装目录准备下载安装包
git clone https://github.com/happyfish100/libfastcommon.git --depth 1 #个人多次下载不成功，多试几次就成功了
cd libfastcommon/
./make.sh && ./make.sh install #编译安装
```

（2）安装FastDFS

```
cd /usr/local/src #切换到安装目录准备下载安装包
git clone https://github.com/happyfish100/fastdfs.git --depth 1
cd fastdfs/
./make.sh && ./make.sh install #编译安装
```

### 2、配置文件准备

根据情况将没有的文件拷贝到/etc/fdfs/目录下

```
cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf #客户端文件，测试用
cp /usr/local/src/fastdfs/conf/http.conf /etc/fdfs/ #供nginx访问使用
cp /usr/local/src/fastdfs/conf/mime.types /etc/fdfs/ #供nginx访问使用
```

### 3、安装fastdfs-nginx-module

```
cd /usr/local/src
git clone https://github.com/happyfish100/fastdfs-nginx-module.git --depth 1
cp /usr/local/src/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs
```

### 4、源码安装nginx

```
cd /usr/local/src
wget http://nginx.org/download/nginx-1.15.4.tar.gz #下载nginx压缩包
tar -zxvf nginx-1.15.4.tar.gz #解压
cd nginx-1.15.4/
# nginx安装到了/usr/local/nginx  添加fastdfs-nginx-module模块
./configure --prefix=/usr/local/nginx --add-module=/usr/local/src/fastdfs-nginx-module/src/ 
make && make install #编译安装
```

### 5、修改配置文件

（1）tracker vim /etc/fdfs/tracker.conf

```
# 需要修改的内容如下
port=22122  # tracker服务器端口（默认22122,一般不修改）
base_path=/home/dfs  # 存储日志和数据的根目录
```

（2）storage vim /etc/fdfs/storage.conf

```
#需要修改的内容如下
port=23000  # storage服务端口（默认23000,一般不修改）
base_path=/home/dfs  # 数据和日志文件存储根目录
store_path0=/home/dfs  # 第一个存储目录
tracker_server=192.168.198.137:22122  # tracker服务器IP和端口
http.server_port=8888  # http访问文件的端口(默认8888,看情况修改,nginx通过这个端口访问文件)
```

### 6、启动

执行启动命令：

```
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```

关闭命令（这个地方不需要执行下面这两条命令）：

```
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf stop
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf stop
```

### 7、启动之后修改client配置文件并进行上传测试

vim /etc/fdfs/client.conf

```
#需要修改的内容如下
base_path=/home/dfs
tracker_server=192.168.198.137:22122    #tracker服务器IP和端口
```

修改完client配置文件之后，进行上传文件的测试 执行如下命令，/app/16193421396964441.jpg需要你自己找一张文件代替

```
#保存后测试,返回ID表示成功 如：group1/M00/00/00/xx.tar.gz
fdfs_upload_file /etc/fdfs/client.conf  /app/16193421396964441.jpg #后面这个jpg表示上传到fastdfs的文件
```

执行结果应该是如下这个样子：&#x20;

![执行结果](<../.gitbook/assets/image (39).png>)

### 8、修改mod\_fastdfs.conf

vim /etc/fdfs/mod\_fastdfs.conf

```
base_path=/home/dfs
tracker_server=192.168.198.137:22122  # tracker服务器IP和端口
url_have_group_name=true   #url中包含group名称
```

### 9、之后通过nginx访问

修改nginx配置文件：执行命令vim /usr/local/nginx/conf/nginx.conf

```
 server {
        listen       8888;    ## 该端口为storage.conf中的http.server_port相同
        server_name  localhost;
        location ~/group[0-9]/ {
            ngx_fastdfs_module;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
        root   html;
        }
    }
```

启动nginx并访问

```
/usr/local/nginx/sbin/nginx
```

通过浏览器访问[http://192.168.198.137:8888/xxxx。](http://192.168.198.137:8888/xxxx%E3%80%82)

xxxx即为上传fastdfs返回的结果字符串： 例如： [http://192.168.198.137:8888/group1/M00/00/00/wKjGimCaS1WAdjcxAAUqTf4vr2g782.jpg](http://192.168.198.137:8888/group1/M00/00/00/wKjGimCaS1WAdjcxAAUqTf4vr2g782.jpg)

## 二、集群搭建

**环境描述：**&#x4E24;个trace，两个storage。storage都是group1。&#x20;

**特点：**&#x56E0;为这两个storage的group1一样，所以有两份相同的数据，此时这两个storage可以当做集群

### 1、按照单机步骤，将所有的软件安装好

即上面1、2、3、4步，将软件都安装好后，只需要修改配置文件，并启动即可。 唯一不同的就是修改的配置。

### 2、修改配置

**（1）tracker配置（这里是两个） 有几个就改几个** 第一个tracker 修改/etc/fdfs/tracker.conf（服务器IP：192.168.198.137）

```
port=22122  # tracker服务器端口（默认22122,一般不修改）
base_path=/home/dfs  # 存储日志和数据的根目录
```

第二个tracker 修改/etc/fdfs/tracker.conf（服务器IP：192.168.198.138）

```
port=22122  # tracker服务器端口（默认22122,一般不修改）
base_path=/home/dfs  # 存储日志和数据的根目录
```

**（2）storage （这里是两个，这两个配置一样）** 第一个storage\
修改/etc/fdfs/storage.conf（服务器IP：192.168.198.137）

```
base_path=/home/dfs  # 数据和日志文件存储根目录
store_path0=/home/dfs  # 第一个存储目录
tracker_server=192.168.198.137:22122  # tracker服务器IP和端口
tracker_server=192.168.198.138:22122  # tracker服务器IP和端口
```

第二个storage 修改/etc/fdfs/storage.conf（服务器IP：192.168.198.138）

```
base_path=/home/dfs  # 数据和日志文件存储根目录
store_path0=/home/dfs  # 第一个存储目录
tracker_server=192.168.198.137:22122  # tracker服务器IP和端口
tracker_server=192.168.198.138:22122  # tracker服务器IP和端口
```

### 3、启动

```
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```

### 4、查看集群存储信息

```
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
```

### 5、修改client配置（所有的都要修改）

vim /etc/fdfs/client.conf

```
base_path=/home/dfs
tracker_server=192.168.198.137:22122    #tracker服务器IP和端口
tracker_server=192.168.198.138:22122    #tracker服务器IP和端口
```

### 6、修改mod\_fastdfs.conf（所有的都要修改）

vim /etc/fdfs/mod\_fastdfs.conf

```
base_path=/home/dfs
tracker_server=192.168.198.137:22122  # tracker服务器IP和端口
tracker_server=192.168.198.138:22122    #tracker服务器IP和端口
url_have_group_name=true   #url中包含group名称
```

### 7、修改nginx 配置（每个storage都需要一个这个配置）

```
server {
    listen       8888;    ## 该端口为storage.conf中的http.server_port相同
    server_name  localhost;
    location ~/group[0-9]/ {
        ngx_fastdfs_module;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    root   html;
    }
}
```

### 8、添加最外层nginx配置（对外提供服务）（配置到192.168.198.138上了）

集群模式下，每个nginx只能代理到本服务器上的那些文件，如果需要对外统一的话，需要一个最外层nginx，因为只有一个group1，所有只要配置一个代理即可。

```
upstream fdfs_group01 {
    server 192.168.198.137:8888 weight=1 max_fails=2 fail_timeout=30s;
    server 127.0.0.1:8888 weight=1 max_fails=2 fail_timeout=30s;
}
 server {
    listen       90;    ## 该端口为storage.conf中的http.server_port相同
    server_name  localhost;

    location /group1{
        proxy_next_upstream http_502 http_504 error timeout invalid_header;
        proxy_pass http://fdfs_group01;
        expires 30d;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    root   html;
    }
}
```

## 三、多group集群搭建

**环境描述：**&#x4E24;个trace，两个storage。一个storage只有group1，一个storage只有group2。 多个storage可以都配置group1，这种相当于有多份文件的副本。

### 1、先按照单机，将所有的软件安装好

即上面1、2、3、4步，将软件都安装好后，只需要修改配置文件，并启动即可 唯一不同的就是修改的配置

### 2、修改配置

**（1）tracker配置（这里是两个） 有几个就改几个** 第一个tracker 修改/etc/fdfs/tracker.conf（服务器IP：192.168.198.137）

```
port=22122  # tracker服务器端口（默认22122,一般不修改）
base_path=/home/dfs  # 存储日志和数据的根目录
```

第二个tracker 修改/etc/fdfs/tracker.conf（服务器IP：192.168.198.138）

```
port=22122  # tracker服务器端口（默认22122,一般不修改）
base_path=/home/dfs  # 存储日志和数据的根目录
```

**（2）storage （这里是两个，这两个配置不一样）** 第一个storage\
修改/etc/fdfs/storage.conf（服务器IP：192.168.198.137）

```
group_name=group1
base_path=/home/dfs
tracker_server=192.168.198.137:22122  # tracker服务器IP和端口
tracker_server=192.168.198.138:22122    #tracker服务器IP和端口
url_have_group_name=true   #url中包含group名称
```

第二个storage 修改/etc/fdfs/storage.conf（服务器IP：192.168.198.138）

```
group_name=group2
base_path=/home/dfs
tracker_server=192.168.198.137:22122  # tracker服务器IP和端口
tracker_server=192.168.198.138:22122    #tracker服务器IP和端口
url_have_group_name=true   #url中包含group名称
```

### 3、启动

```
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```

### 4、查看集群存储信息

```
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
```

### 5、修改client配置

```
base_path=/home/dfs
tracker_server=192.168.198.137:22122    #tracker服务器IP和端口
tracker_server=192.168.198.138:22122    #tracker服务器IP和端口
```

### 6、修改mod\_fastdfs.conf

由于group不同了，所以需要指定一下group\_name的名字 storage1

```
group_name=group1
base_path=/home/dfs
tracker_server=192.168.198.137:22122  # tracker服务器IP和端口
tracker_server=192.168.198.138:22122    #tracker服务器IP和端口
url_have_group_name=true   #url中包含group名称
```

storage2

```
group_name=group2
base_path=/home/dfs
tracker_server=192.168.198.137:22122  # tracker服务器IP和端口
tracker_server=192.168.198.138:22122    #tracker服务器IP和端口
url_have_group_name=true   #url中包含group名称
```

### 7、对外服务的nginx的配置

group1有一组负载、group2有一组负载、每个group都需要配置一下这个负载信息。

```
# 如果group1有多个storage，只要配置这个地方即可
upstream fdfs_group01 {
    server 192.168.198.137:8888 weight=1 max_fails=2 fail_timeout=30s;
   # server 127.0.0.1:8888 weight=1 max_fails=2 fail_timeout=30s;
}

upstream fdfs_group02 {
    #server 192.168.198.137:8888 weight=1 max_fails=2 fail_timeout=30s;
    server 127.0.0.1:8888 weight=1 max_fails=2 fail_timeout=30s;
}

 server {
    listen       90;    ## 该端口为storage.conf中的http.server_port相同
    server_name  localhost;


    location /group1{
        proxy_next_upstream http_502 http_504 error timeout invalid_header;
        proxy_pass http://fdfs_group01;
        expires 30d;
    }

    location /group2{
        proxy_next_upstream http_502 http_504 error timeout invalid_header;
        proxy_pass http://fdfs_group02;
        expires 30d;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    root   html;
    }
}
```

### 8、验证多group的fastdfs

上传文件

```
[root@bogon nginx]# fdfs_upload_file /etc/fdfs/client.conf /app/16193421396964441.jpg 
group2/M00/00/00/wKjGimCaJkiAY2J4AAHs2ptsX3Y051.jpg
```

通过url [http://192.168.198.138:90/group2/M00/00/00/wKjGimCaJkiAY2J4AAHs2ptsX3Y051.jpg](http://192.168.198.138:90/group2/M00/00/00/wKjGimCaJkiAY2J4AAHs2ptsX3Y051.jpg) 查看图片&#x20;

![查看图片](<../.gitbook/assets/image (11).png>)

关闭storage2 执行命令

```
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf stop
```

![关闭结果](<../.gitbook/assets/image (44).png>)

&#x20;再次上传文件

```
[root@bogon nginx]# fdfs_upload_file /etc/fdfs/client.conf /app/16193421396964441.jpg  
group1/M00/00/00/wKjGiWCaPO6ADLO0AAHs2ptsX3Y680.jpg
```

发现这个时候文件上传到了group1上

## 四、总结

基本上就是把软件安装好之后，只需要改一下tracker.conf、storage.conf、client.conf、mod\_fastdfs.conf配置文件，重新启动即可。

## 五、参考文章

[https://github.com/happyfish100/fastdfs/wiki](https://github.com/happyfish100/fastdfs/wiki) [https://github.com/happyfish100/fastdfs/blob/master/INSTALL](https://github.com/happyfish100/fastdfs/blob/master/INSTALL) [https://blog.csdn.net/ko0491/article/details/108888445](https://blog.csdn.net/ko0491/article/details/108888445)
