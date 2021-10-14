---
title: shc 加密脚本
date: 2021-10-12 16:28:07
tags: devops
---

有时候，脚本中含有 ip、密码等敏感数据，需要将脚本加密，shc 工具就可以达到这个目的。

<!-- more -->
首先安装 shc，略过

```
shc Usage: shc [-e date] [-m addr] [-i iopt] [-x cmnd] [-l lopt] [-o outfile] [-rvDSUHCABh] -f script

    -e %s  Expiration date in dd/mm/yyyy format [none]
    -m %s  Message to display upon expiration ["Please contact your provider"]
    -f %s  File name of the script to compile
    -i %s  Inline option for the shell interpreter i.e: -e
    -x %s  eXec command, as a printf format i.e: exec('%s',@ARGV);
    -l %s  Last shell option i.e: --
    -o %s  output filename
    -r     Relax security. Make a redistributable binary
    -v     Verbose compilation
    -S     Switch ON setuid for root callable programs [OFF]
    -D     Switch ON debug exec calls [OFF]
    -U     Make binary untraceable [no]
    -H     Hardening : extra security protection [no]
           Require bourne shell (sh) and parameters are not supported
    -C     Display license and exit
    -A     Display abstract and exit
    -B     Compile for busybox
    -h     Display help and exit

    Environment variables used:
    Name    Default  Usage
    CC      cc       C compiler command
    CFLAGS  <none>   C compiler flags
    LDFLAGS <none>   Linker flags
```
示例：
> shc -Uf myscript -o mybinary

会生成两个文件 `*.sh.x` 和 `*.sh.x.c`

`*.sh.x.c` 是脚本源文件，可以删除。
`*.sh.x` 是加密的可执行文件，课改明后运行。

