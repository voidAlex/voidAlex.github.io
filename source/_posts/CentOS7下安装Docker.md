---
title: CentOS 7下安装Docker并部署Spring Boot应用
date: 2017-11-06 10:07:09
tags: ['Docker', '配置']
---
> 这是voidAlex原创的第十四篇博文。
<!-- more -->

从内核和稳定性的角度考虑，Docker最好安装在Ubuntu 16.04上。但是在生产环境中，总是不可避免的使用CentOS。本文讲述在CentOS下安装Docker的过程。CentOS必须是64位，并且版本大于等于6.5。

## 配置静态IP

先用`ip addr`查看网卡信息,得到：

```sh
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:46:eb:2f brd ff:ff:ff:ff:ff:ff
...
```

可以看到使用的网卡是`ens33`，所以再修改`ifcfg-ens33`文件的信息。

```sh
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

修改如下内容：

```sh
BOOTPROTO=static #dhcp改为static   
ONBOOT=yes #开机启用本配置  
IPADDR=202.201.53.161 #静态IP  
GATEWAY=202.201.53.129 #默认网关  
NETMASK=255.255.255.0 #子网掩码  
DNS1=202.201.48.1 #DNS
DNS2=202.201.48.2
```

修改后效果：

```sh
cat /etc/sysconfig/network-scripts/ifcfg-ens33 
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=0c7d8882-dc8a-4e0a-bb82-1bb36aac80a8
DEVICE=ens33
ONBOOT=yes
IPADDR=202.201.53.161
GATEWAY=202.201.53.129
NETMASK=255.255.255.0
DNS1=202.201.48.1
DNS2=202.201.48.2
```

重启网络服务：

```sh
service network restart
```

## 安装Docker

### 使用脚本安装

为了简化安装流程，直接使用官方提供的脚本自动安装：

```sh
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh --mirror Aliyun
```

### 启动Docker

```sh
systemctl enable docker
systemctl start docker
```

### 查看安装信息

安装完毕后，使用`docker version`来查看安装信息：

```sh
Client:
 Version:      17.07.0-ce
 API version:  1.31
 Go version:   go1.8.3
 Git commit:   8784753
 Built:        Tue Aug 29 17:42:01 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.07.0-ce
 API version:  1.31 (minimum version 1.12)
 Go version:   go1.8.3
 Git commit:   8784753
 Built:        Tue Aug 29 17:43:23 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

### 建立Docker用户组

```sh
groupadd docker
# 将当前用户加入docker组
usermod -aG docker $USER
```

### 添加内核参数

默认配置下，在CentOS使用Docker会看到这些警告信息：

```sh
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```

添加内核配置信息启用这些功能：

```sh
tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

重新加载`sysctl.conf`：

```sh
sysctl -p
```

### 生成镜像并运行

生成镜像和运行的步骤与在Ubuntu下的一样，在这里不再赘述。