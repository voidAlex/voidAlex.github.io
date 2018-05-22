---
title: 理解Spring中的依赖注入
tags: ['Java', 'Spring']
date: 2017-11-09 20:55:00
---
> 这是voidAlex原创的第十七篇博文。
<!-- more -->

## 依赖注入和控制翻转

`IoC`(Inversion of Control，控制翻转)和`DI`(Dependency Injection，依赖注入)在Spring下是同等的概念。控制翻转是通过依赖注入来实现的。依赖注入是指由容器负责创建对象和负责维护对象间的依赖关系，从而实现解耦，体现一种组合的理念。

任何一个实际的应用，都会由两个或者更多的类组成。这些类之间相互调用以完成特定的业务逻辑。每个对象负责管理和调用与自己相互协作的对象，这会导致高度耦合的代码。

耦合具有两面性。一方面，高度耦合的代码将会难以测试，难以复用，难以理解，并且会表现出“打地鼠”式的bug特性（修复一个bug，又出现更多的新bug）。但是，一定程度的耦合又是必须的，完全没有耦合的代码什么也做不了。为了完成特定的功能，不同的代码之间必须进行交互。

通过依赖注入，创建被调用者的工作不再由调用者来完成，而是由Spring容器来完成，然后注入到调用者。这也意味着，当需要切换依赖的时候，不需要改变调用者的代码。依赖关系将被自动注入到需要它们的对象当中去。

知乎上有个回答，总结的相当到位。原文点[这里](https://www.zhihu.com/question/27053548/answer/113488399)。

![](http://osuro1ft2.bkt.clouddn.com//17-11-10/77388616.jpg)

## 实例

### 传统的做法

考虑下面代码：

```Java
public class JayChouCD {
    public void play();
}

public class CDPlayer {
    private JayChouCD cd;

    public void playCD() {
        this.cd = new JayChouCD();
        this.cd.play();
    }
}
```

我们有一个CD机，它需要一张CD才能够播放，或者说，`CDPlayer`依赖于`CD`。在上面的代码中，我们直接通过`new`关键字给`CDPlayer`创建了一个`JayChouCD`的实例，这样我们就能在这个CD机上听周杰伦的歌了。

### 面向接口编程

但是，这样做，我们能够听的歌十分有限，只能在这个CD机上听周杰伦的CD。考虑使用面向接口的编程方式改写上面的代码：

```java
public interface CD {
    void play();
}

public class CDPlayer {
    private CD cd;

    public void playCD() {
        this.cd = new JayChouCD();
        this.cd.play();
    }
}
```

与CD机直接打交道的类变成了`CD`，即使它最终实现依然是`JayChouCD`，但是这样做已经有明显的好处，所有调用都通过接口`CD`来完成。需要替换`JayChouCD`类，想听其他人的歌时，也只需要修改`CD`指向新的实现类。

### 动态生成对象

虽然上述代码已经很大程度的降低了耦合，但是耦合依然存在。想听其他人的歌曲时依然需要修改`CDPlayer`类内部的代码。当依赖过多的时候，修改起来会相当的麻烦。考虑通过反射机制中的动态加载类来修改上述代码：

```java
public interface CD {
    void play();
}

public class CDPlayer() {
    private static String CLASS_NAME = "JayChouCD";

    private CD cd;

    public void playCD() {
        Class class = Class.forName(CLASS_NAME);

        this.cd = (CD) class.newInstance();
        this.cd.play();
    }
}
```

这样，我们动态的得到了`CD`的实例，不必每次都为听谁的歌而苦恼了，只需要告诉CD机类名即可。获得类名可以通过配置文件去实现。这样我们就实现了`CD`和`CDPlayer`间的解耦。实际上，Spring中DI的底层就是通过反射机制来实现的。

### 使用Spring的DI

 Spring支持使用`xml`、`Java Config`和注解去装配Bena。Spring Boot推荐使用注解和`Java Config`的方式。使用Spring Boot的方式继续改造上面的代码：

 ```java
public interface CD {
    void play();
}


@Component
public class JayChouCD implements CD {
    private String title = "哎呦，不错哦！";

    public void play() {
        System.out.println(title);
    }
}


public class CDPlayer() {
    @Autowired
    private CD cd;

    public void playCD() {
        this.cd.play();
    }
}

 ```

 首先使用`@Component`注解告诉Spring，这个类是一个让你进行管理的bean，这意味着在其他类中可以通过Spring的依赖注入来得到它的实例。所以，在`CDPlayer`类中，我们使用了`@Autowired`注解，来将一个`CD`注入到`CDPlayer`中。

 Spring中，所有的Bean都通过IoC容器（ApplicationContext）来创建，并负责注入到需要的bean中。Spring Boot中，有四种常用的声明Bena的注解：

 >* `@Component`：组件，没有明确的角色
 >* `@Service`：在业务逻辑层使用（Service层）
 >* `@Repository`：在数据访问层使用（dao层）
 >* `@Controller`：在展现层使用（MVC）

 注入Bena的注解，一般情况下通用：

 >* `@Autowried`：Spring提供的注解
 >* `@Inject`
 >* `@Resource`

 这三个注解都可以用在属性、`set`方法、构造方法上。

 ## 总结

 一句话：控制翻转是将对象的创建权翻转给Spring，依赖注入是在Spring创建对象的过程中，把对象依赖的属性注入到类中。控制翻转是通过依赖注入来实现的。

 ## 参考资料

 《精通Spring 4.x 企业应用开发实战》

 《Spring实战》

 [CSND博客：关于Spring IOC (DI-依赖注入)你需要知道的一切](http://blog.csdn.net/javazejian/article/details/54561302)