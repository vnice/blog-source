---
title: Mac系统使用常见问题汇总
date: 2015-11-12
tags: [Mac]
---

# Mac 卸载rEFInd
```bash
执行检查命令

diskutil list | grep EFI | awk '{print $6}'
如果显示如下：
disk0s1

执行卸载命令

sudo mkdir /Volumes/efi
sudo mount -t msdos /dev/disk0s1 /Volumes/efi
sudo rm -rfP /Volumes/efi/EFI/refind
sudo bless --setBoot --mount /
重新启动即可
```