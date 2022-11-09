---
title: Spring Boot 下的单元测试
tags:
  - Java
  - Spring Boot
  - translate
date: 2022-11-09 11:16:21
---

[原文链接 Unit Testing with Spring Boot](https://reflectoring.io/unit-testing-spring-boot/)  

## 依赖
* JUnit Jupiter (JUnit 5)
* Mockito
* AssertJ

<!-- more -->
## 不要在单元测试中使用 Spring
在以下"单元"测试中，我们只想测试 `RegisterUseCase` 中的一个方法：
```Java
@ExtendWith(SpringExtension.class)
@SpringBootTest
class RegisterUseCaseTest {

  @Autowired
  private RegisterUseCase registerUseCase;

  @Test
  void savedUserHasRegistrationDate() {
    User user = new User("zaphod", "zaphod@mail.com");
    User savedUser = registerUseCase.registerUser(user);
    assertThat(savedUser.getRegistrationDate()).isNotNull();
  }
}
```
这个测试用时4.5秒，启动了一个空的 Spring 项目。由于 `@SpringBootTest` 注解，我们启动了整个 `Spring Boot` 上下文环境，只是为了注入一个 `RegisterUseCase` 的实例而已！

## 创建一个可测试的 Spring Bean
以下手段可以让 Spring Bean 更易测试
### 字段注入（Field Injection）并不好
```Java
@Service
public class RegisterUseCase {

  @Autowired
  private UserRepository userRepository;

  public User registerUser(User user) {
    return userRepository.save(user);
  }
}
```
上面的这个类没办法脱离 Spring 做单元测试，必须依赖 `UserRepository` 的实例Bean，需要用到上一节中展示的，通过 Spring 创建一个实例并且通过 `@Autowired` 来注入。

这个教程教你如果不用字段注入。
### 使用构造器
不如，我们直接抛弃 `@Autowired`:
```Java
@Service
public class RegisterUseCase {

  private final UserRepository userRepository;

  public RegisterUseCase(UserRepository userRepository) {
    this.userRepository = userRepository;
  }

  public User registerUser(User user) {
    return userRepository.save(user);
  }
}
```
以上代码使用了 [构造器注入](https://reflectoring.io/constructor-injection/) 进行依赖注入，在创建上下文环境时 Spring 会自动实例化一个`UserRepository`，在 Spring5之前，还是需要添加 `@Autowired` 让 Spring 找到构造器。

同时，`UserRepository` 现在也是 `final` 的了，因为在整个 Spring 的生命周期里，这个实例不会再变化，同事也避免了一些其他错误。

### 减少模板重复代码
充分利用 `Lombok` 的 [`@RequiredArgsConstructor`](https://projectlombok.org/features/constructor) 自动生成构造器：
```java
@Service
@RequiredArgsConstructor
public class RegisterUseCase {

  private final UserRepository userRepository;

  public User registerUser(User user) {
    user.setRegistrationDate(LocalDateTime.now());
    return userRepository.save(user);
  }
}
```
现在这个类已经没有什么模板性的代码了，非常简洁而且可以通过简单的 Java 代码来实例化并测试：
```Java
class RegisterUseCaseTest {

  private UserRepository userRepository = ...;

  private RegisterUseCase registerUseCase;

  @BeforeEach
  void initUseCase() {
    registerUseCase = new RegisterUseCase(userRepository);
  }

  @Test
  void savedUserHasRegistrationDate() {
    User user = new User("zaphod", "zaphod@mail.com");
    User savedUser = registerUseCase.registerUser(user);
    assertThat(savedUser.getRegistrationDate()).isNotNull();
  }
}
```
还有一个片段没有实现，我们可以通过 mock `UserRepository` 来解决这个依赖问题，毕竟我们测试时是不会用真的实例的。

## 用 `Mockito` 来 mock 依赖
[`Mockito`](https://site.mockito.org/) 是现在比较常用的 mock库，它提供了两种 mock 的方法：
### 用原生 `Mockito` 来 mock 依赖
```Java
private UserRepository userRepository = Mockito.mock(UserRepository.class);
```
这样一个`UserRepository`实例就会被创建出来。默认情况下，所有的方法调用什么都不会做，如果调用的方法有返回值的话，会返回 `null`。

我们的测试在 `assertThat(savedUser.getRegistrationDate()).isNotNull();` 这一行会报错，因为 `savedUser` 是一个 `null`，现在我们需要告诉 `Mockito` 在调用 `registerUseCase.registerUser()` 时要返回点东西。
```Java
@Test
void savedUserHasRegistrationDate() {
  User user = new User("zaphod", "zaphod@mail.com");
  when(userRepository.save(any(User.class))).then(returnsFirstArg());
  User savedUser = registerUseCase.registerUser(user);
  assertThat(savedUser.getRegistrationDate()).isNotNull();
}
```
以上代码中，`userRepository.save()` 会返回第一个入参，也就是手动创建的 User。

其他 `Mockito` 的使用参见文档 [Mockito version 3.8.0](https://www.javadoc.io/doc/org.mockito/mockito-core/3.8.0/org/mockito/Mockito.html)  
-- 在 `Spring Boot 2.5.14` 版本中，`mockito-core` 的版本是 `3.9.0`，官网没有这个版本的文档。

### 用 `@Mock` 注解来 mock 依赖
另一种方式是使用 `@Mock` 注解结合 JUnit 的 `@MockitoExtension` 来创建 mock 依赖：
```Java
@ExtendWith(MockitoExtension.class)
class RegisterUseCaseTest {

  @Mock
  private UserRepository userRepository;

  private RegisterUseCase registerUseCase;

  @BeforeEach
  void initUseCase() {
    registerUseCase = new RegisterUseCase(userRepository);
  }

  @Test
  void savedUserHasRegistrationDate() {
    // ...
  }
}
```
`@Mock` 注解指定哪那些变量需要 mock，`@MockitoExtension` 告诉 Junit 需要用 Mockito 框架来创建这些变量。这样做和手动调用 `Mockito.mock()` 基本一致，只是 `@MockitoExtension` 将测试绑定到了测试框架上。

除了手动创建 `RegisterUseCase` 之外，我们也可以用 `@InjectMocks` 注解来创建 `registerUseCase` 变量。Mockito 会通过一个[特殊机制](https://www.javadoc.io/doc/org.mockito/mockito-core/3.8.0/org/mockito/InjectMocks.html)帮我们创建变量。

## 使用可读易读的 Assertion
`Spring Boot` 提供了 [AssertJ](http://joel-costigliola.github.io/assertj/)，我们在上文中已经使用了：
```Java
assertThat(savedUser.getRegistrationDate()).isNotNull();
```
... 这个部分看原文吧，感觉做修改用处没那么大 ...

## 结论
在测试中引入整个 Spring 环境是有意义的，但是对于单纯的单元测试来说不太必要，甚至长时间、周期性的环境构建时间没有什么好处。
通过一些手段创建易于进行单元测试的类是很有必要的。

[源码](https://github.com/thombergs/code-examples/tree/master/spring-boot/spring-boot-testing)
