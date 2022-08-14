---
title: nginx简介及使用进阶
date: 2022-08-14 11:30:02
categories:
-  编程中间件
tags:
- nginx
- http服务器
- 高并发
---

## 1. nginx介绍
- Nginx是一款轻量级的Web服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好，中国大陆使用nginx的网站有：百度、京东、新浪、网易、腾讯、淘宝等。
- Nginx是由**伊戈尔·赛索耶夫**为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。
[nginx官网](https://nginx.org/)

## 2. 安装
一般推荐安装在linux服务器，其他操作系统可查看官网网站
#### 2.1 联网使用包管理器安装方式
最简单即使用包管理器安装，即红帽系的yum，Debian系的apt等等联网包管理器，这里以yum举例,官方也有[文档](https://nginx.org/en/linux_packages.html#RHEL-CentOS)
- yum官方源中没有nginx，所以需要首先把 nginx 的源加入 yum 中，这一步可以直接下载repo文件，也可以直接编辑生成nginx.repo
```shell
直接下载repo文件方式
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```
```shell
直接编辑生成nginx.repo
vim /etc/yum.repos.d/nginx.repo

[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```
- 执行安装命令
```shell
sudo yum install nginx
```
- 启动及检验版本
```shell
systemctl status nginx
systemctl start nginx
nginx -v
```
- 联网安装优缺点
优点是安装方便，简单，快速，自动提供了systemctl启动重启开机自启
缺点是需要联网安装，版本选择有限，有些配置无法指定，很多路径是默认配置的
```shell
yum安装默认路径等
/etc/nginx/nginx.conf  //yum方式安装后默认配置文件的路径

/usr/share/nginx/html  //nginx网站默认存放目录

/usr/share/nginx/html/index.html //网站默认主页路径
```

#### 2.2 源码编译安装
1. 安装依赖包
由于nginx是基于c语言开发的，所以需要安装c语言的编译环境，及正则表达式库等第三方依赖库。
```shell
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
```
2. 上传安装包
3. 解压nginx压缩包
```shell
tar -zxvf nginx-1.16.1.tar.gz   --根据自己安装的版本灵活解压对应压缩包
说明：
-z：z代表的是gzip，通过gzip命令处理文件，gzip可以对文件压缩或者解压
-c：c代表的是create，即创建新的包文件
-x：x代表的是extract，实现从包文件中还原文件
-v：v代表的是verbose，显示命令的执行过程
-f：f代表的是file，用于指定包文件的名称

```
4. 配置Nginx编译环境
```shell
#切换到软件目录
cd nginx-1.16.1
#--prefix 指定的目录，就是我们安装Nginx的目录，后面可以根据需求激活不同配置
./configure --prefix=/usr/local/nginx
```
5. 编译&安装
```shell
make & make install
```
| 目录/文件 | 说明 | 备注 |
| --------- | ---- | ---- |
| conf | 配置文件的存放目录 |      |
| conf | Nginx的核心配置文件 | conf下有很多nginx的配置文件，我们主要操作这个核心配置文件 |
| html | 存放静态资源(html, css, ) | 部署到Nginx的静态资源都可以放在html目录中 |
| logs | 存放nginx日志(访问日志、错误日志等) |      |
| sbin/nginx | 二进制文件，用于启动、停止Nginx服务 |      |

## 3. Nginx-命令
```shell
#安装了systemctl的前提下，直接使用：
systemctl stop nginx   #停止
systemctl start nginx #启动
systemctl enable nginx #开机自启

#源码安装情况：
查看nginx版本：			./nginx -v
查看配置文件是否正确：		./nginx -t
启动nginx：			./nginx     【默认使用conf/nginx.conf配置文件 windows下启动：start nginx.exe】
停止nginx：			./nginx -s stop
重新加载nginx配置文件：	./nginx -s reload
```
> 源码安装可以制作启动文件系统服务（system）
```
vim /lib/systemd/system/nginx.service
```
```shell
#服务的说明
[Unit]
#描述服务
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
#描述服务类别--比如After= network.target
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

#服务运行参数的设置
[Service]
#后台运行的形式
Type=forking
PIDFile=/var/run/nginx.pid
#服务的具体运行命令
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
#重启命令
ExecReload=/bin/sh -c "/bin/kill -s HUP $(/bin/cat /var/run/nginx.pid)"
停止命令
ExecStop=/bin/sh -c "/bin/kill -s TERM $(/bin/cat /var/run/nginx.pid)"
#PrivateTmp=True表示给服务分配独立的临时空间
#PrivateTmp= true
#注意：[Service]的启动、重启、停止命令全部要求使用绝对路径
#[Install]运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3
[Install]
WantedBy=multi-user.target
```

## 4. nginx配置文件详解
Nginx配置文件(conf/nginx.conf)整体分为三部分：
- 全局块	和Nginx运行相关的全局配置，配置和nginx运行相关的全局配置
- events块	和网络连接相关的配置，配置和网络连接相关的配置
- http块		代理、缓存、日志记录、虚拟主机配置，配置代理、缓存、日志记录、虚拟主机等配置
	- http全局块
	- Server块
		- Server全局块
		- location块
> 注意：http块中可以配置多个Server块，每个Server块中可以
配置多个location块。
```shell
#配置文件中文详解
#运行用户
user www-data;    
#启动进程,通常设置成和cpu的数量相等
worker_processes  1;
#全局错误日志及PID文件
 error_log  /var/log/nginx/error.log;
 pid        /var/run/nginx.pid;
 #工作模式及连接数上限
    events {
        use   epoll;                               #epoll是多路复用IO(I/O Multiplexing)中的一种方式,但是仅用于linux2.6以上内核,可以大大提高nginx的性能
        worker_connections  1024;     #单个后台worker process进程的最大并发链接数
        # multi_accept on;
    }
#设定http服务器，利用它的反向代理功能提供负载均衡支持
    http {
#设定mime类型,类型由mime.type文件定义
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
#设定日志格式
        access_log    /var/log/nginx/access.log;
 #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
 #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
        sendfile        on;
      #tcp_nopush     on;
#连接超时时间
        #keepalive_timeout  0;
        keepalive_timeout  65;
        tcp_nodelay        on;
#开启gzip压缩
        gzip  on;
        gzip_disable "MSIE [1-6]\.(?!.*SV1)";
#设定请求缓冲
        client_header_buffer_size    1k;
        large_client_header_buffers  4 4k;
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
 #设定负载均衡的服务器列表
         upstream mysvr {
#weigth参数表示权值，权值越高被分配到的几率越大
#本机上的Squid开启3128端口
        server 192.168.8.1:3128 weight=5;
        server 192.168.8.2:80  weight=1;
        server 192.168.8.3:80  weight=6;
        }
       server {
 #侦听80端口
            listen       80;
 #定义使用www.xx.com访问
            server_name  www.xx.com;
#设定本虚拟主机的访问日志
            access_log  logs/www.xx.com.access.log  main;
#默认请求-匹配客户端请求url
        location / {
        	##指定静态资源根目录
              root   /root;            #定义服务器的默认网站根目录位置
              index index.php index.html index.htm;        #定义首页索引文件的名称
             fastcgi_pass  www.xx.com;
             fastcgi_param  SCRIPT_FILENAME  $document_root/$fastcgi_script_name;
              include /etc/nginx/fastcgi_params;
            }
# 定义错误提示页面
        error_page   500 502 503 504 /50x.html;  
            location = /50x.html {
            root   /root;
        }
#静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {
            root /var/www/virtual/htdocs;
#过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
            expires 30d;
        }
 #PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
        location ~ \.php {
            root /root;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME /home/www/www$fastcgi_script_name;
            include fastcgi_params;
        }
#设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status            on;
            access_log              on;
            auth_basic              "NginxStatus";
            auth_basic_user_file  conf/htpasswd;
        }
#禁止访问 .htxxx 文件
        location ~ /\.ht {
            deny all;
        }
         
         }
    }
以上是一些基本的配置,使用Nginx最大的好处就是负载均衡
如果要使用负载均衡的话,可以修改配置http节点如下：
#设定http服务器，利用它的反向代理功能提供负载均衡支持
    http {
#设定mime类型,类型由mime.type文件定义
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
#设定日志格式
        access_log    /var/log/nginx/access.log;
#省略上文有的一些配置节点
#。。。。。。。。。。
#设定负载均衡的服务器列表
         upstream mysvr {
 #weigth参数表示权值，权值越高被分配到的几率越大
        server 192.168.8.1x:3128 weight=5;                #本机上的Squid开启3128端口
        server 192.168.8.2x:80  weight=1;
        server 192.168.8.3x:80  weight=6;
        }
       upstream mysvr2 {
#weigth参数表示权值，权值越高被分配到的几率越大
        server 192.168.8.x:80  weight=1;
        server 192.168.8.x:80  weight=6;
        }
#第一个虚拟服务器
       server {
#侦听192.168.8.x的80端口
            listen       80;
            server_name  192.168.8.x;
#对aspx后缀的进行负载均衡请求
        location ~ .*\.aspx$ {
             root   /root;                                                  #定义服务器的默认网站根目录位置
              index index.php index.html index.htm;      #定义首页索引文件的名称
              proxy_pass  http://mysvr ;                          #请求转向mysvr 定义的服务器列表
#以下是一些反向代理的配置可删除.
              proxy_redirect off;
#后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              client_max_body_size 10m;                      #允许客户端请求的最大单文件字节数
              client_body_buffer_size 128k;                  #缓冲区代理缓冲用户端请求的最大字节数，
              proxy_connect_timeout 90;                     #nginx跟后端服务器连接超时时间(代理连接超时)
              proxy_send_timeout 90;                          #后端服务器数据回传时间(代理发送超时)
              proxy_read_timeout 90;                          #连接成功后，后端服务器响应时间(代理接收超时)
              proxy_buffer_size 4k;                              #设置代理服务器（nginx）保存用户头信息的缓冲区大小
              proxy_buffers 4 32k;                               #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
              proxy_busy_buffers_size 64k;                 #高负荷下缓冲大小（proxy_buffers*2）
              proxy_temp_file_write_size 64k;             #设定缓存文件夹大小，大于这个值，将从upstream服务器传
           }
         }
    }
```

## 5. 配置反向代理
在http块中,再添加一个server块虚拟主机的配置,监听82端口,并配置反向代理proxy_pass: 
位置为server下location
```shell
server {
    listen 82;
    server_name localhost;
    location / {
        proxy_pass http://192.168.200.201:8080; 	#反向代理配置，将请求转发到指定服务
    }
}
```
## 6.nginx配置负载均衡
打开nginx的配置文件nginx.conf并增加如下配置: 
```properties
#upstream指令可以定义一组服务器
upstream targetserver{	
    server 192.168.200.201:8080;
    server 192.168.200.201:8081;
}

server {
    listen       8080;
    server_name  localhost;
    location / {
        proxy_pass http://targetserver;
    }
}
```
开放防火墙
```shell
firewall-cmd --zone=public --add-port=82/tcp --permanent

firewall-cmd --reload
```
负载均衡策略
处理上述默认的轮询策略以外，在Nginx中还提供了其他的负载均衡策略，如下： 

| **名称**   | **说明**         | 特点                                                         |
| ---------- | ---------------- | ------------------------------------------------------------ |
| 轮询       | 默认方式         |                                                              |
| weight     | 权重方式         | 根据权重分发请求,权重大的分配到请求的概率大                  |
| ip_hash    | 依据ip分配方式   | 根据客户端请求的IP地址计算hash值， 根据hash值来分发请求, 同一个IP发起的请求, 会发转发到同一个服务器上 |
| least_conn | 依据最少连接方式 | 哪个服务器当前处理的连接少, 请求优先转发到这台服务器         |
| url_hash   | 依据url分配方式  | 根据客户端请求url的hash值，来分发请求, 同一个url请求, 会发转发到同一个服务器上 |
| fair       | 依据响应时间方式 | 优先把请求分发给处理请求时间短的服务器                       |