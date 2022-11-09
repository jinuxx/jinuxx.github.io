---
title: 一个 Windows 下 JAVA_HOME 设置不生效的问题
tags:
  - devops
  - windows
date: 2022-11-08 15:59:33
---


最惨莫过于在 Windows 上部署生产环境了吧 :( 。  
如果有条件，宁愿直接上虚拟机。

<!-- more -->

Windows 的环境变量和平常认知的并不一样，同样是分成用户变量和系统变量，用户变量只在当前用户环境下生效，系统变量全局生效。

但是，在拼接 PATH 时，Windows 会把用户变量 append 到系统变量后面，并且 Windows 的处理方式是：会按照顺序使用发现的程序。

比如系统中已经安装了 Java 1.7
> C:\Program Files\Java\jdk1.7.0_09

这时候新建了一个用户 user ，希望在此用户执行命令时使用 Java11，因此安装
> C:\Program Files\Java\jdk-11.0.2

接着配置环境变量  
用户：
> JAVA_HOME=C:\Program Files\Java\jdk-11.0.2

系统：此时在 System 中存在一个 JAVA_HOME=C:\Program Files\Java\jdk1.7.0_09
> PATH=...;%JAVA_HOME%\bin;

本来以为 User 变量会覆盖 System 变量，在 cmd 中却发现，`java -version: 1.7`，继续查看其他变量值：
```cmd
C:\Users\Administrator>java -version
java version "1.7.0_09"
Java(TM) SE Runtime Environment (build 1.7.0_09-b05)
Java HotSpot(TM) 64-Bit Server VM (build 23.5-b02, mixed mode)

C:\Users\Administrator>where java
C:\Windows\System32\java.exe
C:\Program Files\Java\jdk1.7.0_09\bin\java.exe

C:\Users\Administrator>echo %PATH%
C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\software\mysql-5.721-winx64\bin;C:\Program Files\Java\jdk1.7.0_09\bin;

C:\Users\Administrator>echo %JAVA_HOME%
C:\Program Files\Java\jdk-11.0.2
```
可以看到 `where java` 中没有 Java11 的信息，于是尝试在用户变量中添加 `PATH=%JAVA_HOME%\bin;`，此时：
```cmd
C:\Users\Administrator>echo %PATH%
C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\software\mysql-5.7.21-winx64\bin;C:\Program Files\Java\jdk1.7.0_09\bin;C:\Program Files\Java\jdk-11.0.2\bin;

C:\Users\Administrator>where java
C:\Windows\System32\java.exe
C:\Program Files\Java\jdk1.7.0_09\bin\java.exe
C:\Program Files\Java\jdk-11.0.2\bin\java.exe
```
现在已经有 Java11 的信息了，但是并不会被优先调用，如果一定要调用，必须将 `JAVA_HOME` 放在系统变量最前面。

如果我没有系统管理员权限呢？？？


