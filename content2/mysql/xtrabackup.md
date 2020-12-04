title: xtrabackup 小结
date: 2015-12-13
slug: xtrabackup


## 安装  
官网下载相关rpm 包, 此处为 percona-xtrabackup-20-2.0.8-587.rhel6.x86_64.rpm， 其中5.1版本的mysql只能使用 2.0版本的xtrabackup
```
rpm -ivh xxx.rpm
```


## 使用要点
1. 正常备份会锁表，可以使用 




## 备份脚本
写了一个脚本，用于每周日全备，其他时候增量备份
```
#!/bin/bash
#author: yihan
#date: 2015-12-13
#
#
#创建用户
#mysql> CREATE USER 'bkpuser'@'localhost' IDENTIFIED BY 'bkpuserpassword123';
#mysql> REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'bkpuser'@'localhost';
#mysql> GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'bkpuser'@'localhost';
#mysql> FLUSH PRIVILEGES;


weekday=`date +%w`
port=3306
ip=localhost
base_dir=/data/bakup
back_dir_name=mysql
back_dir=$base_dir/$back_dir_name
file_cnf=/etc/my.cnf
user_name=bkpuser
password=bkpuserpassword123


timestamp=`date +"%Y%m%d%H%M%S"`
Xtrabackup_log=$back_dir/Xtrabackup_log_$timestamp.out
shell_log=$base_dir/backup.log


function log
{
    t=`date +"%Y-%m-%d %H:%M:%S"`
    echo "[$t] $1" >> $shell_log
}

function full_backup
{
    log "full_backup begin, paras: $1"
    target_dir=$1
    innobackupex --defaults-file=$file_cnf --no-timestamp\
      --user=$user_name --password=$password \
      --host=$ip --port=$port  $back_dir/$target_dir 1> $Xtrabackup_log 2>$Xtrabackup_log
    log "full_backup end"
}


function incremental_bakup
{
    log "incremental_bakup begin paras: $1, $2"
    xtra_base_dir=$1
    xtra_target_dir=$2
    innobackupex --defaults-file=$file_cnf --no-timestamp \
    --user=$user_name --password=$password  --host=$ip \
    --port=$port --incremental --incremental-basedir=$back_dir/$xtra_base_dir \
    $back_dir/$xtra_target_dir 1> $Xtrabackup_log 2>$Xtrabackup_log
    log "incremental_bakup end"
}

function back_up_dir
{
    mv $back_dir ${back_dir}_bak_${timestamp}
}

function delete_old_backup
{
    if [ -d $base_dir ]; then
        cd $back_dir
        find ${back_dir_name}_bak_* -mtime +30 | xargs rm -f
    fi
}

####################main###############
mkdir -p $back_dir
log "++++++++++++++++++++++++++++++++++++++++++++++++++"
log "weekday:$weekday"
if [ 0 -eq $weekday ];then
    back_up_dir
    mkdir -p $back_dir
    full_backup backup0
else
    let "last_day = $weekday - 1"
    if [ -d $back_dir/backup$last_day ]; then
      incremental_bakup backup$last_day backup$weekday
    else
      full_backup backup$weekday
    fi
fi

log "delete old backup"
delete_old_backup


log "--------------------------------------------------"



```


## 恢复脚本 
后续
