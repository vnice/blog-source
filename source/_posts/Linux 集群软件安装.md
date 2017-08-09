---
title: Linux 集群软件安装
date: 2017-07-05
tags: [Linux]
---

# 需求
> 之前公司需要将某个系统部署切换到Linux 上部署多台，当时是两台Oracle Linux ，软件的安装让我来负责，刚开始就是一台一台的弄，感觉有点傻，还好是两台，如果是集群，那不得累死人。所以趁休息这个机会在家捣鼓一下，如果使用一台机器同时控制多台机器部署软件。

下面我直接使用MAC 机器下的5台CentOS7 来测试，内存还算给力。这5台机器全部是Centos7 mini 版安装。使用第5台机器来控制剩下的4台同时安装软件。

![](http://ww1.sinaimg.cn/large/818b7fe3gy1fidmmq8et6j21hb0tzwno.jpg)

# 寻找工具
在经过一番Google和知乎上搜索知乎，别人推荐使用pssh 工具来安装。pssh 意思是 parallel ssh program 一个python编写可以在多台服务器上执行命令的工具，同时支持拷贝文件，是同类工具中很出色的，类似[pdsh](http://kumu-linux.github.io/blog/2013/06/19/pdsh/)使用方法 。为方便操作，使用前请在各个服务器上配置好密钥认证访问。项目地址: [parallel-ssh](https://code.google.com/p/parallel-ssh/) (代码托管在google code 需要翻墙)

## 下载pssh
```bash
wget http://parallel-ssh.googlecode.com/files/pssh-2.3.1.tar.gz
tar zxvf pssh-2.3.1.tar.gz
cd pssh-2.3.1/
python setup.py install
```
python 在centos mini版本中已经存在。所以不需要安装python

首先保证这5台机器网络是通的。
在操作的机器上新建立一个host.txt文件里面存放其他需要安装软件的机器配置如下。

![host.txt](http://ww1.sinaimg.cn/large/818b7fe3gy1fidpjgvuwzj20fl06374q.jpg)

## 执行命令测试，所有命令如下
    -h 执行命令的远程主机列表,文件内容格式[user@]host[:port]
    如 test@172.16.10.10:229
    -H 执行命令主机，主机格式 user@ip:port
    -l 远程机器的用户名
    -p 一次最大允许多少连接
    -P 执行时输出执行信息
    -o 输出内容重定向到一个文件
    -e 执行错误重定向到一个文件
    -t 设置命令执行超时时间
    -A 提示输入密码并且把密码传递给ssh(如果私钥也有密码也用这个参数)
    -O 设置ssh一些选项
    -x 设置ssh额外的一些参数，可以多个，不同参数间空格分开
    -X 同-x,但是只能设置一个参数
    -i 显示标准输出和标准错误在每台host执行完毕后

```bash
 pssh -P -A -i -h host.txt -e error  uptime
```
执行上面的命令会报错 Exited with error code 255 这是因为pssh的一个[bug](https://code.google.com/archive/p/parallel-ssh/issues/80)
没有传递接收秘钥的信息给命令执行者，所以后面根本没办法执行命令。你可以先手动ssh单独一台机器接受ssh fingerprint 再看看效果，如下图 
  
  ![](http://ww1.sinaimg.cn/large/818b7fe3gy1fidqq8nceyj20pw0h9q7o.jpg)
  
  我先手动ssh 连接到了10.211.55.10这台机器，接受了fingerprint yes,然后执行了命令，其他没接受的全部抱错，但是这台机uptime命令执行成功，如果机器数量较小的话，可以先手动ssh 连接一下。
  如果机器数量众多，可以安装keychina工具[参考这篇文章](https://unix.stackexchange.com/questions/128974/parallel-ssh-with-passphrase-protected-ssh-key)
  然后我这边没有试验，因为CentOS 7环境中的keychina工具在rpm中无法直接下载。需要手动build一个，太麻烦了。
  
  下图显示在四台机器中执行w命令
![](http://ww1.sinaimg.cn/large/818b7fe3gy1fidrgzj09ij20hf0dtdk7.jpg)

## 测试安装软件
    发现一个很扯淡的问题。凡是遇到需要手动接收yes/no的地方全部处理失败
    
   ![](http://ww1.sinaimg.cn/large/818b7fe3gy1fidrycw4zyj206y06y0so.jpg)
   
   不过同时执行一些普通命令还是有效的。
    