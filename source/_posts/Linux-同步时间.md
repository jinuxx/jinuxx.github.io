---
title: Linux 一些常用命令记录
date: 2021-07-16 22:10:16
tags: devops
---
## 判断脚本执行是来自用户命令，还是cron
```sh
# 获取执行这个脚本的手动还是cron
PPPID=`ps h -o ppid= $PPID`
P_COMMAND=`ps h -o %c $PPPID`
if [ "$P_COMMAND" == "crond" ]; then  # 注意这个 crond，有的系统可能是 cron
    RUNNING_FROM_CRON=1
fi

# 不是从cron运行，
if [ "$RUNNING_FROM_CRON" == "1" ]; then
    # cron
else 
    # manual
fi
```
<!-- more -->

## Linux之间免密
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



## 同步标准时间 ntpd
* 在使用 vm 时，出现了快照恢复后，系统时间是快照创建的时间，导致向阿里云上传时，时间戳验证不通过的问题。
* 因此开启 centos 时间自动同步的功能。

1. 安装 nptd `yum install -y ntp`
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

## 添加/更新字体
1. 查看中文字体
```sh
fc-list :lang=zh
```
2. 安装字体
```sh
yum -y install fontconfig	#安装字体库
yum -y install ttmkfdir mkfontscale	#安装字体索引信息
```
可拷贝Windows的字体目录：`C:\Windows\Fonts`
3. Linux字体目录：`/usr/share/fonts`，建议创建一个目录放中文字体 `mkdir chinese`
4. 把字体上传到 `/usr/share/fonts/chinese` 目录
5. 然后在 `/usr/share/fonts/chinese` 执行命令，生成字库索引信息
```sh
mkfontscale
mkfontdir
```
6. 更新字体缓存
```sh
fc-cache
```