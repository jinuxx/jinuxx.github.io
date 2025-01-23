---
title: Mybatis Plus 相关
tags:
  - java
date: 2025-01-22 16:23:35
---


### 记录使用 Mybatis 过程中的一些问题

#### [ActiveRecord](https://baomidou.com/guides/data-interface/#activerecord)

现在只要继承 `Model` 类，就可以获得 `CRUD` 的能力。

> 必须确保项目中已经有对应实体的 `BaseMapper`

<!-- more -->

##### 继承

```java
import com.baomidou.mybatisplus.extension.activerecord.Model;

public class User extends Model<User> {
    // 实体类的字段定义...
    private Long id;
    private String name;
    private Integer age;
    // ... 其他字段和 getter/setter 方法
}
```

##### 调用

```java
// 创建新用户并插入数据库
User user = new User();
user.setName("John Doe");
user.setAge(30);
boolean isInserted = user.insert(); // 返回值表示操作是否成功

// 查询所有用户
List<User> allUsers = user.selectAll();

// 根据 ID 更新用户信息
user.setId(1L);
user.setName("Updated Name");
boolean isUpdated = user.updateById(); // 返回值表示操作是否成功

// 根据 ID 删除用户
boolean isDeleted = user.deleteById(); // 返回值表示操作是否成功
```

#### [Db Kit](https://baomidou.com/guides/data-interface/#db-kit)

可以通过 `Db.xxx` 方法直接调用 sql

> 必须确保项目中已经有对应实体的 `BaseMapper`

##### 示例

```java
// 假设有一个 User 实体类和对应的 BaseMapper

// 根据 id 查询单个实体
User user = Db.getById(1L, User.class);

// 根据 id 查询多个实体
List<User> userList = Db.listByIds(Arrays.asList(1L, 2L, 3L), User.class);

// 根据条件构造器查询
LambdaQueryWrapper<User> queryWrapper = Wrappers.lambdaQuery(User.class)
    .eq(User::getStatus, "active");
List<User> activeUsers = Db.list(queryWrapper);

// 插入新实体
User newUser = new User();
newUser.setUsername("newUser");
newUser.setAge(25);
boolean isInserted = Db.insert(newUser);

// 根据 id 更新实体
User updateUser = new User();
updateUser.setId(1L);
updateUser.setUsername("updatedUser");
boolean isUpdated = Db.updateById(updateUser);

// 根据条件构造器更新
LambdaUpdateWrapper<User> updateWrapper = Wrappers.lambdaUpdate(User.class)
    .set(User::getAge, 30)
    .eq(User::getUsername, "updatedUser");
boolean isUpdatedByWrapper = Db.update(null, updateWrapper);

// 根据 id 删除实体
boolean isDeleted = Db.removeById(1L);

// 根据条件构造器删除
LambdaDeleteWrapper<User> deleteWrapper = Wrappers.lambdaDelete(User.class)
    .eq(User::getStatus, "inactive");
boolean isDeletedByWrapper = Db.remove(deleteWrapper);

// 批量插入
List<User> batchUsers = Arrays.asList(
    new User("user1", 20),
    new User("user2", 22),
    new User("user3", 24)
);
boolean isBatchInserted = Db.saveBatch(batchUsers);

// 批量更新
List<User> batchUpdateUsers = Arrays.asList(
    new User(1L, "user1", 21),
    new User(2L, "user2", 23),
    new User(3L, "user3", 25)
);
boolean isBatchUpdated = Db.updateBatchById(batchUpdateUsers);
```

#### [自动维护DDL ](https://baomidou.com/guides/auto-ddl/)  TODO 企业特性？

类似于 `flyway` 的数据库版本管理







