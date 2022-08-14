---
title: mysql读写分离-主从复制及sharing-jdbc使用
date: 2022-08-14 10:12:03
categories:
-  编程与数据库
tags:
- MySQL
- 数据库操作
---
#### 1. Mysql主从复制介绍
MySQL主从复制是一个异步的复制过程，底层是基于Mysql数据库自带的二进制日志功能。就是一台或多台MySQL数据库（slave，即从库）从另一台MySQL数据库（master，即主库）进行日志的复制然后再解析日志并应用到自身，最终实现从库的数据和主库的数据保持一致。MySQL主从复制是MySQL数据库自带功能，无需借助第三方工具。
> **二进制日志：** 
>
> ​	二进制日志（BINLOG）记录了所有的 DDL（数据定义语言）语句和 DML（数据操纵语言）语句，但是不包括数据查询语句。此日志对于灾难时的数据恢复起着极其重要的作用，MySQL的主从复制， 就是通过该binlog实现的。默认MySQL是未开启该日志的。
Mysql主从复制分成三步
1.1 master将改变记录到二进制日志（binary log）
1.2 slave将master的binary log拷贝到它的中继日志（relay log）
1.3 slave重做中继日志中的事件，将改变应用到自己的数据库中

#### 2. 主从配置关键配置代码
2.1 修改Mysql数据库的配置文件/etc/my.cnf
```cnf
[mysqld]
log-bin=mysql-bin   #[必须]启用二进制日志
server-id=100      #[必须]服务器唯一ID

```
2.2 重启Mysql服务
systemctl restart mysqld
2.3 登录Mysql数据库，新建从库用户及授权---**重要配置！关键配置！**

```mysql
GRANT REPLICATION SLAVE ON *.* to 'xxx'@'%' identified by 'Root@123456';
flush privileges;
xxx为自定义用户名 
Root@123456 为自定义密码
==注：上面SQL的作用是创建一个用户 xxx ，密码为 Root@123456 ，并且给xiaoming用户授予REPLICATION SLAVE权限。常用于建立复制时所需要用到的用户权限，也就是slave必须被master授权具有该权限的用户，才能通过该用户复制。==
目前mysql5.7默认密码校验策略等级为 MEDIUM , 该等级要求密码组成为: 数字、小写字母、大写字母 、特殊字符、长度至少8位
```
2.4 登录Mysql数据库，执行下面SQL，记录下结果中File和Position的值
```mysql
show master status;
上面SQL的作用是查看Master的状态，执行完此SQL后主库不要再执行任何操作!切记！
```
2.5 配置-从库Slave
```mysql
修改Mysql数据库的配置文件/etc/my.cnf
[mysqld]
server-id=101 #[必须]服务器唯一ID
```
2.6 重启从库Mysql服务
systemctl restart mysqld
2.7 登录从库，执行下面连接主库SQL语句---**重要配置！关键配置！**
```mysql
#下面是关键配置，是一行执行
#其中，master_host为主库IP地址，master_user为上面主库新建的从库用户
#master_password为从库用户密码
#master_log_file为2.4查询出的对应文件File--注意！一定要和主库保持一致！
#master_log_pos为2.4查询出的对应Position的值--注意！一定要和主库保持一致！
change master to master_host='192.168.138.100',master_user='xiaoming',master_password='Root@123456',master_log_file='mysql-bin.000005',master_log_pos=441;
# 启动从库
start slave;
#使用列形式打印--查看主从配置是否都是yes
#通过状态信息中的 Slave_IO_running 和 Slave_SQL_running 可以看出主从同步是否就绪，如果这两个参数全为Yes，表示主从同步已经配置完成。
show slave status\G;   
```
> MySQL命令行技巧： 
>
> ​	\G : 在MySQL的sql语句后加上\G，表示将查询结果进行按列打印，可以使每个字段打印到单独的行。即将查到的结构旋转90度变成纵向；

#### 虚拟机克隆导致配置问题
如果是通过克隆虚拟机方式创造第二台虚拟机，那么它们的server_uuid 是同一个，这是mysql在启动的时候自动生成的一个身份id。
1. 查询 auto.cnf 文件的所在位置
```
find / -name 'auto.cnf'
```
2. 查看该文件的内容 【下面的地址是通过上面的命令查询出来的。大家的地址可能不同。】
```
cat /var/lib/mysql/auto.cnf
```
3. 把这个文件改名，然后重启mysql即可，mysql会重新生成这个文件以及uuid值
```
mv /var/lib/mysql/auto.cnf /var/lib/mysql/auto.cnf.bak  | rm –rf /var/lib/mysql/auto.cnf
systemctl restart mysqld
```
#### 3.  ShardingJDBC实现读写分离步骤
Sharding-JDBC定位为轻量级Java框架，在Java的JDBC层提供的额外服务。 它使用客户端直连数据库，以jar包形式提供服务，无需额外部署和依赖，可理解为增强版的JDBC驱动，完全兼容JDBC和各种ORM框架。
使用Sharding-JDBC可以在程序中轻松的实现数据库读写分离。
1. 导入maven坐标
2. 在配置文件中配置读写分离规则
3. 在配置文件中配置允许bean定义覆盖配置项
##### 3.1 项目添加ShardingJDBC起步依赖
```java
<!--shardingJDBC起步依赖-->
        <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
            <version>4.0.0-RC1</version>
        </dependency>
```
##### 3.2  配置ShardingJDBC数据源 管理多个数据库 实现读写分离,配置application.yml
```yml
spring:
		  shardingsphere:
			datasource:
			  names:
				master,slave
			  # 主数据源
			  master:
				type: com.alibaba.druid.pool.DruidDataSource
				driver-class-name: com.mysql.cj.jdbc.Driver
				url: jdbc:mysql://数据库主库网址地址:3306/rw_demo?characterEncoding=utf-8
				username: 数据库主库用户名
				password: 数据库密码
			  # 从数据源
			  slave:
				type: com.alibaba.druid.pool.DruidDataSource
				driver-class-name: com.mysql.cj.jdbc.Driver
				url: jdbc:mysql://数据库从库网址地址:3306/rw_demo?characterEncoding=utf-8
				username: 数据库从库库用户名
				password: 数据库密码
			masterslave:
			  # 读写分离配置
			  load-balance-algorithm-type: round_robin
			  # 最终的数据源名称
			  name: dataSource
			  # 主库数据源名称
			  master-data-source-name: master
			  # 从库数据源名称列表，多个逗号分隔
			  slave-data-source-names: slave
			props:
			  sql:
				show: true #开启SQL显示，默认false
```
##### 3.3 开启Spring设置 允许自定义的bean覆盖SpringBoot自动配置提供的bean对象
```yml
# 允许程序自定义的bean对象覆盖SpringBoot自动配置的bean对象
		spring:
		  main:
			allow-bean-definition-overriding: true
```