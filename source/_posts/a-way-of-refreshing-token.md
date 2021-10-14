---
title: 一种刷新token的方式
date: 2021-07-30 09:08:37
tags: java
---

在调用第三方接口时，需要带Token进行验证是很常见的做法，像企业微信/公众号之类，需要先获取access_token，在之后的接口调用中，需要bear这个token，进行调用，下图为[公众号access_token](https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/Get_access_token.html)说明
![](wechat_access_token.jpg)
一般在分布式环境下，或者多人开发时，使用数据库或者redis来保存，防止出现多次获取，导致服务器不稳定。
<!-- more -->
在使用forge时，源码中给到了一个用Java Timer类做定时任务的方式，记录如下:
```java
public class OAuth2TwoLegged {

    public Credentials credentials;

    public Credentials authenticate() {
        if (credentials != null && credentials.notExpired()) {
            return credentials;
        }

        this.credentials = doAuthenticate(); // real authenticate method

        // 定义一个定时器，在这里自己去刷新
        Timer timer = new Timer();
        timer.schedule(new TimerTask(){
            @Override
            public void run() {
                // get token again
                authenticate();
            }
        }, 115 * 60 * 1000); // 每 115 分钟刷新一次
    }

    public static class Credentials {
        private String accessToken;
        private long expiresAt;
        // ...
    }
}
```
当然，这只适用于单机模式下。