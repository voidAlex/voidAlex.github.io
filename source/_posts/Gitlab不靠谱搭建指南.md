---
title: Gitlab不靠谱搭建指南
date: 2017-11-01 19:20:34
tags: ['Gitlab','配置']
---
> 在Ubuntu 16.04下快速搭建Gitlab并汉化。
> 这是voidAlex原创的第八篇博文。
<!-- more -->

## 配置ip

### 使用`ifconfig`查看网卡名称，然后修改配置文件

### 修改`/etc/network/interfaces`

```sh
sudo vi /etc/network/interfaces
```

将内容修改为：

```sh
auto ens33
iface ens33 inet static
address 202.201.53.159
gateway 202.201.53.128
netmask 255.255.255.0  
```

### 重启网络，使之生效：

```sh
sudo /etc/init.d/networking restart
```

### 配置DNS：

```sh
sudo vi /etc/resolvconf/resolv.conf.d/base
```

添加：

```sh
nameserver 223.5.5.5
nameserver 223.6.6.6
```

保存后执行：

```sh
resolvconf -u
```

### 重启网络

```sh
sudo ifdown ens33 && sudo ifup ens33
```

## 更新apt，安装ssh和git

```sh
sudo apt-get update
sudo apt install ssh
sudo apt install git
```

测试ssh安装是否成功：

```sh
ssh localhost
```

## 安装Gitlab并配置依赖关系

### 安装邮件服务器：

```sh
sudo apt-get install curl openssh-server ca-certificates postfix
```

### 配置安装源：

```sh
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```

### 安装Gitlab并初始化

```sh
sudo apt install gitlab-ce
sudo gitlab-ctl reconfigure
```

### 修改host

修改`/etc/gitlab/gitlab.rb`中的`external_url`：

```sh
external_url 'http://nwnu.git.com'
```

在`/etc/hosts`中添加hosts映射：

```sh
127.0.0.1   nwnu.git.com
```

让Gitlab配置生效：

```sh
sudo gitlab-ctl reconfigure
```

## 汉化Gitlab

### 确定Gitlab的版本

```sh
sudo cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
```

### clone汉化版本库

```sh
git clone https://gitlab.com/xhang/gitlab.git
```

### 导出diff文件并补丁

```sh
sudo git diff v10.1.0 v10.1.0-zh > ../10.1.0-zh.diff
sudo gitlab-ctl stop
sudo patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < 10.1.0-zh.diff
```

### 重启Gitlab

```sh
sudo gitlab-ctl start
sudo gitlab-ctl reconfigure
```

## 备份

Gitlab默认的备份目录在`/var/opt/gitlab/backups`。

### 手动备份

```sh
sudo gitlab-rake gitlab:backup:create
```

### 自动备份

```sh
# 每天2点备份
0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1
```

### 恢复

恢复之前，确保备份文件所安装 GitLab 和当前要恢复的 GitLab 版本一致。首先，恢复配置文件：

```sh
sudo mv /etc/gitlab /etc/gitlab.$(date +%s)
# 将下面配置备份文件的时间戳改为你所备份的文件的时间戳
sudo tar -xf etc-gitlab-1399948539.tar -C /
```

恢复数据文件：

```sh
# 将数据备份文件拷贝至备份目录
sudo cp 1393513186_gitlab_backup.tar /var/opt/gitlab/backups/

# 停止连接数据库的进程
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop sidekiq

# 恢复1393513186这个备份文件，将覆盖GitLab数据库！
sudo gitlab-rake gitlab:backup:restore BACKUP=1393513186

# 启动 GitLab
sudo gitlab-ctl start

# 检查 GitLab
sudo gitlab-rake gitlab:check SANITIZE=true
```