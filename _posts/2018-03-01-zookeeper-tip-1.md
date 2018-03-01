---
layout: post
title: "zookeeper_server.pid: 没有那个文件或目录"
author: "Guu"
categories: shell
tags: [documentation,sample]
image: cityNight.png
---

今天捣鼓zookeeper，安装完成启动时却出现了一个奇怪的问题，提示“.../zookeeper/data/zookeeper_server.pid: 没有那个文件或目录 FAILED TO WRITE PID”。如下图：
![alt text](/assets/img/20180301zk_start_failed.png "zookeeper启动失败截图")

既然提示了103行报错，那索性到zkServe中找找原因吧
```shell
start)
    echo  -n "Starting zookeeper ... "
    if [ -f $ZOOPIDFILE ]; then
      if kill -0 `cat $ZOOPIDFILE` > /dev/null 2>&1; then
         echo $command already running as process `cat $ZOOPIDFILE`. 
         exit 0
      fi
    fi
    nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
    -cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "$ZOOCFG" > "$_ZOO_DAEMON_OUT" 2>&1 < /dev/null &
    if [ $? -eq 0 ]
    then
      if /bin/echo -n $! > "$ZOOPIDFILE"
      then
        sleep 1
        echo STARTED
      else
        echo FAILED TO WRITE PID
        exit 1
      fi
```

这才想起来Java环境变量没配好，在/etc/profile中添加环境变量并source，但是再次启动zookeeper依旧报同样的错误。
最终在网上找到一个解答，引用如下：
```
首先知道交互式shell和非交互式shell、登录shell和非登录shell是有区别的
在登录shell里，环境信息需要读取/etc/profile, ~/.bash_profile, ~/.bash_login, ~/.profile按顺序最先的一个，并执行其中的命令。除非被 —noprofile选项禁止了；
在非登录shell里，环境信息只读取 /etc/bash .bashrc 和 ~/.bashrc 
手工执行是属于登陆shell，脚本执行数据非登陆shell，而我的linux环境配置中只对/etc/profile进行了jdk1.6等环境的配
置，所以脚本执行/usr/local/zookeeper3.4/bin/zkServer.sh start 启动zookeeper失败了
```

也就是说ZK启动是脚本执行，脚本执行是非登陆shell，非登陆shell读取 /etc/bash .bashrc 和 ~/.bashrc ，我却只设置了/etc/profile，环境变量不会被脚本使用到，当然会因为找不到java启用失败了。


