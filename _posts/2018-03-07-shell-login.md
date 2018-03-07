---
layout: post
title: "shell的交互式和非交互式,登陆和非登陆"
author: "Guu"
categories: shell
tags: [documentation,sample]
image: nederland.jpeg
---

&emsp;上次安装zookeeper启动服务失败时遇到了shell的变量配置文件加载顺序的问题，从而引出了登陆式和非登陆式shell、交互式和非交互式shell，今天对照着bash官方文档把相关概念整理一下。  

## 登陆shell和非登陆shell
**登陆shell**：有关登陆shell官网给出了这样的定义：
> A login shell is one whose first character of argument zero is ‘-’, or one invoked with the --login option.

登陆shell指的是启用时传递的第0个参数为'-'开头的shell，或者使用'--login'参数启用的shell。

&emsp;简单点解释，有两种情况可以称之为登陆shell，第一种是shell的第0个参数，也就是当前shell的名字以'-'开头，例如：
```bash
[root@localhost ~]# echo $0
-bash

# 或者也可这样查询，$$代表当前进程号
[root@localhost ~]# ps | grep $$
65211 ttys000    0:00.03 -bash      #这里进程名以‘-’开头
```
第二种情况是启用shell时使用了'--login'参数：
```bash
ps |grep $$
11101 pts/1    00:00:00 bash  #当前bash进程为11101

bash --login       #启用一个bash 使用'--login'参数
ps                 #查看所有进程
--------------------------------
PID TTY          TIME CMD
11101 pts/1    00:00:00 bash
11117 pts/1    00:00:00 bash   #显示增加了一个bash进程，这就是一个登陆shell
11130 pts/1    00:00:00 ps
```

**非登陆shell：**如果直接用bash启用一个shell，这就是非登陆shell了。

&emsp;还有一个办法能看出当前shell是登陆还是非登陆，那就是exit命令和logout命令.  
**exit** 可以退出登陆和非登陆两种shell，如果退出了登陆shell 会返回logout，如果是非登陆会返回exit.  
**logout** 只能退出登陆shell ,如果用logout退出非登陆shell 会提示如下：  
```bash
bash: logout: not login shell: use `exit`
```
打开一个终端或者通过ssh远程登陆，通常我们使用的都是非登陆的shell，那么登陆和非登陆有什么区别呢,主要是加载环境变量配置文件的不同，我们后面再讲。

## 交互式和非交互式
**交互式shell：**就是在终端上执行，shell等待你的输入，并且立即执行你提交的命令。这种模式被称作交互式是因为shell与用户进行交互。这种模式也是大多数用户非常熟悉的：登录、执行一些命令、退出。当你退出后，shell也终止了。  
**非交互式shell：**运行bash script，执行脚本中的命令 ，读到文件末尾结束EOF，这就是非交互式shell。  

怎么判断是否是交互式呢，查看 $- ，它代表着当前shell的选项标志。查看返回的字符串中是否有字母i ,有则说明是交互式shell  
```bash
[root@localhost ~]# echo $-
himBH              #返回有i 为交互式
```

## 不同模式配置文件读取的不同
&emsp;首先说明一下，其实登陆非登陆、交互式非交互式不过是从两个不同的角度看shell，这两个属性是没有冲突的，可以有登陆交互式shell，可以有登陆非交互式shell各种情况。下面来了解一下不同情况下对应的配置文件读取规则：  

### 1、交互式登陆shell
启用时按照如下顺序读取执行配置文件，前提是文件存在且可读：  
/etc/profile  
～/.bash_profile  
~/.bash_login  
～/.profile  
除非当前shell在启用时使用了—noprofile选项，如果用了这个选项，则shell不会加载以上所有配置文件。  
另外，交互式登陆shell或者非交互式登陆shell执行exit退出时，shell会执行 ～/.bash_logout

### 2、交互式非登陆shell
会首先执行 ～/.bashrc  
但是如果shell使用了‘-norc’选项则不会读取这个文件；  
相反的，如果用了'--rcfile file'选项，则会强制用指定的file来替代 ～/.bashrc配置。   

~/.bash_profile通常会调用这个文件
```bash
if [ -f ~/.bashrc ];    # 判断home目录的.bashrc是普通文件的话 返回真
	then . ~/.bashrc;   # 等于source ~/.bashrc  让home目录下的.bashrc里的设置生效
fi
```

### 3、非交互式
非交互shell不执行初始化文件，从登陆shell处继承配置。  

### 4、使用sh命令
无论交互式登陆还是非交互式登陆，使用sh命令都会首先尝试/etc/profile ，然后执行 ~/.profile, 而且使用sh，--norc是默认开启的。  