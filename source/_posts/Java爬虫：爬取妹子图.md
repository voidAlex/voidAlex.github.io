---
title: Java爬虫：上车吧！爬取妹子图
date: 2017-07-20 09:10:38
tags: ['爬虫','Java']
---
> 这是voidAlex原创的第三篇博文。
> 源码在[我的GitHub](https://github.com/voidAlex/Meizi_Crawler)
<!-- more -->

## 爬虫

上一篇博文介绍了如何模拟登录和解析JSON数据，这篇博文介绍怎么爬取不需要登录的网站的信息。

上一篇博文中关于爬虫的介绍可以点[这里](https://voidalex.github.io/2017/07/09/Java%E7%88%AC%E8%99%AB%EF%BC%9A%E7%88%AC%E5%8F%96%E5%AD%A6%E6%A0%A1%E6%95%99%E5%8A%A1%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9F%E7%9A%84%E4%BF%A1%E6%81%AF/)查看。

## 引入JSOUP

在`pom.xml`中添加JSOUP依赖：

```xml
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.10.2</version>
</dependency>
```

JSOUP是一款Java的HTML解析库，可以解析HTML中的文本内容。它的官网地址是[https://jsoup.org/](https://jsoup.org/)。

## 查看网页源码

好了，要干正事了。Google一下妹子图，找到这两个网站：[http://jandan.net/ooxx](http://jandan.net/ooxx)，[http://www.youmzi.com/tuinvlang.html](http://www.youmzi.com/tuinvlang.html).先看第一个，它的源码长这样：

![](http://osuro1ft2.bkt.clouddn.com//17-7-20/99168668.jpg)

我们要找的是所有的img标签里的URL，然后把它下载下来。但是这样似乎只能爬取单个页面的妹子图？所以我们还要找到下一页的URL：

![](http://osuro1ft2.bkt.clouddn.com//17-7-20/68438084.jpg)

恩，找到了。开始写代码吧！

## downloadImage方法

首先写一个下载图片的方法，该方法传入图片的URL和要写入的路径，然后将文件写入本地。需要调用`java.net`包中的一些方法。

```Java
public static boolean downloadImage(String imageUrl, String path) {
    try {
        //分割字符串，获得文件名
        String filePath = path + imageUrl.substring(imageUrl.lastIndexOf("/"));
        //获得文件流
        URL url = new URL(imageUrl);
        HttpURLConnection connection = (HttpURLConnection)url.openConnection();
        connection.setConnectTimeout(10000);
        connection.setReadTimeout(10000);
        InputStream in = connection.getInputStream();

        //写入本地文件
        File file = new File(filePath);
        FileOutputStream out = new FileOutputStream(file);
        int i = 0;
        while ((i = in.read()) != -1){
            out.write(i);
        }
        System.out.println(imageUrl + "下载成功");
        out.close();
        in.close();
        return true;
    }catch (Exception e){
        System.out.println(imageUrl + "下载失败");
        return false;
    }
}
```

需要注意的是要设置超时的时间，要不然会导致很多的图片下载失败。

## 解析HTML

接下来我们需要理一理这个爬虫的思路：
1. 打开这个网页。获取到网页所有图片的URL，然后遍历这些URL去下载图片；
2. 当遍历结束后，去找下一页的URL，然后执行1；
3. 直到找不到下一页的URL为止。

### 煎蛋妹子图

首先用`jsoup`中的方法获取到网页并且得到Document对象：

```Java
Document doc = null;
doc = Jsoup.connect(url).get();
```

获得所有的img标签：

```Java
 Elements elements = doc.getElementsByTag("img");
```

遍历这些标签并且下载：

```Java
for (Element element : elements){
    //获取标签中src属性的绝对路径
    String imgSrc = element.attr("abs:src");
    if (downloadImage(imgSrc, path)){
        count ++;
    }
}
```

获取下一页的地址，如果没有则退出循环:

```Java
try {
    url = doc.getElementsByClass("previous-comment-page").get(0)
            .getElementsByTag("a").attr("abs:href");
}catch (Exception e){
    System.out.println("没链接了~");
    break;
}
```

大功告成！

### 优妹子

第二个网站的略微复杂一点，上方有导航栏，每一页有若干个专题，点击进去了才是大图。所以需要爬取的链接稍微多一点。

首先还是获取到网页并且得到Document对象：

```Java
Document doc = null;
doc = Jsoup.connect(url).get();
```

然后获得每一个二级页面（即专题）的URL，并放到一个List里面：

```Java
Elements imageUrl = doc.getElementsByClass("tzpic3-mzindex").get(0).getElementsByTag("a");
ArrayList<String> urlList = new ArrayList<String>();
for (Element element : imageUrl){
    urlList.add(element.attr("abs:href"));
}
```

对于List里的每一个URL，去找它每一个的img标签并且获取下一页的URL：

```Java
for (String s : urlList){
    String next = s;
    while (true){
        try {
            doc = Jsoup.connect(next).get();
        }catch (IOException e){
            System.out.println(url + "请求失败");
        }
        Element e = doc.getElementsByClass("arpic").get(0);
        //获取所有img标签
        Elements elements = e.getElementsByTag("img");
        for (Element element : elements){
            //获取标签中src属性的绝对路径
            String imgSrc = element.attr("abs:src");
            if (downloadImage(imgSrc, path)){
                count ++;
            }
        }
        String tmp = next;
        try {
            Elements nextPage = doc.getElementsByClass("jogger2").get(0).getElementsByTag("a");

            next = null;
            for (Element element : nextPage){
                //获取标签中src属性的绝对路径
                if (element.text().equals("下一页")){
                    next = element.attr("abs:href");
                }
            }
        }catch (Exception ex){
            System.out.println("没链接了~");
            break;
        }
        if (next == null || tmp.equals(next)){
            break;
        }

    }

}
```

## 运行

```Java
public static void main(String args[]) throws Exception{
    String path = "image";
    jiandan(path);
    youmeizi(path);
}
```

跑了两个多小时终于跑完了，看一下战果：

![](http://osuro1ft2.bkt.clouddn.com//17-7-20/40407389.jpg)

恩，将近1w张。不说了，我先喝瓶营养快线去~