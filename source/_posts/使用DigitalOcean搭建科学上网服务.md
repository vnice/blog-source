---
title: 使用DigitalOcean搭建科学上网服务
date: 2017-07-02
tags: [其它]
---

# 注册
  首先在[DigitalOcean](https://www.digitalocean.com/)官网使用邮箱注册账号
  
  ![使用邮箱注册](http://upload-images.jianshu.io/upload_images/6722369-7d00549614a9ae27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  
# 绑定信息
在注册成功后需要绑定信用卡或者Paypal支付账号。这里只是绑定一个信息，并不是需要真正支付，后面每个月会有订单发给你支付。  
，这里推荐大家使用Paypal，没有可以先注册一个，因为信用卡可能是直接扣美金，如果你的信用卡不支持的话扣款会失败。
  ![信用卡或者Paypal绑定](http://upload-images.jianshu.io/upload_images/6722369-8c4c5a9ed5260f17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  
# 创建VPS
![创建VPS](http://upload-images.jianshu.io/upload_images/6722369-bf5571d6627ac246.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里创建vps就是DigitalOcean中所说的创建一个水滴

# 选择服务器和配置
这里大家根据自己的情况，熟悉哪种操作系统，或者使用的内存。我这里选择的CentOS和最便宜的配置，对于当做SS服务器足够了，然后选择的是新加坡节点，之前测试过因为这个地方离自己最近，延迟相对来说小一点。选择好后直接create

![选择服务器和配置](http://upload-images.jianshu.io/upload_images/6722369-3f868e67b5235d92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![创建中](http://upload-images.jianshu.io/upload_images/6722369-d66e79d2bcfa6df5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 设置账号和密码
创建完成后使用提供的console ssh登录 设置管理员账号和密码，新的密码会发送到你注册的邮箱中，是很长的一串随机数

![重置密码](http://upload-images.jianshu.io/upload_images/6722369-23d88e8ec9137017.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![登陆服务器](http://upload-images.jianshu.io/upload_images/6722369-4ed2db3e56437f3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

登录完成后会提示你修改密码。先输入刚刚邮件发送给你的随机密码，然后输入自己想设置的密码，设置完成后，vps就可以用啦。

# 登录服务器重置密码
![](http://upload-images.jianshu.io/upload_images/6722369-3b186fe673d4d1e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 搭建科学上网服务
下面开始搭建shadowsock 服务器，这里直接别人写好的[shadowsock-go](https://teddysun.com/392.html)一键安装脚本,可以参考，也有其他安装方式。
具体步骤就直接安装https://teddysun.com/392.html 上面来了
![SS服务器搭建完成](http://upload-images.jianshu.io/upload_images/6722369-d2794fae462ba111.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

默认按照上面开启的ss server 只有一个账号可以用，就是8989端口。其实可以配置多个账号，可以参考上面的说明，这里注意一下
开多个账号，你的端口也需要开通，例如Centos 7 需要开通 下面的8989 9001 9002 9003 9004 端口。具体根据你选择的操作系统来操作。

# SS 开通多个账号

![多个账号](http://upload-images.jianshu.io/upload_images/6722369-b40d02ee963dd859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 下载客户端
客户端的使用 在[shadowsocks github](https://github.com/shadowsocks?page=1)仓库里面有各种客户端，大家可以按照说明使用