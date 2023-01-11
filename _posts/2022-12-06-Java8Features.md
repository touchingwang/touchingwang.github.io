---
layout: post
title: 'Java接口中的default方法（Java8新特性）'
date: 2022-12-06
description: 'Java8新特性'
tags: '技术'
--- 

### Java接口中的default方法

在Java8以后,接口中可以添加使用default或者static修饰的方法，在这里我们只讨论default方法,default修饰方法只能在接口中使用,在接口中被default标记的方法为普通方法,可以直接写方法体。

#### 特性1

**解释：** 实现类会继承接口中的default方法

```java
public class dog implements animal {
    public static void main(String[] args) {
        dog dog = new dog();
        dog.sleep();
    }
}

interface animal {
    default void sleep() {
        Console.log("sleeping");
    };
}
```

#### 特性2

**解释：** 如果一个类同时实现接口A和B,接口A和B中有相同的default方法,这是,该类必须重写接口中的default方法。

```java
public class dog implements animal,biology {
    public static void main(String[] args) {
        dog dog = new dog();
        dog.sleep();
    }

    @Override
    public void sleep() {
        biology.super.sleep();
    }
}

interface animal {
    default void sleep() {
        Console.log("sleeping");
    };
}
interface biology {
    default void sleep() {
        Console.log("resting");
    };
}
```

#### 特性3

**解释：** 如果子类继承父类,父类中有b方法,该子类同时实现的接口中也有b方法(被default修饰),那么子类会继承父类的b方法而不是继承接口中的b方法。

```java
public class dog extends animal implements biology {
    public static void main(String[] args) {
        dog dog = new dog();
        dog.sleep();
    }
}

abstract class animal implements biology2 {
    @Override
    public void sleep() {
        Console.log("sleep");
    }
}

interface biology {
    default void sleep() {
        Console.log("resting");
    }
}
interface biology2 {
    default void sleep() {}
}
```

<br>

转载请注明：[touchingwang的博客](http://touchingwang.github.io) » [点击阅读原文](http://https://github.com/touchingwang/touchingwang.github.io/tree/master/_posts/2022-12-07-Java8Features.md)
