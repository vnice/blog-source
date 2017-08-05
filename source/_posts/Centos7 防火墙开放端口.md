---
title: Centos7防火墙开放端口
date: 2017-06-05
tags: [Linux]
---

``` bash
    firewall-cmd --zone=public --add-port=1017/tcp --permanent
    success
    firewall-cmd --reload
    success
```
