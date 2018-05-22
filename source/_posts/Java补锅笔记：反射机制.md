---
title: Java补锅笔记：反射机制
date: 2017-11-02 08:39:31
tags: ['补锅笔记','Java','反射']
---
> 这是voidAlex原创的第九篇博文。
<!-- more -->

## 什么是反射

能够分析类能力的程序成为反射（*reflective*）。Java的反射机制可以让我们在编译期(*Compile Time*)之外的运行期(*Runtime*)检查类，接口，变量以及方法的信息。反射机制可以用来：

>* 在运行中分析类的能力；
>* 在运行中查看对象,获取成员变量、接口、构造方法等；
>* 动态创建和访问数组；
>* 运行时复制对象
>* ……

## Class类

在面向对象的世界里，一切皆对象。在Java中，只有静态成员变量和普通数据类型不是对象。类也是对象，它是`java.lang.Class`类的实例对象，任何一个类都是`Class`类的实例对象，并且一个类只有可能是`Class`类的一个实例对象。

在`Class`类的源码里，它的构造方法是私有的，上面有这么一段注释：

```
/*
 * Private constructor. Only the Java Virtual Machine creates Class objects.
 * This constructor is not used and prevents the default constructor being
 * generated.
 */
```

所以，`Class`类无法通过构造方法去实例化，只有JVM虚拟机才能创建`Class`类的实例对象。获得一个`Class`类的实例对象有下面三种方法：

### 第一种方法直接通过类的隐含的成员变量class去获取

```java
Class c1 = Student.class;
```

### 第二种方法，已知该类的实例对象，通过`getClass`方法去获取

```java
Class c2 = student.getClass;
```

### 第三种方法，通过完整的类名获得

```java
Class c3 = Class.forName("com.enity.Student");
```

所以我们可以通过该类的类类型来创建该类的实例对象，但是前提是该类需要有无参数的构造方法：

```java
Class c = Student.class;
Student student = (Student) c.newInstance();
```