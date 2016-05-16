+++
date = "2016-05-14T22:40:29+08:00"
description = "网络安全很重要，很重要，很重要"
draft = false
tags = ["vps","安全","centos7"]
title = "对新vps必要的安全设置"
topics = ["vps"]

+++
    
### 以作备忘

最近vultr在做活动，新手冲钱送20$，我按捺不住的冲了5$，正好拿来跑测试网站。

我一直喜欢用centos，所以这次选了的是最新的centos7_64版本，它有了大幅度的更新，比如防火墙从 iptables 换成了 firewalld，service 换成了 systemctl，还有别的很多地方都跟我以前熟悉的6有了比较大区别，所以我需要记录下来。

<!--more-->

### 安装epel源，并且更新系统

```
$yum -y install epel-release
$yum update
```

### 创建新用户并赋权，禁止root登陆，更换ssh端口号，并且在防火墙放行必要端口

```bash
#创建用户
$useradd jam -d /home/jam -m -s /bin/bash
$passwd jam

#赋予权限
$visudo
root    ALL=(ALL)       ALL
jam     ALL=(ROOT)NOPASSWD：   All

#编辑ssh配置
$ vi /etc/ssh/sshd_config
Port  12345  #修改端口
PermitRootLogin no   #禁止root登陆
AllowUsers jam

$systemctl restart sshd.service

#当然用密钥登陆，而禁止密码登陆最好
```
编辑防火墙配置，放行 12345端口
```
1.命令方法
$firewall-cmd –permanent –add-port=12345/tcp
$systemctl restart firewalld.service

2.修改配置方法
$cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/
$vi /etc/firewalld/services/ssh.xml

$<port protocol="tcp" port="2345"/>

$systemctl start firewalld.service
$systemctl enable firewalld.service
```
### 开启SELINUX（可选）

```
$vi /etc/selinux/config
SELINUX=enforcing       #(enforcing为开启，disabled为关闭)
```
### 增加SWAP分区

```
#查看分区情况
$free -m

#增加 swap分区，容量为1GB
$dd if=/dev/zero of=/var/swap bs=1024 count=1024000

#设置交换文件
$mkswap /var/swap

#立即激活启用交换分区
$swapon /var/swap

#收回swap空间
$swapoff /var/swap
$rm /var/swap
```

### 配置hosts.deny文件预防暴力破解
```
vi /etc/hosts.deny

将东北大学的host黑名单下载复制进去[hosts.deny](http://antivirus.neu.edu.cn/ssh/lists/hosts.deny)
```


### 安装fail2ban禁止ssh暴力破解

需要安装支持 firewalld 的 fail2ban
```
yum -y install fail2ban-firewalld fail2ban-systemd
```
新建配置 jail.local
```
[DEFAULT]
bantime = 86400  #一旦触犯，ban一整天，也就是这些秒
usedns = no      #这个必须关闭，否则影响 tail -f /var/log/fail2ban。。的输出
enabled = false   #总开关默认为false不能改，需要哪个服务单独写
banaction = firewallcmd-ipset  #禁止ip的方法
action = %(action_)s    #采取的措施为， 仅仅ban掉

[sshd]
enabled = true
port    = ssh
logpath = %(sshd_log)s
action = %(action_)s

[sshd-ddos]
enabled = true
port    = ssh
logpath = %(sshd_log)s
action = %(action_)s


$systemctl enable fail2ban
$systemctl restart fail2ban.service
```
查看fail2ban的运行状况
```$systemctl status fail2ban```

查看实时禁止状况
```$tail -f /var/log/fail2ban.log```

查看已经禁止ip列表
```ipset --list```

### 参考文章

+ [ifshow](https://www.ifshow.com/centos-7-install-fail2ban-with-firewalld-to-defend-brute-force-password/)
+ [freehao123](http://www.freehao123.com/linux-vps-root/)



