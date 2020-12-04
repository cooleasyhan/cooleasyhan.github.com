title: redhat6 sftp 设置
date: 2015-12-03
tags: redhat, sftp, linux
slug: redhat6-sftp-setup

## 创建用户组
用于与sftp设置进行匹配
```
groupadd  sftp_users
```

## 设置sftp

```
# Subsystem	sftp	/usr/libexec/openssh/sftp-server
Subsystem       sftp    internal-sftp
Match Group sftp_users
        X11Forwarding no
        AllowTcpForwarding no
        ChrootDirectory %h
        ForceCommand internal-sftp
```

- Match Group sftp_users – 匹配sftp_users组中的用户
- ChrootDirectory %h – 该参数指定用户验证后用于chroot环境的路径，用户登录后只能查看到该目录下内容，此处设置成你home目录
- ForceCommand internal-sftp – 该参数强制执行内部sftp


## 添加sftp用户
写了一个小脚本，    
需要注意的是root dirctory onwer 必须是root， 权限必须是755  
有问题可以查看日志：/var/log/syslog and /var/log/auth.log  
或者sftp -vvv user@host 显示详细的日志
```
#!/bin/bash
read -p "enter the user name: " USER_NAME

BASE_DIR=/sftp
mkdir -p $BASE_DIR

HOME_DIR=$BASE_DIR/$USER_NAME
DEFAULT_DATA_DIR=$HOME_DIR/data

useradd  -G sftp_users  -s /sbin/nologin  $USER_NAME -d $HOME_DIR
chmod 755 $HOME_DIR
chown root $HOME_DIR
chgrp -R sftp_users $HOME_DIR

mkdir $DEFAULT_DATA_DIR
chown $USER_NAME $DEFAULT_DATA_DIR

passwd $USER_NAME

```
