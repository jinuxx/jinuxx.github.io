---
title: Centos转发系统邮件
date: 2021-08-27 14:36:41
tags: 
  - devops
  - centos
---
> You have new mail.

在连接到服务器的时候总会有上面这个提示，可以将邮件转发到自己的邮箱。

在 `~` 目录:
```sh
touch ~/.forward
echo [你的邮箱] >> ~/.forward
```