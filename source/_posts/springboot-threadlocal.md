---
title: springboot threadlocal
tags: java
date: 2021-10-14 15:58:01
---

Spring Boot 中，线程都是从线程池中获取的，而这个线程用完后，不会销毁，而是放回线程池中。  
因此如果使用 `ThreadLocal`，需要及时的 `remove`。

[stackoverflow](https://stackoverflow.com/questions/37332219/questions-about-using-threadlocal-in-a-spring-singleton-scoped-service?answertab=active#tab-top)