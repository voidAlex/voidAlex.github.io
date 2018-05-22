---
title: Java爬虫：爬取学校教务管理系统的信息
date: 2017-07-09 22:15:46
tags: ['爬虫','Java']
---
> 这是voidAlex原创的第二篇博文。
> 源码在[我的GitHub](https://github.com/voidAlex/NWNU_Crawler)
<!-- more -->

## 爬虫
关于爬虫是什么就不介绍了。这里简单说一下我所理解的爬虫的思路。

就如同把大象放冰箱里分三步一样，爬虫也分三步。第一步，由于教务管理系统这种网站需要登录才能获取信息，我们需要先用抓包工具抓取原始的请求地址；第二步，获得Cookie，伪装请求模拟登录，然后获得原始数据；第三步，解析数据，得到想要的东西。

## 抓包
学校教务管理系统的地址是[http://210.26.111.34/](http://210.26.111.34/)。登录界面长这样：

![](http://osuro1ft2.bkt.clouddn.com//17-7-10/3887073.jpg)

打开万能的Chrome开发者工具，填好表单点提交，然后在Network中能看到这样的信息：

![](http://osuro1ft2.bkt.clouddn.com//17-7-10/38663337.jpg)

标记的地方是我们要关注的。第一处能得到登录的请求地址是[http://210.26.111.34/mlogin.do](http://210.26.111.34/mlogin.do)，请求方式是POST；第二处能得到请求参数，分别是utype（用户类型），ucode（学号），pwd（密码），rember（记住账号）。原本登录时要输的验证码不见了，interesting。

登录之后通过同样的方式能获取到其他的请求地址，比如成绩，课表，考试安排等等。这里不再一一阐述。

## 模拟登录
用代码模拟登录请求也分三步：

1. 获取该网站的Cookie，并添加到请求头；

2. 添加参数，模拟登录；

3. 得到请求结果。

这里需要用到Apache HttpClient，它是一个支持Http协议的客户端编程工具包。HttpClient的官网是[https://hc.apache.org/index.html](https://hc.apache.org/index.html)。

### 获取Cookie

```Java
BasicCookieStore cookieStore = new BasicCookieStore();
CloseableHttpClient httpclient = HttpClients.custom().setDefaultCookieStore(cookieStore).build();

HttpGet getCookie = new HttpGet("http://210.26.111.34");
CloseableHttpResponse response1 = httpclient.execute(getCookie);
response1.close();
```

### 模拟登录

构造一个POST请求，将请求参数加进去，用httpclient提交。

```Java
HttpUriRequest postLogin = RequestBuilder.post().setUri(new URI(loginUrl))
                           .addParameter("utype", "S")
                           .addParameter("ucode", username)
                           .addParameter("pwd", password)
                           .addParameter("rember", "true")
                           .build();
CloseableHttpResponse response2 = httpclient.execute(postLogin);
response2.close();
```

## 得到结果，解析数据

以获取考试成绩为例，通过Chrome开发者工具可以得到考试成绩的原始请求和返回的数据，返回的数据是JSON，这里用Google Gson包去解析。

```Java
String getExamMark = "http://210.26.111.34/result/stqryResult/view.do";
HttpUriRequest postExamMark = RequestBuilder.post().setUri(new URI(getExamMark))
                        .build();

CloseableHttpResponse response = httpclient.execute(postExamMark);
HttpEntity entity = response.getEntity();
String json = EntityUtils.toString(entity);
EntityUtils.consume(entity);

Gson gson = new Gson();

JsonObject jo = gson.fromJson(json, JsonObject.class);
```

输出结果得到如下信息：
```Json
{"footer":[{"FE_CU_CREDIT":"<b>0</b>",
"FE_CU_YEAR":"副修学分:",
"FE_CU_NAME":"主修学分:[总分<b></b>,必修<b></b>]"}],
"rows":[{"FE_CU_NAME":"HTML5与JAVASCRIPT",
"FE_CU_YEAR":"2016秋季",
"FE_CU_CREDIT":"3",
"FE_SR_USUAL1":86,
...省略若干字
```

成功！

爬虫刚入门，不足之处欢迎大家批评指正。