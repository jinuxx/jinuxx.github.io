---
title: Linux字体添加与更新
date: 2021-07-21 17:54:14
tags:
---
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
<!-- more -->

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