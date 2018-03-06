---
layout: post
title: "zookeeper单机&集群部署"
author: "Guu"
categories: zookeeper
tags: [documentation,sample]
image: cutting.jpg
---

## 单机模式：


1. 创建安装目录  mkdir -p /home/install

2. 进入目录 cd /home/install,下载安装包
wget http://apache.fayea.com/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz

3. 解压 tar -zxvf zookeeper-3.4.10.tar.gz

4. 改一下目录名 mv zookeeper-3.4.10 zookeeper

5. 进入 zookeeper目录 创建data logs目录

6. 进入目录 cd /home/install/zookeeper/conf ,修改zoo.cfg配置信息如下： 
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/home/install/zookeeper/data
dataLogDir=/home/install/zookeeper/logs
# the port at which the clients will connect
clientPort=3181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

7. 启动服务  /home/install/zookeeper/bin/zkServer.sh start
jps检查是否有 QuorumPeerMain 如下，有则启用成功
```
[root@localhost zookeeper]# jps
4600 QuorumPeerMain
4622 Jps
```

8. 查看日志 /home/install/zookeeper下生成日志 zookeeper.out


## 集群版

使用三台虚拟机如下：
192.168.87.240 \ 192.168.87.241 \ 192.168.87.242

1. 首先在每台上执行上面单机模式的流程，确保都能成功启动

2. 修改每个节点的zoo.cfg文件，在末尾添加下面三行
```
server.1=192.168.87.240:2887:3887
server.2=192.168.87.241:2888:3888
server.3=192.168.87.242:2889:3889
```
```
server.X=A:B:C 
其中 X-代表服务器编号，例如240就是第一个节点，data目录下新建的myid文件中应该写1
A-代表ip
B用于peer间通信使用，例如商定更新的顺序，更具体一些，一个ZooKeeper服务器用这个端口来建立追随者到领导者的连接。当新的领导者出现时，追随者使用此端口打开与领导者的TCP连接。
由于领导选举也使用TCP连接，所以我们用端口号C进行领导选举。
```
3. 在server.1 虚拟机上的 /home/install/zookeeper/data 目录下创建文件myid
cd  /home/install/zookeeper/data
touch myid
vi myid
myid内容为1
相应的 server.2 虚拟机上的myid 为2 , server.3 虚拟机上的myid内容为3

4. 重启服务
cd /home/install/zookeeper/bin
./zkServer.sh stop
./zkServer.sh start
5. 查看是否成功
./zkServer.sh status
成功则现实节点身份如下：
![alt text]({{ site.github.url }}/assets/img/20180305zk_status_leader.jpg "zookeeper 角色 leader")
![alt text]({{ site.github.url }}/assets/img/20180305zk_status_follower.jpg "zookeeper 角色 follower")

