---
title: h2内置数据库的简单使用
date: 2021-08-03 14:33:24
tags: 
  - java
  - db
---
有时想做一些简单demo，需要使用到数据库，但为此安装一个mysql之类的数据库又过于庞大，H2内置数据库正适合这样的工作。  
基于Springboot：
### 添加依赖
```xml
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
```
<!-- more -->

### application配置
```yml
spring:
  datasource:
    url: jdbc:h2:mem:test # 内存内数据库
    driver-class-name: org.h2.Driver
    username: sa
    password:
    # 要同时有这两个配置，sql才会导入？
    schema: classpath:db/schema.sql
    data: classpath:db/data.sql
```
### 添加初始化库的sql
如上一步的配置中，好像需要写明schema才会去创建库表：
```sql
-- classpath:db/schema.sql
DROP TABLE IF EXISTS billionaires;

CREATE TABLE billionaires
(
    id         INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(250) NOT NULL,
    last_name  VARCHAR(250) NOT NULL,
    career     VARCHAR(250) DEFAULT NULL
);

-- classpath:db/data.sql
INSERT INTO billionaires (first_name, last_name, career)
VALUES ('Aliko', 'Dangote', 'Billionaire Industrialist'),
       ('Bill', 'Gates', 'Billionaire Tech Entrepreneur'),
       ('Folrunsho', 'Alakija', 'Billionaire Oil Magnate');
```

配置完成，之后的开发和正常连接数据库相同，需要注意每次停止项目后，所有的数据库改动都会丢失。

TODO: 持久化