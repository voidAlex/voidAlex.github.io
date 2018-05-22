---
title: 使用Docker打包和部署Spring Boot应用
date: 2017-11-04 11:46:13
tags: ['Docker','Spring Boot']
---
> 在Ubuntu 16.04下使用Docker部署Spring Boot应用。
> 这是voidAlex原创的第十一篇博文。
<!-- more -->

## Docker

### Docker简介

简单的来说，Docker是一种容器，属于操作系统层面的虚拟化技术，基于Linux内核对进程进行封装隔离。Docker从文件系统、网络通信到进程隔离等等，极大的简化了容器的创建与维护，使得Docker比传统的虚拟机技术更加轻便快捷。

### 与虚拟机技术的比较

传统虚拟机技术是虚拟出一套硬件，然后在其上运行一个完整的操作系统，在该系统上再运行所需要应用进程。而容器内的进程则直接运行在宿主的内核上，容器没有自己的内核，没有进行硬件虚拟。因此容器比传统的虚拟机更为轻便。

![](http://osuro1ft2.bkt.clouddn.com//17-11-4/82954513.jpg)

![](http://osuro1ft2.bkt.clouddn.com//17-11-4/18362650.jpg)

### Docker的优势

1. 更高效的利用系统资源
2. 更快速的启动时间
3. 一致的运行环境
4. 持续交付和部署
5. 更轻松的迁移
6. 更轻松的维护和扩展

## 准备工作

首先安装Oracle JDK和Maven，在编译Spring Boot应用时会用到。

```sh
sudo apt update
# 配置安装源
sudo apt install python-software-properties
sudo apt install software-properties-common
sudo add-apt-repository ppa:webupd8team/java
# 安装Oracle JDK
sudo apt update
sudo apt install oracle-java8-installer
# 安装Maven
sudo apt install maven
```

查看安装信息：

```sh
java -version
mvn -v
```

确保JDK版本为8及以上，Maven版本为3及以上。

## 安装和配置Docker

### 配置国内安装源

```sh
sudo apt-get update

sudo apt-get install apt-transport-https
sudo apt-get install ca-certificates
sudo apt-get install curl
sudo apt-get install software-properties-common

curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```

### 安装Docker CE

```sh
sudo apt-get update
sudo apt-get install docker-ce
```

### 脚本自动安装

可以使用Docker官方提供的脚本来简化安装流程：

```sh
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

### 启动Docker

```sh
sudo systemctl enable docker
sudo systemctl start docker
# 建立Docker用户组
sudo groupadd docker
# 将当前用户加入到用户组
sudo usermod -aG docker $USER
```

### 配置Docker远程访问

Docker默认不会监听任何端口，因此只能在本地使用Docker。如果先在其他机器上操作Docker主机，就要让Docker守护进程监听一个端口。修改Docker服务配置文件，添加一个未被占用的端口后，重启Docker服务：

```sh
vi /etc/default/docker
```

添加：

```sh
DOCKER_OPTS="-H 0.0.0.0:6000"
DOCKER_OPTS="-H unix:///var/run/docker.sock -H 0.0.0.0:5555"
```

重启Docker：

```sh
service docker restart
```

## Spring Boot应用配置

### 配置Maven依赖

在`pom.xml`中，加入这些内容：

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <docker.image.prefix>jzfp</docker.image.prefix>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>0.4.13</version>
            <configuration>
                <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                <dockerHost>http://host:port</dockerHost>
                <dockerDirectory>src/main/docker</dockerDirectory>
                <resources>
                    <resource>
                        <targetPath>/</targetPath>
                        <directory>${project.build.directory}</directory>
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
            </configuration>
        </plugin>

        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

Spotify的`docker-maven-plugin`插件是用于构建Maven的Docker镜像。`imageName`指定了镜像的名称，`dockerHost`指定了Docker主机的地址，`dockerDirectory`指定了Dockerfile文件的路径。

### 配置Dockerfile

在`src/main/docker`下创建`Dockerfile`，然后编辑：

```
FROM frolvlad/alpine-oraclejdk8
VOLUME /tmp
ADD gsjzfp-0.0.1-SNAPSHOT.jar app.jar
ENV JAVA_OPTS=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
```

其中，`FROM`指定了当前镜像集成的基镜像为`oraclejdk8`；`VOLUME`指定了临时文件目录为`/tmp`，该配置会在`/var/lib/docker`下创建一个临时文件并链接到容器的`/tmp`目录下；`ADD`将该应用的jar文件作为`app.jar`添加到容器里；`ENTRYPOINT`执行`app.jar`。

## 打包运行

### 编译并构建为Docker镜像

将整个项目目录copy到Docker主机，cd到项目目录下，执行：

```sh
mvn clean package docker:build
```

### 运行

```sh
docker run -p 8848:8848 -t jzfp/gsjzfp
```

如果程序运行正确，浏览器访问`http://host:8848`就能看到Spring Boot应用的主页了。