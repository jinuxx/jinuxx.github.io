---
title: docker-init
date: 2021-07-22 14:36:01
tags: 
  - docker
  - devops
---

## docker
### [官网安装](https://docs.docker.com/install/linux/docker-ce/centos/) [中科大镜像安装](https://mirrors.ustc.edu.cn/help/docker-ce.html)



#### 自动安装

```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sudo DOWNLOAD_URL=https://mirrors.ustc.edu.cn/docker-ce sh get-docker.sh
```



#### 手动安装

1. 卸载旧版本
```sh
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
```
<!-- more -->

2. 安装必要依赖
```sh
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

3. 配置安装文件下载地址（使用 ustc 源）（清华：https://download.docker.com/linux/centos/docker-ce.repo）
```sh
sudo yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

4. 替换源
```sh
sudo sed -i 's+download.docker.com+mirrors.ustc.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo
```

4. 安装
```sh
sudo yum install docker-ce docker-ce-cli containerd.io
```

5. 启动(可在下面第二步配置完成后启动)
```sh
sudo systemctl start docker
```


### 镜像下载加速
<details>
  <summary> 镜像源基本都由于种种原因过期 </summary>


1. 加速地址（不能保证还能使用）
| 源     | 地址                                          |
| ------ | --------------------------------------------- |
| 道客云 | http://f1361db2.m.daocloud.io                 |
| 华为   | https://f7vi4u4n.mirror.swr.myhuaweicloud.com |
| ustc   | https://docker.mirrors.ustc.edu.cn/           |
| 腾讯   | https://mirror.ccs.tencentyun.com             |

2. 使用
```sh
sudo mkdir -p /etc/docker
```
```sh
sudo tee /etc/docker/daemon.json <<- 'EOF'
```
```json
{
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}
EOF
```
```sh
sudo systemctl daemon-reload
```
```sh
sudo systemctl restart docker
```

</details>  



使用毫秒镜像：[官网](https://1ms.run/)

```sh
docker pull docker.1ms.run/nginx:latest
```

> 这里的 `nginx:latest` 的请替换成你需要的镜像和版本

### 开启 http 端口管理

1. 编辑docker宿主机文件 `/lib/systemd/system/docker.service`
```
sudo vi /lib/systemd/system/docker.service
```
2. 修改以 `ExecStart` 为开头的行 ，修改为：
```sh
ExecStart=/usr/bin/dockerd -H unix://var/run/docker.sock -H tcp://0.0.0.0:2375
```
3. 保存修改后的文件，通知docker服务做出的修改
```sh
sudo systemctl daemon-reload
```
4. 重启docker服务
```sh
sudo service docker restart
```
5. 测试可以连接到docker api
> curl http://localhost:2375/version

## docker-compose
```sh
curl -L https://get.daocloud.io/docker/compose/releases/download/1.22.0/docker-compose-\`uname -s\`-\`uname -m\` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## [docker swarm](https://docs.docker.com/engine/swarm/) -- 集群管理  
`docker-swarm` 传统模式：[https://docs.docker.com/swarm/](https://docs.docker.com/swarm/)

下面使用 `swarm mode`。

1. 创建 `manager`  
> docker swarm init --advertise-addr \<MANAGER IP\>