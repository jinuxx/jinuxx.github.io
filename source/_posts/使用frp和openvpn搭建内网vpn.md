---
title: 使用frp和openvpn搭建内网vpn
date: 2021-07-21 10:44:53
tags:
---
为了满足有时需要在家办公的需求，而且公司没有固定IP，只能出此下策。

原理就是用frp转发到公司内部的机器，主要使用：
* [frp](https://github.com/fatedier/frp)
* [openvpn](https://github.com/OpenVPN/)，但是用了这个简易安装脚本 [angristan/openvpn-install](https://github.com/angristan/openvpn-install)

<!-- more -->
### frp配置
服务端配置：
```ini
# server frps.ini
[common]
bind_port = 57000
token = FAKE_TOKE_CHANGE_ME
log_file = ./log/frps.log
log_level = info
log_max_days = 3
```
客户端配置：
```ini
# client frpc.ini
[common]
server_addr = a.b.c.d # 服务端ip
server_port = 57000
token = FAKE_TOKE_CHANGE_ME
log_file = ./log/frpc.log
log_level = info
log_max_days = 3

[vpn_udp]
type = udp # 协议为udp
local_port = 1194 # 本地vpn端口
remote_port = 51194 # 服务端开放端口
```
> 注意在开启云服务器的防火墙时，将协议类型设置为 `UDP`。
### vpn
下载脚本，给予权限
```sh
curl -O https://raw.githubusercontent.com/angristan/openvpn-install/master/openvpn-install.sh
chmod +x openvpn-install.sh
```
运行脚本
```sh
./openvpn-install.sh
```
这个脚本应该是生成配置文件的，要注意的地方
* 其中第二步(?)，填写为服务端ip
* 生成后的 ovpn 文件，修改端口为 51194

这些都可以在生成配置文件后修改