date: 2015-11-18
author: YiHan
title: iptables 小结
tags: Linux, iptables
sulg: iptables

## 规则顺序
iptables 按照设定顺序比对规则内容，如果封包资料与规则一致则执行操作：ACCEPT，DROP，REJECT， LOG。 LOG只会记录信息到/var/log/messages中，该封包还是会继续下面的规则比对  
所以顺序很重要,错误的顺序错误的结果  
其中内网限制建议REJECT， drop并不会知会client端，client会等待直到timeout

## 封包政策， Policy
Policy 如果不是ACCEPT，务必小心，很可能会把自己挡住，务必开放自己的ssh 22端口权限

## table 和 chain
fileter  
mangle  
nat  
一般控制本机资源的访问，只要设置 filter中的INPUT 和 OUTPUT 就可以了  
![链路图](/images/iptables_chain.gif)

## 查看设置
```
iptables -L -n -v -t tables
-t 默认为filter
```

使用iptables-save 可以看到更有价值的信息，这部分内容也可以直接作为/etc/sysconfig/iptables 的配置文件内容
```
iptables-save
```

## 清除设置
先清除设置后加入需要设置是推荐做法  
下面操作不会更改policy， 如果policy为DROP，要非常小心，远程登陆的话自己就进不去了
```
iptables [-t tables] [-FXZ]
iptables -F #默认清除filter 删除所有规则
iptables -X #清除所有自定义的chain，tables
iptables -Z # 计数清零


```

## 设置预设政策
```
iptables [-t nat] -P [INPUT, OUTPUT, FORWARD] [ACCEPT, DROP]

iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -p FORWARD ACCEPT

```

## IP, 网域， 借口装置【网卡】

```
#接受所有本地lo网卡的所有封包
iptables -A INPUT -i lo -j ACCEPT

#接受所有来自特定ip和网段的封包,-i 可指定网卡
iptables -A INPUT -s 192.168.100.0/24 -i -eth1 -j ACCEPT
iptables -A INPUT -s 123.12.1.12/32 -j ACCEPT

```
## tcp， udp 端口
```
#-i 网卡 -p tcp udp --dport 目标端口 --sport 来源端口 -s 来源ip
# -d 目标ip【本机不常用】
# 上面参数可以各种组合使用
# --syn tcp 建立tcpip连接时握手信号，经常利用防火墙防御syn攻击，DDos攻击的一种
iptables -A INPUT -i eth0 -p tcp --dport 21 -j DROP
iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --sport 1:1022 --dport 1:1023 --syn -j DROP
```

## 封包状态
如果状态为响应状态，则接受
```
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
```


## 例子
clint 端常用，开放22端口用于ssh
```
iptables -F
iptables -X
iptables -Z
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

```

## 语法
```

[root@www ~]# iptables [-t tables] [-L] [-nv]
选项与参数：
-t ：后面接 table ，例如 nat 或 filter ，若省略此项目，则使用默认的 filter
-L ：列出目前的 table 的规则
-n ：不进行 IP 与 HOSTNAME 的反查，显示讯息的速度会快很多！
-v ：列出更多的信息，包括通过该规则的封包总位数、相关的网络接口等



[root@www ~]# iptables-save [-t table]
选项与参数：
-t ：可以仅针对某些表格来输出，例如仅针对 nat 或 filter 等等


[root@www ~]# iptables [-t tables] [-FXZ]
选项与参数：
-F ：清除所有的已订定的规则；
-X ：杀掉所有使用者 "自定义" 的 chain (应该说的是 tables ）啰；
-Z ：将所有的 chain 的计数与流量统计都归零


[root@www ~]# iptables [-t nat] -P [INPUT,OUTPUT,FORWARD] [ACCEPT,DROP]
选项与参数：
-P ：定义政策( Policy )。注意，这个 P 为大写啊！
ACCEPT ：该封包可接受
DROP   ：该封包直接丢弃，不会让 client 端知道为何被丢弃。


[root@www ~]# iptables [-AI 链名] [-io 网络接口] [-p 协议] \
> [-s 来源IP/网域] [-d 目标IP/网域] -j [ACCEPT|DROP|REJECT|LOG]
选项与参数：
-AI 链名：针对某的链进行规则的 "插入" 或 "累加"
    -A ：新增加一条规则，该规则增加在原本规则的最后面。例如原本已经有四条规则，
         使用 -A 就可以加上第五条规则！
    -I ：插入一条规则。如果没有指定此规则的顺序，默认是插入变成第一条规则。
         例如原本有四条规则，使用 -I 则该规则变成第一条，而原本四条变成 2~5 号
    链 ：有 INPUT, OUTPUT, FORWARD 等，此链名称又与 -io 有关，请看底下。

-io 网络接口：设定封包进出的接口规范
    -i ：封包所进入的那个网络接口，例如 eth0, lo 等接口。需与 INPUT 链配合；
    -o ：封包所传出的那个网络接口，需与 OUTPUT 链配合；

-p 协定：设定此规则适用于哪种封包格式
   主要的封包格式有： tcp, udp, icmp 及 all 。

-s 来源 IP/网域：设定此规则之封包的来源项目，可指定单纯的 IP 或包括网域，例如：
   IP  ：192.168.0.100
   网域：192.168.0.0/24, 192.168.0.0/255.255.255.0 均可。
   若规范为『不许』时，则加上 ! 即可，例如：
   -s ! 192.168.100.0/24 表示不许 192.168.100.0/24 之封包来源；

-d 目标 IP/网域：同 -s ，只不过这里指的是目标的 IP 或网域。

-j ：后面接动作，主要的动作有接受(ACCEPT)、丢弃(DROP)、拒绝(REJECT)及记录(LOG)



[root@www ~]# iptables -A INPUT [-m state] [--state 状态]
选项与参数：
-m ：一些 iptables 的外挂模块，主要常见的有：
     state ：状态模块
     mac   ：网络卡硬件地址 (hardware address)
--state ：一些封包的状态，主要有：
     INVALID    ：无效的封包，例如数据破损的封包状态
     ESTABLISHED：已经联机成功的联机状态；
     NEW        ：想要新建立联机的封包状态；
     RELATED    ：这个最常用！表示这个封包是与我们主机发送出去的封包有关



[root@www ~]# iptables -A INPUT [-p icmp] [--icmp-type 类型] -j ACCEPT
选项与参数：
--icmp-type ：后面必须要接 ICMP 的封包类型，也可以使用代号，
              例如 8  代表 echo request 的意思。



```
## 参考
[鸟哥私房菜](http://vbird.dic.ksu.edu.tw/linux_server/0250simple_firewall_3.php)


