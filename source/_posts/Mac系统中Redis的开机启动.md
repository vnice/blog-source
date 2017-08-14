---
title: Mac系统中Redis服务的开机启动
date: 2016-12-12
tags: [Mac,Redis]
---

# 场景
    今天在Mac系统上安装了Redis，来做一些程序的测试,但是重新开机后，Redis服务需要手动启动，Linux可以制作启动服务，Mac是否也可以呢
    
# launchd

  这里有一篇文章[launchd — 你应该了解的 OS X 工具](https://www.ulumen.com/launchd-tool-of-os-x-you-should-know-about/)已经说的很清楚，就是我要找的东西
    
    launchd 是苹果公司开发的一个开源的进程管理器，从 Mac OS X 10.4 Tiger 开始，苹果就使用 launchd 来管理系统的守护进程、程序、脚本、定时任务及 OS X 系统环境
    
# 使用
  首先下载Redis软件，在Make install 之后编写 redis.plist文件，plist文件其实就是一个xml文件。

```xml
  <?xml version="1.0" encoding="UTF-8"?>
      <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
      <plist version="1.0">
        <dict>
          <key>KeepAlive</key>
          <true/>
          <key>Label</key>
          <string>local.autorun.redis</string>
          <key>ProgramArguments</key>
          <array>
              <!--这里的目录根据自己的情况来创建-->
            <string>/usr/local/bin/redis-server</string>
            <!--这里的目录根据自己的情况来创建-->
            <string>/usr/local/var/redis/redis.conf</string>
          </array>
          <key>RunAtLoad</key>
          <true/>
          <key>UserName</key>
          <string>tanwei</string>
          <key>WorkingDirectory</key>
          <string>/usr/local/var</string>
          <key>StandardErrorPath</key>
          <string>/usr/local/var/log/redis.log</string>
          <key>StandardOutPath</key>
          <string>/usr/local/var/log/redis.log</string>
        </dict>
      </plist>
```
执行命令,plist文件路径根据自己的情况放置。
```bash
launchctl load /usr/local/var/redis/redis.plist
```

# 重启系统验证Redis是否启动