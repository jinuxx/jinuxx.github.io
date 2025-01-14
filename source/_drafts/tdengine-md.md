---
title: tdengine.md
tags:
---
docker 启动时  
1. docker-compose.yml 文件需要添加环境变量 TAOS_FQDN: {xxxx},  防止在启动时，host变化无法启动
2. 映射外部文件 -v ./taos.cfg:/etc/taos/taos.cfg 其中添加 `keepColumnName 1`，在执行 last, last_row() 等函数时，可以直接展示列名