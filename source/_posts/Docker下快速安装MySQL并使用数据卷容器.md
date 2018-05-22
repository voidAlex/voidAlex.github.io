---
title: Docker下快速安装MySQL并使用数据卷容器
date: 2017-11-06 19:32:34
tags: ['Docker', 'MySQL']
---
> 这是voidAlex原创的第十五篇博文。
<!-- more -->

## 数据卷和数据卷容器

### 数据卷

Docker下数据卷是一个可供容器直接使用的特殊目录，有如下特性：

>* 数据卷可以在一个或多个容器见共享和重用
>* 对数据卷的修改会立即生效
>* 对数据卷的操作不影响镜像
>* 数据卷的生命周期独立于容器

### 数据卷容器

数据卷容器也是一个正常的容器，专门提供数据卷供其他容器挂载的。

## 创建数据卷容器

创建一个名为`sqldata`的数据卷容器：

```sh
docker run -d -v /data/mysql:/var/lib/mysql --name sqldata training/postgres
```

这个命令会创建一个名为`sqldata`的数据卷容器，并且将容器中的`/var/lib/mysql`目录映射到宿主机的`/data/mysql`下。

## 创建mysql容器并挂载数据卷容器

首先拉去`mysql`的镜像：

```sh
docker pull mysql
```

创建`mysql`容器，并挂载数据卷`sqldata`：

```sh
docker run -d --name myMysql --volumes-from sqldata -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 mysql
```

这个命令会创建一个MySQL容器，为`root`用户设置密码为`123456`，并且挂载`sqldata`数据卷。

进入到`myMysql`容器中，并且登录到mysql服务器：

```sh
docker exec -t -i myMysql /bin/bash
mysql -uroot -p
```

这时候，root用户只能通过`localhost`访问，增加用户，授予其对应权限：

```sh
grant all on *.* to 'test'@'%' identified by '1234';
```

创建`test`用户后，就可以以正常访问mysql数据库的方式远程访问`myMysql`容器了。

## 通过数据卷容器备份、恢复和迁移数据

### 备份

创建一个新容器，加载`sqldata`容器中的数据卷，并从主机挂载当前目录到容器的`/backup`目录。容器启动后，将`sqldata`数据卷备份为当前容器中的`/backup/backup.tar`文件。

```sh
docker run --volumes-from sqldata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /data/mysql
```

### 恢复

先创建一个带有空的数据卷的容器：

```sh
docker run -v /data/mysql:/var/lib/mysql --name backupdata ubuntu /bin/bash
```

再创建另一个容器，挂载`backupdata`容器中的数据卷，解压备份文件到挂载的容器数据卷中：

```sh
 docker run --volumes-from backupdata -v $(pwd):/backup busybox tar xvf /backup/backup.tar
```