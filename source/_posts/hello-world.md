---
title: Hello World
tags: 1
---
这是用GitHub的站点，百度了一番教程，做出来的静态blog。用hexo驱动，主题是 hexo官网里的archer。在本地编写md文件后，由hexo编译，再上传到GitHub上，完成博文更新。

## 环境准备
- ### hexo基于nodejs，所以需要nodejs环境
- ### python--如果没有，安装nodejs时可以通过npm自动安装
- ### git

[nodejs官网](https://nodejs.org/en/)
npm下载慢的话也可以下载淘宝下载源cnpm
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

安装hexo
```
npm install -g hexo-cli
或者
cnpm install -g hexo-cli

hexo的上传插件deploy-git
```
npm install hexo-deployer-git --save
```

```
## 快速开始

### 新建一篇文章

``` bash
$ hexo new "My New Post"
```

### 启动hexo用于调试测试

``` bash
$ hexo server
```

### 生成静态页面

``` bash
$ hexo generate
```

### 部署到服务端

``` bash
$ hexo deploy
```
