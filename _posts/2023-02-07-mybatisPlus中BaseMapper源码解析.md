---
layout: post
title: 'BaseMapper源码解析'
date: 2023-02-01
description: 'BaseMapper源码解析'
tags: '技术'
--- 

## mybatisPlus v3.5.3.1
看到这篇文章,说明你对`IService接口`有了了解。`IService`是`mybatisPlus`提供的一个接口,该接口包含了一些封装的`ServiceCRUD`,进一步封装`CRUD`采用`get查询单行``remove删除``list查询``page分页`浅醉命名方式区分`Mapper`层`BaseMapper`避免混淆。

[IService](https://gitee.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/IService.java)

## BaseMapper接口
首先我们看一下作者是如何定义这个接口的
```java
/**
 * Mapper 继承该接口后，无需编写 mapper.xml 文件，即可获得CRUD功能
 * <p>这个 Mapper 支持 id 泛型</p>
 *
 * @author hubin
 * @since 2016-01-23
 */
```
可以看到作者十分的谦虚,并不像写IService接口时那样猖狂(说IService是顶级Service接口)。

`Java语言`我们了解,接口里面的变量只能为常量,方法只能是抽象方法,我们不能在接口中写方法的具体实现操作。`Java8.0`之后,官方更新新增了默认方法和静态方法,也就是说`java8`以后,我们可以在接口中写具体的方法和方法的实现。

我讲这个的原因是因为`BaseMapper接口`中有各种各样的方法,抽象方法、默认方法和静态方法。我们提前了解每个方法是什么样子的,这样有助于我们接下来解析源码。

这里我也主要是对`BaseMapper接口`中方法的解析,方法总的有19个。

| 类型 | 方法个数 |
|:---:|:---:|
|插入方法|1|
|删除方法|5|
|修改方法|2|
|查询方法|10|
|其他判断方法|1|

可以看到查询方法占了所有方法的一半,看来查询很重要啊

### 1.插入方法
#### 1.1 插入一条记录

