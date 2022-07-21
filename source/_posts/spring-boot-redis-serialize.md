---
title: SpringBoot 中 RedisTemplate 序列化问题若干
tags:
  - spring boot
  - redis
  - 序列化
date: 2022-07-11 17:15:16
---

`Spring Boot (2.5.14)` 默认使用的是 `Jackson` 来做序列化操作，在使用 `redis stream` 的过程中，暴露出了一些问题，遂做整理。  

## `StringRedisTemplate` 和 `RedisTemplate`
`spring-boot-starter-data-redis` 默认注入了两个 `bean`:
```java
package org.springframework.boot.autoconfigure.data.redis;

@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

}
```
<!-- more -->

在大部分情况下都是使用的 `StringRedisTemplate`，而且所有的序列化器都是 `RedisSerializer.string()`。

```java
package org.springframework.data.redis.core;

public class StringRedisTemplate extends RedisTemplate<String, String> {

    public StringRedisTemplate() {
        setKeySerializer(RedisSerializer.string());
        setValueSerializer(RedisSerializer.string());
        setHashKeySerializer(RedisSerializer.string());
        setHashValueSerializer(RedisSerializer.string());
    }
    ...
}
```
但是在操作特殊的值时（即 `RedisTemplate<Object, Object>`），会有一些特殊的问题。

`RedisTemplate` 默认序列化工具为 `JdkSerializationRedisSerializer`，会将所有的key和value转换为二进制，而且对于需要序列化的类，要实现接口 `Serializable`，才能被正确的序列化。如果不关心在 `redis` 中存储的内容（无需查看等），那么其他配置可能使用默认。

如果需要经常查看甚至修改 `redis` 的内容，就需要配置 `redisTemplate`。
```java
@Configuration
public class RedisConfig {
    ...
    private final RedisTemplate<Object, Object> redisTemplate;

    /**
     * {@link org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration#redisTemplate}
     */
    @Autowired
    public RedisConfig(RedisTemplate<Object, Object> redisTemplate, ObjectMapper objectMapper) {
        this.jsonRedisSerializer = obtainJsonRedisSerializer(objectMapper);

        // 设置redis的序列化工具
        redisTemplate.setKeySerializer(RedisSerializer.string());
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        redisTemplate.setValueSerializer(this.jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(this.jsonRedisSerializer);
    }

    private GenericJackson2JsonRedisSerializer obtainJsonRedisSerializer(ObjectMapper objectMapper) {
        ObjectMapper objectMapperCopy = objectMapper.copy();
        // objectMapperCopy.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        objectMapperCopy.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,
                                               ObjectMapper.DefaultTyping.NON_FINAL,
                                               JsonTypeInfo.As.PROPERTY
        );
        return new GenericJackson2JsonRedisSerializer(objectMapperCopy);
    }
    ...
}
```
如以上代码所示：  
1. 使用 `Spring Boot` 的 `objectMapper` 做为基础配置，生成 `GenericJackson2JsonRedisSerializer`  
   如果使用 `new ObjectMapper()`，会有部分模块未注册问题，如（java8的LocalDateTime, jsr310等）  
   使用 `objectMapper.copy()` 是为了防止污染原始bean
2. `objectMapper` 默认不添加类信息，当从缓存中取出进行反序列化时，`GenericJackson2JsonRedisSerializer` 不知道该反序列为什么类，因此添加这个配置，不同的配置生成不同的序列化字符串格式：
```java
    // 这样会作为属性添加，且属性名为 "@class"
    // objectMapperCopy.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);  // 这个方法已经被弃用
    objectMapperCopy.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,
                                            ObjectMapper.DefaultTyping.NON_FINAL,
                                            JsonTypeInfo.As.PROPERTY);
```
其他格式看 `activateDefaultTyping` 的几个重载方法

## `Redis Stream` MQ
在使用 `Redis Stream` 做简单消息队列时，如果不关心存储的数据格式，全部使用 `StringRedisTemplate` 即可
