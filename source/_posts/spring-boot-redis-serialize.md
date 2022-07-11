---
title: SpringBoot 中 RedisTemplate 序列化问题若干
tags:
  - spring boot
  - redis
  - 序列化
date: 2022-07-11 17:15:16
---


`Spring Boot` 默认使用的是 `Jackson` 来做序列化操作，在使用 `redis stream` 的过程中，暴露出了一些问题。  
在大部分情况下都是使用的 `StringRedisTemplate`, 也就是 `RedisTemplate<String, String>`，但是在操作特殊的值时（即 `<V> RedisTemplate<String, V>`），会有一些特殊的问题。

`RedisTemplate` 默认序列化工具为 `JdkSerializationRedisSerializer`，会将所有的key和value转换为二进制，而且对于需要序列化的类，要实现接口 `Serializable`，才能被正确的序列化。如果不关心在 `redis` 中存储的内容（无需查看等），那么其他配置可能使用默认。

<!-- more -->
如果需要经常查看 `redis` 内容，就需要配置 `redisTemplate`，本文代码全部都在 `Spring Boot 2.5.14` 版本下
```java
@Configuration
public class RedisConfig {
    ...
    private final RedisTemplate<Object, Object> redisTemplate;
    private final ObjectMapper objectMapper;

    @Autowired
    public RedisConfig(RedisTemplate<Object, Object> redisTemplate, ObjectMapper objectMapper) {
        this.redisTemplate = redisTemplate;
        this.objectMapper = objectMapper;
    }

    @PostConstruct
    public void init() {
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        redisTemplate.setHashValueSerializer(RedisSerializer.string());
        redisTemplate.setKeySerializer(RedisSerializer.string());
        // 1. 使用了 Spring Boot 自己的 ObjectMapper 作为基础设置
        ObjectMapper objectMapperCopy = objectMapper.copy();
        // 2. 添加序列化的类信息，保证可以反序列化
        objectMapperCopy.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL);
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer(objectMapperCopy);
        redisTemplate.setValueSerializer(jsonRedisSerializer);
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
    objectMapperCopy.activateDefaultTypingAsProperty(LaissezFaireSubTypeValidator.instance,
                                                     ObjectMapper.DefaultTyping.NON_FINAL,
                                                     "@class");
```
其他格式看 `activateDefaultTyping` 的几个重载方法
