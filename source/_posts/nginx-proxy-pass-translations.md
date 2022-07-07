---
title: nginx proxy_pass 与 url 路径的转换问题
tags:
  - nginx
  - devops
date: 2022-07-07 15:19:56
---

原文 [Understanding Nginx proxy_pass and url path translations](https://tarunlalwani.com/post/nginx-proxypass-server-paths/)

对于
```
location / {
    proxy_pass http://127.0.0.1:8080;
}
```
和
```
location / {
    proxy_pass http://127.0.0.1:8080/;
}
```
两种配置来说，如果你请求 `/abc/def`, 他们都会将请求定位到 `http://127.0.0.1:8080/abc/def`。  
<!-- more -->
但是如果你的配置是 `non-root path` 的话：
```
location /app {
    proxy_pass http://127.0.0.1:8080;
}
```
和
```
location /app/ {
    proxy_pass http://127.0.0.1:8080/;
}
```
这两种配置会被定位到哪儿呢？

下表展示了不同 `location` 和 `proxy_pass` 组合，会将请求定位到的最终地址：

| Case | Nginx location | proxy_pass URL              | Test URL         | Path received         |
| ---- | -------------- | --------------------------- | ---------------- | --------------------- |
| 1    | /test1         | http://127.0.0.1:8080       | /test1/abc/test  | /test1/abc/test       |
| 2    | /test2         | http://127.0.0.1:8080/      | /test2/abc/test  | //abc/test            |
| 3    | /test3/        | http://127.0.0.1:8080       | /test3/abc/test  | /test3/abc/test       |
| 4    | /test4/        | http://127.0.0.1:8080/      | /test4/abc/test  | /abc/test             |
| 5    | /test5         | http://127.0.0.1:8080/app1  | /test5/abc/test  | /app1/abc/test        |
| 6    | /test6         | http://127.0.0.1:8080/app1/ | /test6/abc/test  | /app1//abc/test       |
| 7    | /test7/        | http://127.0.0.1:8080/app1  | /test7/abc/test  | /app1abc/test         |
| 8    | /test8/        | http://127.0.0.1:8080/app1/ | /test8/abc/test  | /app1/abc/test        |
| 9    | /              | http://127.0.0.1:8080       | /test9/abc/test  | /test9/abc/test       |
| 10   | /              | http://127.0.0.1:8080/      | /test10/abc/test | /test10/abc/test      |
| 11   | /              | http://127.0.0.1:8080/app1  | /test11/abc/test | /app1test11/abc/test  |
| 12   | /              | http://127.0.0.1:8080/app2/ | /test12/abc/test | /app2/test12/abc/test |

NOTE: 如果你的 `proxy_pass` 以 `/` 结尾, 那么 `proxy_pass` 在  `url` 中的内容会被移除，剩下的部分会被传递到代理服务器（见 Case#4）。