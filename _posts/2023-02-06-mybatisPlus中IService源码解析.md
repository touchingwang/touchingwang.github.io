---
layout: post
title: 'Iservice源码解析'
date: 2023-02-01
description: 'Iservice源码解析'
tags: '技术'
--- 

### mybatisPlus v3.5.3.1
看到这篇文章,说明你对`IService接口`有了了解。`IService`是`mybatisPlus`提供的一个接口,该接口包含了一些封装的`ServiceCRUD`,进一步封装`CRUD`采用`get查询单行``remove删除``list查询``page分页`浅醉命名方式区分`Mapper`层`BaseMapper`避免混淆。

[IService](https://gitee.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/IService.java)

### IService接口
打开源码我们可以看到,作者称该接口为顶级Service,泛型 T 为任意实体对象
```java
/**
 * 顶级 Service
 *
 * @author hubin
 * @since 2018-06-23
 */
```

`Java语言`我们了解,接口里面的变量只能为常量,方法只能是抽象方法,我们不能在接口中写方法的具体实现操作。`Java8.0`之后,官方更新新增了默认方法和静态方法,也就是说`java8`以后,我们可以在接口中写具体的方法和方法的实现。

我讲这个的原因是因为`IService接口`中有各种各样的方法,抽象方法、默认方法和静态方法。我们提前了解每个方法是什么样子的,这样有助于我们接下来解析源码。

这里我也主要是对`IService接口`中方法的解析,所有的方法主要分为四类:CRUD。

### 1.插入方法
IService接口中的插入方法有四个：
#### 1.1 插入一条记录
```java
/**
  * 插入一条记录（选择字段，策略插入）
  *
  * @param entity 实体对象
  */
default boolean save(T entity) {
    return SqlHelper.retBool(getBaseMapper().insert(entity));
}
```
`save默认方法`是调用了当前接口中`getBaseMapper()`的`insert方法`。

getBaseMapper()又是什么呢？
```java
/**
  * 获取对应 entity 的 BaseMapper
  *
  * @return BaseMapper
  */
BaseMapper<T> getBaseMapper();
```
`IService接口`中有一个抽象方法,返回的是一个`当前实体(T)的BaseMapper`,也就是说我们直接调用`getBaseMapper()抽象方法`就相当于实现了`BaseMappe`r,我们就可以使用`BaseMapper接口`中的方法了。


#### 1.2 插入（批量）
```java
    /**
     * 插入（批量）
     *
     * @param entityList 实体对象集合
     */
    @Transactional(rollbackFor = Exception.class)
    default boolean saveBatch(Collection<T> entityList) {
        return saveBatch(entityList, DEFAULT_BATCH_SIZE);
    }

    /**
     * 插入（批量）
     *
     * @param entityList 实体对象集合
     * @param batchSize  插入批次数量
     */
    boolean saveBatch(Collection<T> entityList, int batchSize);
```
`saveBatch默认方法`则是调用当前接口中的`saveBatch抽象方法`,多了一层嵌套。

#### 1.3 批量修改插入
```java
/**
  * 批量修改插入
  *
  * @param entityList 实体对象集合
  */
@Transactional(rollbackFor = Exception.class)
default boolean saveOrUpdateBatch(Collection<T> entityList) {
    return saveOrUpdateBatch(entityList, DEFAULT_BATCH_SIZE);
}

/**
  * 批量修改插入
  *
  * @param entityList 实体对象集合
  * @param batchSize  每次的数量
  */
boolean saveOrUpdateBatch(Collection<T> entityList, int batchSize);
```
`saveOrUpdateBatch`方法和`saveBatch`方法的实现类似,加了一层嵌套;当前接口的默认方法调用当前接口的抽象方法。

#### 2.1 根据ID删除
```java
/**
  * 根据 ID 删除
  *
  * @param id 主键ID
  */
default boolean removeById(Serializable id) {
    return SqlHelper.retBool(getBaseMapper().deleteById(id));
}

/**
  * 根据 ID 删除
  *
  * @param id      主键(类型必须与实体类型字段保持一致)
  * @param useFill 是否启用填充(为true的情况,会将入参转换实体进行delete删除)
  * @return 删除结果
  * @since 3.5.0
  */
default boolean removeById(Serializable id, boolean useFill) {
    throw new UnsupportedOperationException("不支持的方法!");
}

/**
  * 根据实体(ID)删除
  *
  * @param entity 实体
  * @since 3.4.4
  */
default boolean removeById(T entity) {
    return SqlHelper.retBool(getBaseMapper().deleteById(entity));
}
```
`removeById`是重载的两个默认方法,`带参数的useFill`已经抛弃了,可以看到。
只有一个`参数id`的`removeById`调用`当前实体的BaseMapper`接口中方法,来实现根据`ID`删除。
`参数为实体的removeById`默认方法和`仅带参数id的removeById`都用过调用`当前实体的BaseMapper`接口中方法,来实现根据`ID`删除。

#### 2.2 根据 columnMap 条件，删除记录
```java
/**
  * 根据 columnMap 条件，删除记录
  *
  * @param columnMap 表字段 map 对象
  */
default boolean removeByMap(Map<String, Object> columnMap) {
    Assert.notEmpty(columnMap, "error: columnMap must not be empty");
    return SqlHelper.retBool(getBaseMapper().deleteByMap(columnMap));
}
```
`removeByMap`抽象方法通过`参数Map`来进行条件删除。同样调用`BaseMapper`接口中方法

#### 2.3 根据 entity 条件，删除记录
```java
/**
  * 根据 entity 条件，删除记录
  *
  * @param queryWrapper 实体包装类 {@link com.baomidou.mybatisplus.core.conditions.query.QueryWrapper}
  */
default boolean remove(Wrapper<T> queryWrapper) {
    return SqlHelper.retBool(getBaseMapper().delete(queryWrapper));
}
```
`remove`抽象方法调用`BaseMapper`中参数为`Wrapper<T>`的`delete`方法。

#### 2.4 删除（根据ID 批量删除）
```java
/**
  * 删除（根据ID 批量删除）
  *
  * @param list 主键ID或实体列表
  */
default boolean removeByIds(Collection<?> list) {
    if (CollectionUtils.isEmpty(list)) {
        return false;
    }
    return SqlHelper.retBool(getBaseMapper().deleteBatchIds(list));
}
/**
  * 批量删除
  *
  * @param list    主键ID或实体列表
  * @param useFill 是否填充(为true的情况,会将入参转换实体进行delete删除)
  * @return 删除结果
  * @since 3.5.0
  */
@Transactional(rollbackFor = Exception.class)
default boolean removeByIds(Collection<?> list, boolean useFill) {
    if (CollectionUtils.isEmpty(list)) {
        return false;
    }
    if (useFill) {
        return removeBatchByIds(list, true);
    }
    return SqlHelper.retBool(getBaseMapper().deleteBatchIds(list));
}
```
`removeByIds`抽象方法,通过集合(主键id的集合)对数据进行批量删除,调用了`BaseMapper`接口中`deleteBatchIds`方法
带参数useFill的removeByIds抽象方法和一个参数的removeByIds抽象方法相比,多了一个判断的操作。

#### 2.5 批量删除(jdbc批量提交)**(作者已抛弃)**
```java
/**
  * 批量删除(jdbc批量提交)
  *
  * @param list 主键ID或实体列表(主键ID类型必须与实体类型字段保持一致)
  * @return 删除结果
  * @since 3.5.0
  */
@Transactional(rollbackFor = Exception.class)
default boolean removeBatchByIds(Collection<?> list) {
    return removeBatchByIds(list, DEFAULT_BATCH_SIZE);
}

/**
  * 批量删除(jdbc批量提交)
  *
  * @param list    主键ID或实体列表(主键ID类型必须与实体类型字段保持一致)
  * @param useFill 是否启用填充(为true的情况,会将入参转换实体进行delete删除)
  * @return 删除结果
  * @since 3.5.0
  */
@Transactional(rollbackFor = Exception.class)
default boolean removeBatchByIds(Collection<?> list, boolean useFill) {
    return removeBatchByIds(list, DEFAULT_BATCH_SIZE, useFill);
}
/**
  * 批量删除(jdbc批量提交)
  *
  * @param list      主键ID或实体列表
  * @param batchSize 批次大小
  * @return 删除结果
  * @since 3.5.0
  */
default boolean removeBatchByIds(Collection<?> list, int batchSize) {
    throw new UnsupportedOperationException("不支持的方法!");
}

/**
  * 批量删除(jdbc批量提交)
  *
  * @param list      主键ID或实体列表
  * @param batchSize 批次大小
  * @param useFill   是否启用填充(为true的情况,会将入参转换实体进行delete删除)
  * @return 删除结果
  * @since 3.5.0
  */
default boolean removeBatchByIds(Collection<?> list, int batchSize, boolean useFill) {
    throw new UnsupportedOperationException("不支持的方法!");
}
```
`removeBatchByIds`的所有重载方法已经去除了,反正原来什么样我也没用过,`removeBatchByIds`的方法就五个套着套着发现最后被抛出异常了，这是作者把这种方法抛弃了。

### 3.修改
#### 3.1 根据 ID 选择修改
```java
/**
  * 根据 ID 选择修改
  *
  * @param entity 实体对象
  */
default boolean updateById(T entity) {
    return SqlHelper.retBool(getBaseMapper().updateById(entity));
}
```
很简单的一个修改调用`BaseMapper方法`

#### 3.2 根据 whereEntity 条件，更新记录 **(有意思的方法)**
```java
/**
  * 根据 UpdateWrapper 条件，更新记录 需要设置sqlset
  *
  * @param updateWrapper 实体对象封装操作类 {@link com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper}
  */
default boolean update(Wrapper<T> updateWrapper) {
    return update(null, updateWrapper);
}

/**
  * 根据 whereEntity 条件，更新记录
  *
  * @param entity        实体对象
  * @param updateWrapper 实体对象封装操作类 {@link com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper}
  */
default boolean update(T entity, Wrapper<T> updateWrapper) {
    return SqlHelper.retBool(getBaseMapper().update(entity, updateWrapper));
}
```
`只有一个参数的update默认方法`是一个普通的`wrapper`包装的条件更新方法,我们会发现该方法调用的是它的另一个重载方法。我们直接分析这个方法就行。

我根据也就是这个方法有点意思。如果你不经常使用这个方法,刚一看到这个方法的参数会有一脸的蒙蔽,怎么有`wrapper`条件了还要加个实体。

`update方法`的`updateWrapper参数`是起到一个筛选作用，比如查询所有性别为男的学生，而另一个`entity`是要更改的结果，比如我在`entity`中设置了年龄为`10`,则所有满足`updateWrapper`条件的学生年龄都改成了`10`。

#### 3.3 根据ID 批量更新
```java
/**
  * 根据ID 批量更新
  *
  * @param entityList 实体对象集合
  */
@Transactional(rollbackFor = Exception.class)
default boolean updateBatchById(Collection<T> entityList) {
    return updateBatchById(entityList, DEFAULT_BATCH_SIZE);
}

/**
  * 根据ID 批量更新
  *
  * @param entityList 实体对象集合
  * @param batchSize  更新批次数量
  */
boolean updateBatchById(Collection<T> entityList, int batchSize);
```
`updateBatchById方法`和`update方法`相比就逊色了许多,只能根据`id`进行批量更新。`update方法`可以根据任何条件进行批量更新。

#### 3.4 TableId 注解存在更新记录，否插入一条记录
```java
/**
  * TableId 注解存在更新记录，否插入一条记录
  *
  * @param entity 实体对象
  */
boolean saveOrUpdate(T entity);

/**
  * <p>
  * 根据updateWrapper尝试更新，否继续执行saveOrUpdate(T)方法
  * 此次修改主要是减少了此项业务代码的代码量（存在性验证之后的saveOrUpdate操作）
  * </p>
  *
  * @param entity 实体对象
  */
default boolean saveOrUpdate(T entity, Wrapper<T> updateWrapper) {
    return update(entity, updateWrapper) || saveOrUpdate(entity);
}
```
个人感觉这是一个最鸡肋的方法。很多方法,我建议还是使用底层的,如果人家封装的方法有你认为十分优秀的,不要习惯去使用它,你要做的是去了解它是如何用底层方法封装的,你在使用的时候直接用底层的方法去写。

### 4.查询
#### 4.1 BaseMapper查询封装
```java
/**
  * 根据 ID 查询
  *
  * @param id 主键ID
  */
default T getById(Serializable id) {
    return getBaseMapper().selectById(id);
}

/**
  * 查询（根据ID 批量查询）
  *
  * @param idList 主键ID列表
  */
default List<T> listByIds(Collection<? extends Serializable> idList) {
    return getBaseMapper().selectBatchIds(idList);
}

/**
  * 查询（根据 columnMap 条件）
  *
  * @param columnMap 表字段 map 对象
  */
default List<T> listByMap(Map<String, Object> columnMap) {
    return getBaseMapper().selectByMap(columnMap);
}
  /**
    * 根据 Wrapper 条件，查询总记录数
    *
    * @param queryWrapper 实体对象封装操作类 {@link com.baomidou.mybatisplus.core.conditions.query.QueryWrapper}
    */
  default long count(Wrapper<T> queryWrapper) {
      return SqlHelper.retCount(getBaseMapper().selectCount(queryWrapper));
  }
```
getById(),listByIds,listByMap(),count(),list(),page(),listMaps(),pageMaps()这些查询方法都是封装BaseMapper接口中方法直接使用的,我建议这些方法就不要使用了,你要使用就直接用BaseMapper接口中方法就行

#### 4.2 根据 Wrapper，查询一条记录
```java
/**
* 根据 Wrapper，查询一条记录 <br/>
* <p>结果集，如果是多个会抛出异常，随机取一条加上限制条件 wrapper.last("LIMIT 1")</p>
*
* @param queryWrapper 实体对象封装操作类 {@link com.baomidou.mybatisplus.core.conditions.query.QueryWrapper}
*/
default T getOne(Wrapper<T> queryWrapper) {
    return getOne(queryWrapper, true);
}

/**
* 根据 Wrapper，查询一条记录
*
* @param queryWrapper 实体对象封装操作类 {@link com.baomidou.mybatisplus.core.conditions.query.QueryWrapper}
* @param throwEx      有多个 result 是否抛出异常
*/
T getOne(Wrapper<T> queryWrapper, boolean throwEx);

/**
  * 根据 Wrapper，查询一条记录
  *
  * @param queryWrapper 实体对象封装操作类 {@link com.baomidou.mybatisplus.core.conditions.query.QueryWrapper}
  */
Map<String, Object> getMap(Wrapper<T> queryWrapper);

/**
  * 根据 Wrapper，查询一条记录
  *
  * @param queryWrapper 实体对象封装操作类 {@link com.baomidou.mybatisplus.core.conditions.query.QueryWrapper}
  * @param mapper       转换函数
  */
<V> V getObj(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
```
IService查询很经典的方法。getOne(),getMap(),getObj()这些方法的不同是参数和返回值为不同类型的。

还有一些可以进行链式查询更改的方法,这个比较深入我也就没看了。

### 5.总结
以上就是IService接口中的方法解析,我平时开发的时候更多使用的是BaseMapper接口,IService接口和BaseMapper接口相比也有很多优秀的方法,比如说更新的update()方法,可以说能解决大部分的业务更新需求。
