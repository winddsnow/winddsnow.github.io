---
title: Hello World
date: 1970-01-01 08:00:00
categories:
-  编程
tags:
- 42
- helloworld
---
这是用GitHub的站点，百度了一番教程，做出来的静态blog。用hexo驱动，主题是 hexo官网里的archer。在本地编写md文件后，由hexo编译，再上传到GitHub上，完成博文更新。

## 1. 环境准备
- hexo基于nodejs，所以需要nodejs环境
- python--如果没有，安装nodejs时可以通过npm自动安装
- git

[nodejs官网](https://nodejs.org/en/)
npm下载慢的话也可以下载淘宝下载源cnpm
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

1. 安装hexo
```
npm install -g hexo-cli
或者
cnpm install -g hexo-cli

hexo的上传插件deploy-git
```
2. 部署所需插件：
```
cnpm install hexo-deployer-git --save
```
3. 本文章使用的是Archer 主题
Archer 主题依赖于 hexo-generator-json-content 和 hexo-wordcount，因此需要在 Hexo 根目录执行以下命令：
```
npm和cnpm看个人网络自行选择
cnpm install hexo-generator-json-content --save
cnpm install hexo-wordcount --save
```
4. 依赖安装完成后，拉取 Archer 主题到 themes/archer 目录，在 Hexo 根目录执行下面的命令：
```
这里是下载主题，可以自行查看下载文件夹
git clone https://github.com/fi3ework/hexo-theme-archer.git themes/archer --depth=1
```
5. 设置主题，主题也有github，可以自行去查看文档
```
修改 Hexo 根目录下的 _config.yml 文件中的 theme 字段为 archer：
theme: archer
主题GitHub地址：https://github.com/fi3ework/hexo-theme-archer
```

## 2. 快速开始
- 新建一篇文章

``` bash
$ hexo new "My New Post"
```

- 启动hexo用于调试测试

``` bash
$ hexo server
```

- 生成静态页面

``` bash
$ hexo generate
```

- 部署到服务端

``` bash
$ hexo deploy
```
## 3. 日常更新文章

```
#以下操作均在hexo根目录下使用gitbash命令行操作
#新建文章标题
hexo new "新文章标题"
#打开hexo根目录\source\_posts---找到新建md文件，书写文章
#生成静态页面
hexo g
#部署--这里前提是已经在配置文件_config.yml中完成相关配置，未配置的，可自行查看hexo官网配置
hexo d
```
部署完成后，查看网站效果：
https://winddsnow.github.io