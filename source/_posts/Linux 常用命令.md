---
title: Linux 常用命令
date: 2014-04-05
tags: [Linux]
---

# ls
    显示当前目录下的所有文件
    ls [OPTION]... [FILE]...
    -a --all显示隐藏文件(linux隐藏文件都是.开头)
    -l --list 显示文件的详细信息(依次为文件权限,引用次数,所有者,所属组,文件大小,最后修改时间，文件名)
    ls -alh 显示所有文件详细信息，文件大小按照可读的格式显示出来
    ls -ld 显示当前目录本身的详细信息

# mkdir
    make  directories  创建目录
    mkdir -p [目录名称] 递归创建目录

# cd
    change directories 切换目录
    cd ~ 切换到根目录
    cd .. 切换到上级目录

# pwd
    print working directories 显示当前目录

# rmdir
    remove empty directories 删除空目录

# cp
    copy 复制目录
    cp [源文件...] [目标文件]
    cp -r 复制目录
    cp -rp 复制目录保留属性

# mv
    move 剪切和改名
    mv[源文件或者目录...] [目标文件或者目录]

# rm
    remove 删除命令
    rm -r -f [文件或者目录]
      ■ -r 删除目录
      ■ -f 强制执行

# touch
     创建空文件
     touch [文件名称]

# cat
     查看文件 适合查看内容较少的文件
     cat -n 查看文件并显示行号

# more
     分页查看文件
     more [文件名]
     空格或者f 翻页
     Enter 换行
     q或者Q 退出

# less
    分页查看文件，可以选择往回翻页
    在翻页过程中可以 输入/xx 查找相关类容

# head
    查看文件的前几行
    head -n 10 查看文件前10行

# tail
    查看文件的末尾几行
    tail -n 10 显示文件末尾10行
    tail -f 动态显示文件末尾内容
# ln
    link 创建链接文件
    ln -s [源文件] [目标文件] 创建软链接文件

 ***

# chmod
    权限管理命令 更改目录或者文件的权限
    change the permissions of a file
    chmod [{ugoa}{+-=}{rwx}] [文件或者目录]
    chmod -R 777 使用数字表示权限 r=4,w=2,x=1, -R递归修改

# chown
    权限管理命令 chown 更改目录或者文件的所有者
    change file ownership
    chown [用户] [文件或者目录]

# chgrp
    权限管理命令 chgrp 更改文件或者目录的所属组
    chgrp [组的名称] [文件或者目录]

# umask
    权限管理命令 umask 显示新建文件的缺省权限
    umask -S 以rwx形式显示文件的缺省权限

***
# find
      文件搜索命令
      find [搜索范围] [匹配条件]
      find /etc -name init 在 etc 目录下搜索 名称为init的文件
      find /etc -name *init* 在 etc 目录下模糊搜索名称带有init的文件
      find /etc -name init??? 在 etc 目录下搜索 名称为init后带有三个字符的文件？代表三个字符
        -iname 不区分大小写
        -size 根据文件大小来查找 +n 204800 代表大于100MB的文件 -n 小于多少的文件 根据数据块的大小来搜索
          ■ 一个数据块=512字节=0.5k
        -user 查找所有者的文件
        -amin 访问时间 -cmin 文件属性 -mmin 文件内容 find /etc -mmin -5 表示查找5分钟之类被修改过得文件
        -type 按照文件类型查找 f 文件 d 是目录 l 是软链接
        -a -o 多个条件搜索是使用and 还是or
        find /etc/ -name inittab -exec ls -l {} \; 查找文件并执行命令
        find /home  -user dev -ok rm {} \; 查找用户dev下的文件找到后询问删除
        find . -inum 31531 -exec rm {} \; 查找文件节点是31531的文件并删除

# locate
     文件快速搜索命令
     在新建文件之后需要updatedb 更新索引 -i不区分大小写

# which
     命令搜索
     which rm 查找命令所在目录 并且列出是否有 别名

# whereis
    命令搜索 以及帮助文档的位置

# grep
    文件内容搜索
    grep -iv [指定字符] [文件]
    -i 不区分大小写
    -v 排除指定字符 grep -v ^#  /ect/inittab  查找指定文件 忽略以#号开头的行

# man
    帮助命令 manual
    man ls 查看ls命令目录 同less模式，可以输入/查找相关选项
    man services 查看配置文件
    man 5 passwd 查看配置文件的帮助信息 1代表命令 ，5代表配置文件

# help
    帮助命令
    查看内置命令或者shell 编程语法帮助

***

# useradd
    添加用户
    useradd tanwei 添加用户tanwei
    passwd tanwei 给新用户设置密码

# who
    查看当前系统登录信息
    tty 代表本地登录    pts 代表远程登录

# w
    查看当前系统的登录详细信息和负载
    21：35：27 当前系统时间
    up 10:54 系统连续运行时间  同命令uptime
    2 users 当前登录了多少用户
    load average: 系统连续 1分钟之类 5分钟之类 15分钟之类  负载值
    IDLE 登录空闲时间
    JCPU 当前登录用执行的命令户占用cpu时间
    PCPU 当前登录用户执行命令累计占用cpu时间
    WHAT 当前用户执行了什么


  ***

# gzip
    .gz压缩格式   只能压缩文件，并且不保留源文件
    gzip [文件名...]
    gunzip 或者gzip -d

# tar
    .tar.gz 压缩格式
     -c 打包
     -v 显示详细信息
     -f 指定文件名
     -z 压缩或解压
     -x 解包
     tar -zcvf app.tar.gz app 将目录app 压缩打包 并显示详细信息
     tar -zxvf app.tar.gz app 将目录app 解压缩 并显示详细信息

# zip
    -r 压缩目录
    unzip 解压缩文件

# .bz2
      bzip2压缩格式 之能压缩文件
      -k 保留源文件
      tar -cjf app.tar.bz2 和tar 搭配使用能够压缩目录
      bunzip2 解压缩文件 -k 解压后保留源文件
      tar -xjf app.tar.bz2 和tar搭配解压文件

# write
      write tanwei 给在线用户tanwei发送命令 Ctrl+D 保存结束

#  wall
    (writeall)命令
    发送广播消息

# ping
    测试网络连通性

# ifcofig
    (interface configure) 查看设设置网卡信息

# mail
    邮件发送命令

# last
    统计系统用户所有登陆信息

# traceroute
    探测网站路由

# netstat
    查询网络状态信息
    netstat -tlun 查询本机开启的端口
    netstat -an 查询本机开启的网络端口以及正在链接的网络程序
    netstat -rn 查看本机网关

# setup
    配置网络命令 有些系统需要先下载 yum install -y system-config-network-tui

# mount
    手动挂载cd
    mkdir /mnt/cdrom
    mount /dev/sr0 /mnt/cdrom 讲设备文件名 链接到挂载点
    umount /dev/sr0

