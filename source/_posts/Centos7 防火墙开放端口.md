---
title: Centos7防火墙开放端口
date: 2017-06-05
tags: [Linux]
---

CentOS升级到7之后，发现无法使用iptables控制Linuxs的端口，google之后发现Centos 7使用firewalld代替了原来的iptables。下面记录如何使用firewalld开放Linux端口：

开启端口

firewall-cmd --zone=public --add-port=80/tcp --permanent

命令含义：

--zone #作用域

--add-port=80/tcp  #添加端口，格式为：端口/通讯协议

--permanent  #永久生效，没有此参数重启后失效

重启防火墙

firewall-cmd --reload
