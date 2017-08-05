---
title: Linux 使用常见问题汇总
date: 2014-06-05
tags: [Linux]
---

# 后台运行程序将日志输出到指定文件

```bash
 nohup ./IntelliJIDEALicenseServer_linux_amd64 > ~/var/logs/idealicense.log 2>&1 &
```

# apt install Unmet dependencies

```bash
    sudo apt --fix-broken install
```

# CentOS 7 x86_64 新建用户 图形化界面密码正确但是不能登陆的问题
```bash
root@chenghy ~]# vi /etc/pam.d/login
#将如下行：
session required /lib/security/pam_limits.so
#修改成：
session required /lib64/security/pam_limits.so
```
> 如果没有该行则添加 然后重启，故障排除,貌似这种现象只发现在x86_64服务器中。

# CentOS 设置JAVA环境变量之后java和javac 版本不一致的问题​

> centOS 7 默认安装后自带了jdk 1.8.0_65 的open jdk
不想使用这个版本 然后自己添加了一个jdk 1.8.0_121 版本
在/etc/profile 中设置了JAVA_HOME 和CLASSPATH
``` bash
export JAVA_HOME=/root/develop/java/jdk-1.8.0_121
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
source /etc/profile
```
执行 java -version 显示的任然是open jdk 1.8.0_65
执行命令 whereis java
显示 java:/usr/bin/java  而不是我们root/develop/下的java
备份 /usr/bin/java  mv /usr/bin/java /usr/bin/open-jdk.1.8.0_65
将我们自己的jdk软链接到/usr/bin/java
```bash
ln -s $JAVA_HOME/bin/java /usr/bin/java
 ```
验证 java -version  javac -version  版本一致OK

