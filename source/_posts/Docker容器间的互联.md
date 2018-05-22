---
title: Docker容器间的互联
date: 2017-11-08 16:55:53
tags: ['Docker']
---
> 这是voidAlex原创的第十六篇博文。
<!-- more -->

在Docker中，容器间是相互独立的。各个容器有自己的进程空间、文件系统、网络空间等。然而，容器如果不能和外界通信，是没用什么卵用的。它们只有相互通信的时候才能发挥作用。

## 端口映射

实际上，在之前使用`Spring Boot`和`mysql`的文章中已经在使用端口绑定了。在启动容器的时候可以可以加`-p`参数将容器的端口绑定到宿主机的端口：

```sh
docker run -d --name myMysql --volumes-from sqldata -e MYSQL_ROOT_PASSWORD=123456 -p 3307:3306 mysql
```

上面这条命令就是将`mysql`在容器中的`3306`端口映射到了宿主机的`3307`端口。然后通过宿主机的IP和端口，就能在外部访问这个`mysql`容器了。

## 容器链接

使用容器链接可以更方便的实现容器间互联。将一个容器链接到另一个容器时，Docker会添加一些环境变量来获取关联容器之间的信息。在启动容器的时候使用`--link`参数来链接其他容器：

```sh
docker run -p 8848:8848 --name jzfp --link myMysql -t jzfp/gsjzfp
```

启动后，该容器会将`myMysql`的网络信息以环境变量的形式添加到`jzfp`容器中，使得这个容器能够访问它。

## 实例

在Docker部署`Spring Boot`应用，并且通过容器链接来访问`mysql`中的数据。

### 启动mysql容器

首先启动一个`mysql`容器，映射到宿主机的`8849`端口：

```sh
docker run -d --name myMysql --volumes-from sqldata -e MYSQL_ROOT_PASSWORD=123456 -p 8849:3306 mysql
```

进入容器，设置外部访问账户：

```sh
docker exec -t -i myMysql /bin/bash
mysql -uroot -p
grant all on *.* to 'test'@'%' identified by '1234';
```

这时候，容器外部可以通过`8849`端口，使用`test`账户访问`mysql`。

### 配置Spring Boot数据库连接

修改`application.properties`中的数据库连接配置：

```sh
spring.datasource.url = jdbc:mysql://myMysql:3306/jzfpsd?characterEncoding=UTF-8
spring.datasource.username = test
spring.datasource.password = 1234
spring.datasource.driver-class-name = com.mysql.jdbc.Driver
```

可以看到，我们直接配置了数据库的IP地址为`myMysql`容器的名称。在启动该容器链接到`myMysql`后，即可通过Docker添加的环境变量去访问`myMysql`容器。

编译`Spring Boot`应用并生成镜像：

```sh
mvn clean package -Dmaven.test.skip=true docker:build
```

### 启动Spring Boot容器，链接到mysql

```sh
docker run -p 8848:8848 --name jzfp --link myMysql -t jzfp/gsjzfp
```

### 总结

使用Docker，即可在一台宿主机上实现数据库与WEB应用的分离。而且由于容器间链接和数据卷的特性，我们可以很方便的使用Docker打包数据文件和镜像，真正的实现一次打包，到处运行。