---
title: Linux使用systemd添加系统启动项目
date: 2014-05-24
tags: [Linux]
---

```bash
/etc/systemd/system/nexus.service

This example is a script that uses systemd to run the repository manager service. Create a file called nexus.service. Add the following contents, then save the file in the /etc/systemd/system/ directory.

[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
Activate the service with the following commands:

sudo systemctl daemon-reload
sudo systemctl enable nexus.service
sudo systemctl start nexus.service
After starting the service for any Linux-based operating systems, verify that the service started successfully.

tail -f /opt/sonatype-work/nexus3/log/nexus.log

check status
sudo systemctl status nexus
```