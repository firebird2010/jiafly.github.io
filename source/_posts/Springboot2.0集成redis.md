---
title: SpringBoot2.0集成Redis(一)
date: 2016-12-26 20:16:13
tags: [SpringBoot,Redis]
category: [学习日记]
---

## 1.在pom.xml中添加如下依赖

```shell
		<!-- redis -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
		<!--spring2.0集成redis所需common-pool2-->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-pool2</artifactId>
			<version>2.5.0</version>
		</dependency>
		<!-- 序列化需要 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.28</version>
		</dependency>
```

## 2. 在application.yml中配置redis信息

```yaml
# redis相关配置
spring:
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
    password:
    timeout: 100ms
    lettuce:
      pool:
        max-active: 8
        max-wait: -1ms
        max-idle: 8
        min-idle: 0
```

## 3.定义 StringRedisTemplate ，指定序列化和反序列化的处理类

```java
package com.jiafly.libra.common.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

/**
 * @Author dongxie
 * @Email liuyi1181@163.com
 * @Date 2018/9/13 下午5:01
 * @Description redis 配置类
 */
@Configuration
public class RedisConfig {

    /**
     * 定义 StringRedisTemplate ，指定序列化和反序列化的处理类
     * @param factory
     * @return
     */
    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate template = new StringRedisTemplate(factory);
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(
                Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        // 序列化值时使用此序列化方法
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}
```

## 4.测试

```java
package com.jiafly.libra.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


/**
 * @Author dongxie
 * @Email liuyi1181@163.com
 * @Date 2018/7/31 下午5:57
 * @Description
 */
@RestController
@RequestMapping("/redis")
@Slf4j
public class TestController {

    @Autowired
    RedisTemplate redisTemplate;

    @GetMapping("set/{key}/{value}")
    public String set(@PathVariable("key")String key, @PathVariable("value") String value) {
        //注意这里的 key不可以为null
        log.info("redis设置Key:{} 的value为{}", key, value);
        redisTemplate.opsForValue().set(key, value);
        return key + "," + value;
    }

    @GetMapping("get/{key}")
    public String get(@PathVariable("key") String key) {
        log.info("redis获取Key:{} 的value值", key);
        return "key=" + key + ",value=" + redisTemplate.opsForValue().get(key);
    }
}

```

## 5.浏览器发起请求

- 在浏览器输入一下链接设置key的值value

  ```bash
  http://localhost:8080/redis/set/key/value
  ```

- 在浏览器输入以下链接获取key的值value

  ```ba
  http://localhost:8080/redis/get/key
  ```
