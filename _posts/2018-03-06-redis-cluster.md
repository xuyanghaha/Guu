---
layout: post
title: "redis-trib构建redis集群"
author: "Guu"
categories: redis
tags: [documentation,sample]
image: spools.jpg
---

## 1. 准备工作
在安装redis集群前，首先要做一些准备工作，因为redis-trib工具是redis官方提供的集群构建工具，这个工具是用ruby实现的，所以要先安装ruby
```shell
yum -y install ruby ruby-devel rubygems rpm-build   #安装ruby、gem
gem update --system   #更新gem   
gem install redis -v 3.3.5 #安装redis扩展
```
如果安装失败，执行如下两句命令修改源再试一次
```shell
gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/
```
安装完成后检查一下是否安装成功
```shell
ruby -v
gem list #查看是否已安装redis扩展
```

## 2. 安装redis集群
1. 下载编译
```shell
cd /home/install
wget http://download.redis.io/releases/redis-3.2.8.tar.gz #下载
tar -zxvf redis-3.2.8.tar.gz   #解压
cd redis-3.2.8    #切换目录
make && make install  #编译安装（需要安装gcc 通过命令 yum install -y gcc g++ gcc-c++ make)
```

2. 创建cluster目录，这里以端口号命名，从7000-7005创建6个目录
```shell
mkdir -p /home/install/redis-3.2.8/cluster
cd /home/install/redis-3.2.8/cluster
mkdir 7000
mkdir 7001
mkdir 7002
mkdir 7003
mkdir 7004
mkdir 7005
```

3. 在每个目录下增加修改配置文件redis.conf
    例如7000的目录，复制一份redis.conf
```
cd 7000
cp /home/install/redis-3.2.8/redis.conf ./
```
    对其配置进行修改如下内容：
```
    port 7000 （与目录名一致）
    daemonize yes
    cluster-enabled yes
    cluster-config-file nodes.conf
    cluster-node-timeout 5000
    pidfile /var/run/redis.pid
```

4. 启动六个实例
举例第一个实例端口号为7000
```shell
cd /home/install/redis-3.2.8/cluster/7000
../../src/redis-server  ./redis.conf
```
启动第二个 7001
```shell
cd /home/install/redis-3.2.8/cluster/7001
../../src/redis-server  ./redis.conf
```
全部启用后，检查是否启用成功
ps -ef \| grep redis
出现下图说明OK：
![alt text]({{ site.github.url }}/assets/img/20180306redis_cluster_start.jpg "redis启用成功")
5. 构建集群
```shell
cd /home/install/redis-3.2.8/
./src/redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```
![alt text]({{ site.github.url }}/assets/img/20180306redis_cluster_slot.jpg "redis集群构建成功")
6. 集群验证
```shell
./src/redis-cli -h127.0.0.1 -c -p 7003      加参数 -C 可连接到集群
```
set hello world

可成功写入一个key "hello"  value 为 “world”

登到另一个实例上
```shell
./src/redis-cli -h 127.0.0.1 -c -p 7001
```
get hello   ,可成功读取值world 且指向7003

说明集群创建成功


