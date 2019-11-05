---
title: 使用Orika实现Java Bean映射
date: 2019-05-29 12:00:00
tags: 
    - Bean映射
    - Orika
    - Java
categories: 
    - java
notshow: true
---

Orika是Java Bean映射框架，可以实现从一个对象递归拷贝数据至另一个对象。这样我们可以将数据在实体，DTO，VO之间切换。关于Bean映射，其实BeanUtil中的copyProperties()方法也可以实现，但是如果名字相同类型不同的的是不能直接复制的，而orika却可以解决这个问题。
<!-- more -->
# 1.引入orika的jar包
现在用的大部分都是使用的maven文件，可以在pom.xml文件中引入下列配置即可。
```xml
<dependency>
    <groupId>ma.glasnost.orika</groupId>
    <artifactId>orika-core</artifactId>
    <version>1.5.4</version>
</dependency>
```

# 2.创建映射工具类
之前创建没有使用单例，造成了内存溢出的情况，后来采用单例的模式，解决了这个问题。

```java
package com.jiafly.libra.common.utils;

import ma.glasnost.orika.MapperFacade;
import ma.glasnost.orika.MapperFactory;
import ma.glasnost.orika.impl.DefaultMapperFactory;
import ma.glasnost.orika.metadata.ClassMapBuilder;

import java.util.Collection;
import java.util.List;
import java.util.Map;
import java.util.Objects;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 映射工具类
 *
 * @author 东邪Jiafly
 */
public enum MapperUtils {

    /**
     * 实例
     */
    INSTANCE;

    /**
     * 默认字段工厂
     */
    private static final MapperFactory MAPPER_FACTORY = new DefaultMapperFactory.Builder().build();

    /**
     * 默认字段实例
     */
    private static final MapperFacade MAPPER_FACADE = MAPPER_FACTORY.getMapperFacade();

    /**
     * 默认字段实例集合
     */
    private static Map<String, MapperFacade> CACHE_MAPPER_FACADE_MAP = new ConcurrentHashMap<>();

    /**
     * 映射实体（默认字段）
     *
     * @param toClass 映射类对象
     * @param data    数据（对象）
     * @return 映射类对象
     */
    public <E, T> E map(Class<E> toClass, T data) {
        return MAPPER_FACADE.map(data, toClass);
    }

    /**
     * 映射实体（自定义配置）
     *
     * @param toClass   映射类对象
     * @param data      数据（对象）
     * @param configMap 自定义配置
     * @return 映射类对象
     */
    public <E, T> E map(Class<E> toClass, T data, Map<String, String> configMap) {
        MapperFacade mapperFacade = this.getMapperFacade(toClass, data.getClass(), configMap);
        return mapperFacade.map(data, toClass);
    }

    /**
     * 映射集合（默认字段）
     *
     * @param toClass 映射类对象
     * @param data    数据（集合）
     * @return 映射类对象
     */
    public <E, T> List<E> mapAsList(Class<E> toClass, Collection<T> data) {
        return MAPPER_FACADE.mapAsList(data, toClass);
    }


    /**
     * 映射集合（自定义配置）
     *
     * @param toClass   映射类
     * @param data      数据（集合）
     * @param configMap 自定义配置
     * @return 映射类对象
     */
    public <E, T> List<E> mapAsList(Class<E> toClass, Collection<T> data, Map<String, String> configMap) {
        T t = data.stream().findFirst().orElseThrow(() -> new ResourceNotExistException("映射集合，数据集合为空"));
        MapperFacade mapperFacade = this.getMapperFacade(toClass, t.getClass(), configMap);
        return mapperFacade.mapAsList(data, toClass);
    }

    /**
     * 获取自定义映射
     *
     * @param toClass   映射类
     * @param dataClass 数据映射类
     * @param configMap 自定义配置
     * @return 映射类对象
     */
    private <E, T> MapperFacade getMapperFacade(Class<E> toClass, Class<T> dataClass, Map<String, String> configMap) {
        String mapKey = dataClass.getCanonicalName() + "_" + toClass.getCanonicalName();
        MapperFacade mapperFacade = CACHE_MAPPER_FACADE_MAP.get(mapKey);
        if (Objects.isNull(mapperFacade)) {
            MapperFactory factory = new DefaultMapperFactory.Builder().build();
            ClassMapBuilder classMapBuilder = factory.classMap(dataClass, toClass);
            configMap.forEach(classMapBuilder::field);
            classMapBuilder.byDefault().register();
            mapperFacade = factory.getMapperFacade();
            CACHE_MAPPER_FACADE_MAP.put(mapKey, mapperFacade);
        }
        return mapperFacade;
    }
}
```

# 3.使用映射工具类
这个工具列中主要有四个方法来根据不同需求去映射对象。
- **map(Class<E> toClass, T data)**
这个是普通的映射实体，主要映射命名相同的默认字段。

- **map(Class<E> toClass, T data, Map<String, String> configMap)**
这是个自定义配置的映射集合，当name
- **mapAsList(Class<E> toClass, Collection<T> data)**
这个是针对集合的映射
- **mapAsList(Class<E> toClass, Collection<T> data, Map<String, String> configMap)**
这是针对集合配置并且自定义配置

# 4.结语
这篇主要是针对使用Orika的使用写了一个工具类，以方便我们在需要的时候直接使用，当然如果想对Orika有一些更高级的使用，可以继续阅读Orika的源码，分析它具体的实现原理。


