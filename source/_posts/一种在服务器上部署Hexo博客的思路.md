title: 一种在服务器上部署Hexo博客的思路
author: voidAlex
tags:
  - 杂记
  - 配置
categories: []
date: 2018-02-19 13:01:00
---

## 前言

笔者在过年期间买了域名和VPS，打算迁移博客到VPS上（GitHub Page访问速度实在是……你懂得）。由于平时使用的主力语言是Java，所以找了两个开源的Java博客系统[Solo](https://github.com/b3log/solo)和[Tale](https://github.com/otale/tale)，试用了之后发现都有不满意的地方，比如说：
- 主题少
- 有广告（虽然是开源的能理解，但是强迫症接受不了……）
- 不支持MathJax渲染（虽然自己加上了，但还是有问题，强迫症同样无法接受……）
- ……

思来想去还是hexo好，遂决定把hexo部署在服务器上。搜了一下发现大家都是把VPS当做Git服务器在用，然后用Nginx去解析静态资源。这种方式本质上和托管在GitHub上没什么区别，无非就是解析速度快点而已。这样做除了加快解析速度外，并没有任何好处。写一篇博文还是要经历`创建文件--写博文--预览--生成静态资源--使用Git同步`这么一个过程。而且在更换电脑后必须配置`Node.js`、`Git`、`Hexo`等一大堆东西之后才能写，等于花VPS的钱，只起到了一个加速效果。

好在`Hexo`还有`hexo-server`和`hexo-admin`这样的插件。之前并没有觉得`hexo-admin`这样的插件有什么用，但是在VPS上，它的价值体现出来了。于是，一个部署的思路诞生了：
- 使用`hexo-server`作为hexo的服务器
- 使用`hexo-admin`作为hexo的管理后台
- 使用`Nginx`作为反向代理服务器
- 使用`Git`与GitHub仓库同步（可选）

这样的话，等于给Hexo博客多了一个后台。成功部署后在本地只需要一个浏览器就可以美滋滋的写博客了~

阅读前方内容需要一定的基础，假设你已经在GitHub Page上成功的部署了你的博客，并且有一定的Linux基础知识。如果你还没有使用过Hexo，那么可以先看看[官方文档](https://hexo.io/zh-cn/docs/index.html)和[这篇教程](https://zhuanlan.zhihu.com/p/25471760)。

## （题外话）域名和VPS

在国内购买服务器和域名需要备案。为了避免麻烦笔者选择了[狗爹](https://godaddy.com/)和[vultr](https://www.vultr.com/?ref=7336080)。VPS套餐选择的是每个月5刀的套餐，配置只有1核CUP和1GB内存（编译Node.js用了40分钟），但是已经足够。

## 服务器的环境配置

服务器配置的所有操作均为在Ubuntu 16.04下的操作。

### 安装Node.js

Hexo是在Node.js上构建的博客系统，通过Node.js将`.md`文件渲染为`.html`文件。所以，安装Node.js必不可少。

安装Node.js的方法很多，笔者选择下载源码编译安装。注意，编译安装的时间长短视你的服务器配置而定，笔者的小水管上编译了40多分钟才玩。

首先安装`gcc`、`g++`、`make`、`python`等编译解释环境：
```
sudo apt update #老版本Ubuntu请使用apt-get
sudo apt install gcc g++ make python
```
获取Node.js的源码：
```
wget http://nodejs.org/dist/latest-v8.x/node-v8.9.4.tar.gz
tar zxvf node-v8.9.4.tar.gz
```
开始安装：
```
cd node-v8.9.4
./configure	#如果gcc、make等依赖未安装会报错
make install	#安装时间较长
```
安装完成之后验证：
```
node -v
v8.9.4
```

### 安装Nginx并配置反向代理

Nginx在Ubuntu下的安装较为简单，直接通过apt源即可安装：

```
sudo apt update
sudo apt install nginx
```

启动Nginx：
```
service nginx start
```

在浏览器中输入你的服务器ip进行验证（不用加端口号），如果能看见Nginx的欢迎页面说明安装成功。

编辑`/etc/nginx/nginx.conf`，配置反向代理：
```
server {
	listen       80; #监听端口，默认为http请求的80端口
    server_name  voidalex.one;	#你的域名
    
    location / {
        proxy_pass http://localhost:4000/;	#代理转发，你的hexo服务器的地址
    }
}
```

保存完毕，执行`service nginx reload`重启Nginx服务器。

## 配置DNS解析

在你购买的域名服务商的控制面板中，配置DNS解析，IP为你的服务器公网IP（不加端口）。设置完毕稍等一会（DNS服务器需要刷新缓存），访问你的域名即可看到Nginx的欢迎页面。

## 本地配置Hexo

### 安装插件

在服务器运行Hexo需要依赖于`hexo-server`和`hexo-admin`。如果你在本地已经安装这两个插件，请跳过这一步。（当然你也可以直接copy你的hexo博客目录到服务器上安装这两个插件，不过在本地先安装方便调试）

cd到你的博客根目录下，执行：
```
npm install hexo-server@0.3.1 --save
npm install hexo-admin@2.3.0 --save
```

安装完毕后，执行：
```
hexo s
```
打开浏览器，访问http://localhost:4000 即可预览博客，访问http://localhost:4000/admin 即可进入`hexo-admin`后台管理界面。

### 配置hexo-admin

hexo-admin默认没有开启密码保护，需要自己手动开启。

访问http://localhost:4000/admin 点击`Settings--Setup authentification here`进行密码设置：
![](https://voidalex-blog.oss-cn-beijing.aliyuncs.com/18-2-21/93762063.jpg)

输入用户名、密码后，将生成的代码复制：

![](https://voidalex-blog.oss-cn-beijing.aliyuncs.com/18-2-21/98662173.jpg)

然后打开博客根目录下的`_config.yml`，将复制的代码粘贴到末尾。

重新启动hexo服务器，访问http://localhost:4000/admin 如果出现登录界面，则配置成功。

## 将Hexo博客迁移至服务器

很简单，将你的整个博客目录打包，然后上传到服务器，然后解包。你在本地使用npm安装的hexo插件都在博客根目录下的`node_modules`目录下。如果你没有动这个目录，那么在服务器上解包之后就能直接用了。解包推荐使用`unzip`。

```
unzip blog.zip
cd blog
hexo s
```

如果一切正常，访问你的域名就可以看到博客了，访问域名/admin就能进入到后台界面。

> Tips：如果执行hexo相关命令报错的话，按照博客根目录下`package.json`中列出的插件名和版本重新按照一遍就好了。如：`npm install hexo@3.5.0 --save`

## 善后工作

实际上，到上一步已经成功的把Hexo部署在服务器上了。但是为了获得更好的体验还是需要再进行一些配置。

### 后台运行hexo
直接使用`hexo s`启动服务器，Ctrl+C或者shell关掉就结束进程了。可以使用`nohup`来后台运行hexo。在博客根目录下，执行：

```
nohup hexo s &
```
这样hexo就会运行在后台，输出的日志会被记录在`nohup.out`中。

### 优化解析速度
部署好了，但是加载速度仍然很慢。原因是每次访问的时候hexo都会动态的去加载`.md`文件，然后由Node.js渲染成html，再展示出来。看到hexo的官方文档中提了这么一句：
![](https://voidalex-blog.oss-cn-beijing.aliyuncs.com/18-2-21/65761270.jpg)
重启服务器之后，果然加载快了很多很多。

然而，这样启动服务器，你在后台编辑过的文章都不会被加载出来。需要你手动的执行`hexo g`命令，才能加载出来。

强迫症是忍受不了这样的操作的，好在`hexo-admin`中提供了这样一个功能：

![](https://voidalex-blog.oss-cn-beijing.aliyuncs.com/18-2-21/5139675.jpg)

点击按钮就能执行你的部署脚本。在博客根目录下的`_config.yml`中添加配置：
```
admin:
	deployCommand: './hexo-deploy.sh'
```

然后在博客根目录下创建`hexo-deploy.sh`，并编辑：
```
#!/usr/bin/env sh
hexo g
```

给`hexo-deploy.sh`授予权限：
```
chmod a+x hexo-deploy.sh
```

重启服务器，编辑博文后点击Deploy按钮就可以把文章渲染成html页面来访问了。

### 与GitHub Page同步

如果想在服务器上和原仓库同步的话，只需要配置Git，然后修改部署脚本就行了。Git的安装和配置不在赘述，要注意的是服务器上安装的Git不是用来配置Git服务器的，而是当做一个用户来提交代码的。配置完记得在你的GitHub中添加秘钥。

配置完之后，只需要修改`hexo-deploy.sh`就可以了：
```
#!/usr/bin/env sh
hexo g -d
```

然后重启服务器，以后每次点击deploy按钮时都会把渲染的html页面提交到GitHub仓库里。输出信息如下图：

![](https://voidalex-blog.oss-cn-beijing.aliyuncs.com/18-2-21/8761275.jpg)

![](https://voidalex-blog.oss-cn-beijing.aliyuncs.com/18-2-21/27412.jpg)

如果看到这些信息，说明配置没有问题，代码已经提交成功了。

验证一下：

![](https://voidalex-blog.oss-cn-beijing.aliyuncs.com/18-2-21/52277650.jpg)

![](https://voidalex-blog.oss-cn-beijing.aliyuncs.com/18-2-21/51560676.jpg)

## 安利一下

安利几个好东西吧。

[狗爹](https://sg.godaddy.com/zh/)，全球最大的域名服务商

[vultr](https://www.vultr.com/?ref=7336080)，美帝的VPS提供商，按分钟计费。推荐每个月5刀的套餐，硅谷节点（千万不要选新加坡和日本节点，很容易被墙而且速度奇慢）

[Moeditor](https://github.com/Moeditor/Moeditor)，十分简洁舒服的Markdown编辑器

[indigo](https://github.com/yscoder/hexo-theme-indigo)，很好看的一个Hexo主题




