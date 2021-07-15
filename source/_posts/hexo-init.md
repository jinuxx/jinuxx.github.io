---
title: hexo init
tags: hexo
categories:
  - code
  - other
date: 2021-07-15 16:44:59
---
## hexo next 创建记录
开始时，对 `hexo` 理解错了，以为和 `jekyll` 一样，是每个主题单独部署，其实所有主题都是放在 `themes` 目录下，通过配置文件切换。

### 安装 hexo
[hexo文档](https://hexo.io/zh-cn/docs/)
<!--more-->

### 使用 hexo next 主题
```sh
$ cd your-hexo-site
$ git clone https://github.com/theme-next/hexo-theme-next themes/next # 注意这个文件路径，不要多层嵌套了
```
修改 `_config.yml` 文件
```yml
language: zh-CN # 中文
theme: next # 主题为 next
```
其他配置自行修改

### hexo 启动本地环境测试
> hexo s [--debug]

访问 [http://localhost:4000](http://localhost:4000)

### hexo 编写文章
> hexo new [post | draft | page] "post name"

如果只是需要展示顶部，在分割的地方添加 `<!--more-->`

### hexo 发布
发布至 `github-page`，需要插件 `hexo-deployer-git`
```sh
$ npm install hexo-deployer-git --save
```
先修改 `_config.yml`
```yaml
deploy:
  type: git
  repo: https://github.com.cnpmjs.org/jinuxx/jinuxx.github.io
  branch: gh-pages # 我的仓库中，main分支用来保存这个项目，gh-pages分支是用来发布的
  message: update
```
运行命令发布至 gh-page
> hexo clean & hexo d

