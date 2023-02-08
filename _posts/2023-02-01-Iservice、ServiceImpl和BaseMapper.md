---
layout: post
title: 'Iservice、ServiceImpl和BaseMapper'
date: 2023-02-01
description: 'Iservice、ServiceImpl和BaseMapper'
tags: '技术'
--- 

### IService CRUD接口
- 通用 `Service CRUD` 封装`IService`接口，进一步封装 CRUD 采用 `get 查询单行``remove 删除``list 查询集``page 分页`前缀命名方式区分 `Mapper` 层避免混淆，
- 泛型 T 为任意实体对象
- 建议如果存在自定义通用 Service 方法的可能，请创建自己的 `IBaseService` 继承 `Mybatis-Plus` 提供的基类
- 对象 `Wrapper` 为 条件构造器

如果想详细了解`Service层`的`Iservice接口`,可以看[MybatisPlus中IService接口源码解析]()一篇文章

### BaseMapper CRUD接口
- 通用 CRUD 封装`BaseMapper`接口，为 `Mybatis-Plus` 启动时自动解析实体表关系映射转换为 `Mybatis` 内部对象注入容器
- 泛型 `T` 为任意实体对象
- 参数 `Serializable` 为任意类型主键 `Mybatis-Plus` 不推荐使用复合主键约定每一张表都有自己的唯一 `id` 主键
- 对象 `Wrapper` 为 条件构造器

如果想详细了解`Service层`的`Iservice接口`,可以看[MybatisPlus中BaseMapper接口源码解析]()一篇文章

### ServiceImpl实现类
- 该类主要实现了IService接口中一些方法,同时增加了一些事务级别操作的方法
