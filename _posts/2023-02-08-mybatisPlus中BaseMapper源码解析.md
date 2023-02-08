---
layout: post
title: 'BaseMapper源码解析'
date: 2023-02-08
description: 'BaseMapper源码解析'
tags: '技术'
--- 

你会发现BaseMapper接口中的方法都很简洁,容易理解,有种大道至简的味道。

### mybatisPlus v3.5.3.1
看到这篇文章,说明你对`IService接口`有了了解。`IService`是`mybatisPlus`提供的一个接口,该接口包含了一些封装的`ServiceCRUD`,进一步封装`CRUD`采用`get查询单行``remove删除``list查询``page分页`浅醉命名方式区分`Mapper`层`BaseMapper`避免混淆。

[IService](https://gitee.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/IService.java)

### BaseMapper接口注解
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
|其他判断方法|2|

可以看到查询方法占了所有方法的一半,看来查询很重要啊

### 1.插入方法
```java
/**
  * 插入一条记录
  *
  * @param entity 实体对象
  */
int insert(T entity);
```
BaseMapper接口的insert方法是一个常见的抽象方法。里面做的操作我觉得只要是了解数据库的都知道,插入一条数据。

对于数据的操作BaseMapper接口只有这一种。

### 1.插入方法
#### 1.1 插入一条记录

### 1.删除方法
#### 2.1 根据 ID 删除
```java
/**
  * 根据 ID 删除
  *
  * @param id 主键ID
  */
int deleteById(Serializable id);

/**
  * 根据实体(ID)删除
  *
  * @param entity 实体对象
  * @since 3.4.4
  */
int deleteById(T entity);
```
常见、易理解的删除操作,一种根据id、一种根据实体对象。

#### 2.2 根据 columnMap 条件，删除记录
```java
/**
  * 根据 columnMap 条件，删除记录
  *
  * @param columnMap 表字段 map 对象
  */
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
```
抽象方法deleteByMap,对于这些方法的后缀命名规则如果是by后缀+类型,代表着参数,后缀直接加类型的是返回值类型

#### 2.3 根据 entity 条件，删除记录
```java
/**
  * 根据 entity 条件，删除记录
  *
  * @param queryWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）
  */
int delete(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```
如果你想有规则的批量删除数据,用这个删除方法无疑是最好的。

#### 2.4 删除（根据ID或实体 批量删除）
```java
/**
  * 删除（根据ID或实体 批量删除）
  *
  * @param idList 主键ID列表或实体列表(不能为 null 以及 empty)
  */
int deleteBatchIds(@Param(Constants.COLL) Collection<?> idList);
```
对于同一个表一次删除很多无规则的数据十分让人头疼,使用这种删除方法让你疼的轻点。

### 3. 修改方法
```java
/**
  * 根据 ID 修改
  *
  * @param entity 实体对象
  */
int updateById(@Param(Constants.ENTITY) T entity);

/**
  * 根据 whereEntity 条件，更新记录
  *
  * @param entity        实体对象 (set 条件值,可以为 null)
  * @param updateWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）
  */
int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER) Wrapper<T> updateWrapper);
```
想要修改数据,BaseMapper接口提供了两种食用的方法。
一种是通过主键id进行修改的updateById方法,一种是万能的update修改方法。

IService接口中的修改和BaseMapper接口中修改的前缀命名方式都是update

### 4.查询方法
#### 4.1 查询一条记录的2个方法
```java
/**
  * 根据 ID 查询
  *
  * @param id 主键ID
  */
T selectById(Serializable id);
    /**
  * 根据 entity 条件，查询一条记录
  * <p>查询一条记录，例如 qw.last("limit 1") 限制取一条记录, 注意：多条数据会报异常</p>
  *
  * @param queryWrapper 实体对象封装操作类（可以为 null）
  */
default T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper) {
    List<T> list = this.selectList(queryWrapper);
    // 抄自 DefaultSqlSession#selectOne
    if (list.size() == 1) {
        return list.get(0);
    } else if (list.size() > 1) {
        throw new TooManyResultsException("Expected one result (or null) to be returned by selectOne(), but found: " + list.size());
    } else {
        return null;
    }
}
```
selectById方法通过主键id查询一条记录。selectOne方法根据wrapper条件查询出一条结果,这里不仅在sql语句上加了limit语法限制(这里应该是通过反射在wrapper参数中链式拼接了limit语法),同时在该方法里对取出来的数据进行大小判断。

在返回的参数方面,查询出一条记录的参数类型都是实体对象。

#### 4.2 批量查询的5个方法
```java
/**
  * 查询（根据ID 批量查询）
  *
  * @param idList 主键ID列表(不能为 null 以及 empty)
  */
List<T> selectBatchIds(@Param(Constants.COLL) Collection<? extends Serializable> idList);
    /**
  * 查询（根据 columnMap 条件）
  *
  * @param columnMap 表字段 map 对象
  */
List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
/**
  * 根据 entity 条件，查询全部记录
  *
  * @param queryWrapper 实体对象封装操作类（可以为 null）
  */
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
    /**
  * 根据 Wrapper 条件，查询全部记录
  *
  * @param queryWrapper 实体对象封装操作类（可以为 null）
  */
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
    /**
  * 根据 Wrapper 条件，查询全部记录
  * <p>注意： 只返回第一个字段的值</p>
  *
  * @param queryWrapper 实体对象封装操作类（可以为 null）
  */
List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
    /**
  * 根据 entity 条件，查询全部记录（并翻页）
  *
  * @param page         分页查询条件（可以为 RowBounds.DEFAULT）
  * @param queryWrapper 实体对象封装操作类（可以为 null）
  */
<P extends IPage<T>> P selectPage(P page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
    /**
  * 根据 Wrapper 条件，查询全部记录（并翻页）
  *
  * @param page         分页查询条件
  * @param queryWrapper 实体对象封装操作类
  */
<P extends IPage<Map<String, Object>>> P selectMapsPage(P page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```
对于这些方法的后缀命名规则如果是by后缀+类型,代表着参数,后缀直接加类型的是返回值类型

有了这个命名规则我们一眼可以看出第二个方法是参数为map的批量查询,其他的都是返回值类型各式各样的批次查询。

### 5.其他方法2个
#### 5.1 根据 Wrapper 条件，查询总记录数
```java
/**
  * 根据 Wrapper 条件，查询总记录数
  *
  * @param queryWrapper 实体对象封装操作类（可以为 null）
  */
Long selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```
这个也是十分常用的一个判断。返回满足条件的查询记录数量。

#### 5.2 根据 Wrapper 条件，判断是否存在记录
```java
/**
  * 根据 Wrapper 条件，判断是否存在记录
  *
  * @param queryWrapper 实体对象封装操作类
  * @return 是否存在记录
  */
default boolean exists(Wrapper<T> queryWrapper) {
    Long count = this.selectCount(queryWrapper);
    return null != count && count > 0;
}
```
exists方法可以说是selectCount方法的一个扩展,也是很实用的。

### 6.总结
我推荐在开发的时候使用BaseMapper接口中的方法。推荐的理由有以下几点：
1. BaseMapper接口中的方法更接近数据库语言。
2. IService接口中的方法大部分都是继承BaseMapper接口中的方法。同样的方法使用BaseMapper接口中方法都可以实现。
3. 如果说将来mybatisPlus对代码进行了大量重构,BaseMapper接口处于IService接口上面,受到的影响会更小。
