---
title: Linux 之间免密
date: 2021-07-16 22:39:59
tags: devops
---
A免B密码

1. 在A中执行命令：
    ```sh
    # 这会在 ~/.ssh 目录下生成两个文件：`id_rsa` 和 `id_rsa.pub`
    $ ssh-keygen -t rsa -P "" 
    ```

2. 登录B，并把 `id_rsa.pub` 输入到B的 `authorized_keys` 文件中：
    ```sh
    $ cat id_rsa.pub >> ~/.ssh/authorized_keys
    ```

3. 有时还是不能免密  
    https://stackoverflow.com/a/21640671