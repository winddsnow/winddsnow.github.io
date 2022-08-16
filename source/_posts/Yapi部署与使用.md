---
title: Yapi部署与使用
categories:
  - 工具用法
tags:
  - 前端
  - Yapi
  - 接口管理
date: 2022-08-15 22:37:46
---
# Yapi部署与使用
## 1. 环境依赖，部署环境准备
- Yapi依赖：
 - nodejs（7.6+)--**nodejs版本有大问题！**一定不能太新，Yapi已经一年没有更新了，新版本不兼容！
 - mongodb（2.6+）
 - git

- 这里使用Debian11来部署，单机部署
- **注意！注意！注意！**这里nodois版本选择v8.17.0！！！使用nvm安装
```shell
//依赖nodejs，安装nodejs，这里安装还有点麻烦，需要官网下载nodejs二进制的包，解压安装
#mkdir /usr/local/nodejs
#//nodejs包是两层压缩，外面是xz压缩，里层是tar压缩
#xz -d node-v16.16.0-linux-x64.tar.xz
#tar -xvf node-v16.16.0-linux-x64.tar -C /usr/local/nodejs
#tar -xvJf node-v16.16.0-linux-x64.tar.xz -C /usr/local/nodejs
#设置环境变量
#vim /etc/profile
#nodejs
#NODE_HOME=/usr/local/nodejs
#PATH=$PATH:$NODE_HOME/bin
#使配置生效
#source /etc/profile
#上述16.6安装Yapi失败！
#使用nvm安装nodejs-这里网络不好会下载不了
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
nvm ls-remote
nvm install v8.17.0
nvm use v8.17.0
#检验
node -v
#配置淘宝源
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
- 安装 mongodb
```shell
#依赖安装
apt install curl apt-transport-https software-properties-common gnupg2
# 默认情况下，MongoDB 在 Debian 11 基础存储库中不可用，将 MongoDB 存储库添加到Debian 11 系统
echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/5.0 main" | tee /etc/apt/sources.list.d/mongodb-org.list
echo "deb https://mirrors.tuna.tsinghua.edu.cn/mongodb/apt/debian bullseye/mongodb-org/5.0 main" | tee /etc/apt/sources.list.d/mongodb-org.list
# 添加mongodb源 GPG 密钥
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | apt-key add -
#更新源并安装
apt update
apt install mongodb-org
mongod --version
#启动mongdb并开机自启
systemctl start mongod
systemctl enable mongod
```
## 2.部署Yapi
官方推荐可视化部署
```shell
npm install -g yapi-cli --registry https://registry.npm.taobao.org
#这里部署很容易出问题！这个Yapi报错并不准确，哪怕按提示安装报错所说的缺失插件，一样不识别！经过两天研究，其实归根结底就是nodejs版本兼容问题，对我目前而言无解！只能严格按照版本搭配！nodejs依赖真的有大问题！
#这里最后版本搭配如下：
#node -v-------v8.17.0
##npm -v-------6.13.4
#mongo -version-------------MongoDB shell version v5.0.10
#Yapi-------v1.9.1

yapi server
```
## 3.完成部署
- 按照提示完成部署启动--可查看[官网](https://hellosean1025.github.io/yapi/devops/index.html)配置具体信息
- 我这里使用pm2启动jsapp失败，最后使用forever启动
```shell
npm install forever -g
#完成后台启动
forever start /usr/local/yapi/my-yapi/vendors/server/app.js 

```