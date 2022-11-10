---
title: 家里网络环境/下载机
tags:
  - 瞎搞
date: 2022-11-10 11:00:18
---

一直一来都在瞎搞的一个家用下载机
* r2s [firmware](https://github.com/DHDAXCW/NanoPi-R2S-rk3328)
* qbittorrent [4.3.9_libtorrent1.2.15](https://github.com/userdocs/qbittorrent-nox-static/releases/tag/release-4.3.9_v1.2.15)
* [flexget](https://flexget.com/) 这个在腾讯轻量云上
* smb
* [mdadm](https://wiki.debian.org/SoftwareRAID)/[btrfs](https://wiki.debian.org/Btrfs)/[zfs](https://wiki.debian.org/ZFS)
...  
<!-- more -->
## 硬件
现在用的是一块 DC 供电的 J1900 板子，只有一个 SATA 接口，一个磁盘供电（PH2.0），有两个 minipcie，2个 usb3.0(但只有下面的口插上后显示3.0)，2个 usb2.0，其他什么都没有了
![板子_1](board_1.jpg)
![板子_2](board_2.jpg)

现在有3块硬盘：1.8T的移动硬盘（2\*WD, 1\*TOSHIBA，拆开发现全部都是焊死的特殊接口），其中WD一块有问题，好像坏道特别多。还有一个 120G 的固态，现在系统安装：
1. 128G 固态作为系统盘，装 Debian 系统，插在一个 USB2.0上 -- 应该不需要速度吧，只要系统加载完就结束了
2. 1.8T TOSHIBA 插在 USB3.0上
3. ~~1.8T WD 拆开插在 SATA 上~~
4. ~~1.8T WD (坏) 备用吧...~~


## qbittorrent
自启动：
`vim /etc/systemd/system/qbittorrent-nox.service`
```
[Unit]
Description=qBittorrent-nox
After=network.target

[Service]
User=root
Type=forking
RemainAfterExit=yes
ExecStart=/usr/bin/qbittorrent-nox -d --webui-port=8083

[Install]
WantedBy=multi-user.target
```

`qbit_log.sh`:
```shell
tail -50f /root/.local/share/qBittorrent/logs/qbittorrent.log
```
`torrent_finished.sh`:
这个需要配置 qbittorrent： 设置 - 下载 - Torrent 完成时运行外部程序
```shell
#! /bin/bash
# /root/torrent_finished.sh "%N" "%C" "%Z" "%F" "%L"

torrent_name=$1
total_files=$2
size_bytes=$3
content_path=$4
category=$5

/usr/bin/curl -H "Content-Type: application/json" \
    -d "{\"torrent_name\":\"$torrent_name\",\"category\":\"${category:=?}\",\"content_path\":\"$content_path\",\"total_files\":$total_files,\"size_bytes\":$size_bytes}" \
    -X POST $YOUR_NOTIFY_API
```

## smb
`apt install samba`
`vim /etc/samba/smb.conf`
```
[global]
       workgroup = WORKGROUP
       security = share
       map to guest = Bad User
[tank]
       comment = resources
       path = /tank
       public = yes
       writable = yes
       guest ok = yes
       browseable = yes
[save]
       comment = resources
       path = /mnt
       public = yes
       writable = yes
       guest ok = yes
       browseable = yes
```


## mdadm
1. create raid 0
   `mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb1 /dev/sdc1`
2. create filesystem
   `mkfs.ext4 /dev/md0`
3. auto mount `vim /etc/fstab`
   `UUID=d160dd45-bf3c-4953-83c2-5093192cad7d /tank           ext4    defaults        0       2`
4. Persist the array configuration to mdadm.conf
   `mdadm --detail --scan --verbose | sudo tee -a /etc/mdadm/mdadm.conf`

delete a mdadm
1. `umount /dev/md0`
2. `mdadm --stop /dev/md0`
3. `mdadm --misc --zero-superblock /dev/sd[?]`
4. `rm -rf /etc/mdadm/mdadm.conf`
5. 清除 `/etc/fstab` 中挂载的部分