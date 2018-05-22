---
title: Java补锅笔记：利用反射获得对象的信息
date: 2017-11-02 11:12:20
tags: ['补锅笔记','Java','反射']
---
> 这是voidAlex原创的第十篇博文。
<!-- more -->
 
Java中，类的`Class`类的实例对象，类的成员变量也是对象，它是`java.lang.reflect.Field`的实例对象。`Field`类封装了关于成员变量的操作。

同样的，类的方法也是对象，它是`java.lang.reflect.Method`的实例对象。`Methon`类封装了关于类方法的操作。

构造函数是`java.lang.reflect.Constructor`的实例对象。

## 通过反射获取对象的信息

先声明一个普通的`Student`类：

```java
/**
 * Created by 王麟东 on 2017/11/2 0002 19:12. Email: wangld1994@gmail.com
 */
public class Student {
  private String name;
  private int age;
  private String id;

  public Student(){

  }

  public Student(String name, String id){
    this.name = name;
    this.id = id;
  }

  public String getName() {
    return name;
  }

  public int getAge() {
    return age;
  }

  public String getId() {
    return id;
  }

  public void setName(String name) {
    this.name = name;
  }

  public void setAge(int age) {
    this.age = age;
  }

  public void setId(String id) {
    this.id = id;
  }
}
```

`Student`类有三个私有的成员变量，一个带参数的构造方法和一个不带参数的构造方法，还有对各个成员变量的`getter`和`setter`方法。

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

/**
 * Created by 王麟东 on 2017/11/2 0002 18:01. Email: wangld1994@gmail.com
 */
public class ClassMessage {
  public static void getClassMessage(Object o){
    //获取类信息
    Class c = o.getClass();

    //获取类名称
    System.out.println("类名称：" + c.getName());

    //获取所有的public函数，包括从父类继承而来的
    Method[] methods = c.getMethods();
    for (Method method : methods){
      //获得方法名
      System.out.println("方法名：" + method.getName());

      //获得方法的返回类型
      Class returnType = method.getReturnType();
      System.out.println("返回类型：" + returnType.getName());

      //获得参数类型
      Class[] paramTypes = method.getParameterTypes();
      System.out.println("参数类型：");
      for (Class parmType : paramTypes){
        System.out.println(parmType.getName());
      }
    }

    //获得所有成员变量
    //getDeclaredFields获得的是该类自己声明的变量的信息
    //getFields获得的是所有public的成员变量的信息
    Field[] fields = c.getDeclaredFields();
    System.out.println("成员变量：");
    for (Field field : fields){
      Class fieldType = field.getType();
      System.out.println(fieldType.getName() + " " + field.getName());
    }

    //获得构造函数的信息
    Constructor[] constructors = c.getDeclaredConstructors();
    for (Constructor constructor : constructors){
      System.out.println("构造函数：" + constructor.getName());

      //获取构造函数的参数列表
      System.out.println("参数类型：");
      Class[] cparamTypes = constructor.getParameterTypes();
      for (Class cparamType : cparamTypes){
        System.out.println(cparamType.getName());
      }
    }
  }
}
```

测试运行：

```java
public static void main(String args[]){
  Student student = new Student();ClassMessage.getClassMessage(student);
}
```

输出：

```java
类名称：Student
方法名：getName
返回类型：java.lang.String
参数类型：
方法名：getId
返回类型：java.lang.String
参数类型：
方法名：setName
返回类型：void
参数类型：
java.lang.String
方法名：setId
返回类型：void
参数类型：
java.lang.String
方法名：setAge
返回类型：void
参数类型：
int
方法名：getAge
返回类型：int
参数类型：
方法名：wait
返回类型：void
参数类型：
方法名：wait
返回类型：void
参数类型：
long
int
方法名：wait
返回类型：void
参数类型：
long
方法名：equals
返回类型：boolean
参数类型：
java.lang.Object
方法名：toString
返回类型：java.lang.String
参数类型：
方法名：hashCode
返回类型：int
参数类型：
方法名：getClass
返回类型：java.lang.Class
参数类型：
方法名：notify
返回类型：void
参数类型：
方法名：notifyAll
返回类型：void
参数类型：
成员变量：
java.lang.String name
int age
java.lang.String id
构造函数：Student
构造函数：Student
java.lang.String
java.lang.String

Process finished with exit code 0

```

## 注意！

在`getDeclaredFields()`方法中，有这么一句注释：

```
* <p> The elements in the returned array are not sorted and are not in any
* particular order.
```

这个注释告诉我们，用户不要在代码中依赖这些方法返回的顺序。它“不保证返回的顺序是怎样的”。这是因为在Java编译器与JVM中都有权利对Java类的字段做重排序。详细的信息可以参考：[https://www.zhihu.com/question/52856385](https://www.zhihu.com/question/52856385)。