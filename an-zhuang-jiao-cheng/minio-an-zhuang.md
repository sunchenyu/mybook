# Minio安装

## 一、前言

1. Minio 存储目录必须是独立的分区，也就是该分区没有其它程序使用。
2. Minio 采用 纠删码来防范多个节点宕机和位衰减 bit rot。分布式 Minio 至少需要 4 个硬盘，使用分布式 Minio 自动引入了纠删码功能。
3. 单机 Minio 服务存在单点故障，相反，如果是一个有 N 块硬盘的分布式 Minio,只要有 N/2 硬盘在线，你的数据就是安全的。不过你需要至少有 N/2+1 个硬盘来创建新的对象。
4. 高可用最低配置：
   1. 4节点+4块盘：一个节点上挂载一块盘。
   2. 2节点+4块盘：一个节点上挂载两块盘。
5. 文档部署的方式采用4台节点+4块盘的方式。

## 二、集群部署

### 1.环境准备

```
1. 查看要分区的盘
lsblk 

2. 分区
fdisk /dev/sdb  #fdisk 物理盘
n
p
回车
回车
回车
w

3. 格式化
mkfs.xfs /dev/sdb1 

4. 创建目录
mkdir -p  /data/minio/data

5. 挂载
mount /dev/sdb1 /data/minio/data
```

### 2.安装

#### 2.1下载

```
wget https://dl.minio.org.cn/server/minio/release/linux-amd64/minio
```

#### 2.2安装

```
1. 创建对应目录
cd /data/minio
mkdir log conf app

2. 将程序移动过来
mv /root/minio /data/minio/app

3.赋予执行权限
chmod +x /data/minio/app/minio

4. 新建执行脚本
vim run.sh

#!/bin/bash

export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=admin@minio

/data/minio/app/minio server --config-dir /data/minio/app/conf  --console-address ":9001" \
http://xx.xx.xx.xx:9000/data/minio/data \
http://xx.xx.xx.xx:9000/data/minio/data \
http://xx.xx.xx.xx:9000/data/minio/data \
http://xx.xx.xx.xx:9000/data/minio/data >/data/minio/app/log/start.txt 2>&1 &

# 上述注释
1. http 后面跟着的是各个节点的ip，9001是web页面端口，9000是程序请求接口
2. /data/minio/data 是挂载的数据目录
3. 账号和密码是上面指定的admin / admin@minio


5. 启动
chmod +x run.sh
./run.sh

6. 页面查看
http://xx.xx.xx.xx:9001

# 常见问题
1. 如果没有启动9001端口，查看防火墙有没有放通 9000和9001端口
```
