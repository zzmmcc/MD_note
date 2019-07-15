---
title: Linux笔记
date: 2019-07-08 16:22:10
tags:
---
#Linux的优点
    安全性、硬件要求低、开源免费、稳定性、
###linux常见命令
```linux
安装软件: yum install xxx
卸载软件: yum remove xxx
搜索软件: yum serach xxx
清理缓存:yum clean packages
列出安装列表: yum list
查看软件包信息: yum info xxx

添加用户: useradd xxx   /   adduser xxx
切换用户: su xxx
删除用户: userdel xxx(先退出再删除)

##防火墙
安装防火墙: yum install firewalld
防火墙:操作 systemctl     start/stop/restart/disable(禁止开启启动)/status(查看状态)    firewalld.service
开放端口: vim /etc/sysconfig/iptables 在前面添加-A INPUT -p tcp -m tcp --dport 4000 -j ACCEPT 然后保存重启

```
###文件目录
- ／dev 设备目录
- ／etc/ 系统配置及服务配置文件，比如我们之前修改网卡配置
- ／proc 显示内核及进程信息的虚拟文件系统
- ／tmp 临时文件目录
- ／home 普通用户家目录 :在公司中开发人员能拿到的都是普通用户，运维人员会创建很多普通用户，那么这些用户的信息就放在这下面
- ／root 超级管理员家目录
- ／var 变化的目录，一般是日志文件（／var／log），cache目录。／var／log／messages，／var/log／secure
- ／usr 用户程序及数据，帮助文件，二进制命令等目录（usr／local/），一般我们安装jdk、mysql、maven等都是放在这儿
- ／bin 普通用户命令的目录
- ／sbin 和／usr／sbin／：超级用户命令的目录

###文件操作命令

    查看目录下面的文件:   ls、ll、ll -h 
    新建文件:   touch
    新建文件夹:    mkdir
    删除文件:    rm -f 递归删除文件夹   rm -rf 强制递归删除文件夹
    进入目录:    cd ~ 当前用户的根目录   cd / 根目录    cd .. 上级目录
    查看当前目录: pwd
    复制: cp
    移动(重命名):    mv a/b/xxx.txt a/c/xxx.txt   mv a/b/xxx.txt a/b/yyy.txt
    从尾部读取:  tail -f xxx.txt
    从头部读取:  head xxx.txt (隐藏后面部分)   more xxx.txt(按enter可以往下读取一行)
    读取整个文件: cat xxx.txt
    按住上下箭头移动读取: less xxx.txt(按q退出)
    模糊查询(ab):   grep ab -n(显示行数) xxx.txt 
    统计行数:  cat xxx.txt | wc -l(统计个数)
    查找文件:   find /a/xxx.txt
    压缩: tar -z(有gzip属性的需要使用)  c(建立压缩文档) v(显示过程) f(表示用压缩档案的名称作为压缩文件的名称) /tmp/xmcc.tar.gz -C /usr/test xmcc.txt 将usr下的test文件夹中的文本压缩到tmp下面 跨目录才会使用 -C 同级目录下的俩个文件夹 压缩不需要 -C
    解压: -z x(表示解压) v f

    
    
    
