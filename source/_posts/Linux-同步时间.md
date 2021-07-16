---
title: Linux 同步时间
date: 2021-07-16 22:10:16
tags: devops
---
* 在使用 vm 时，出现了快照恢复后，系统时间是快照创建的时间，导致向阿里云上传时，时间戳验证不通过的问题。

* 因此开启 centos 时间自动同步的功能。

## ntpd

1. 安装 nptd `yum install -y ntp`
<!-- more -->
2. 修改用于同步的服务器 `vim /etc/ntp.conf`
   ```conf
   ...
    server ntp.aliyun.com prefer
    server ntp1.aliyun.com
    # 1 - 7 都可以
    server ntp7.aliyun.com
   ...
   ```

3. 先手动同步一次时间，防止差别过大，ntpd不工作 `ntpdate ntp.aliyun.com`
3. 启动服务 `systemctl start ntpd`
4. 查看状态 
   * `ntpstat`
   * `ntpq -p`
