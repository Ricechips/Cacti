# Cacti

## 环境
> `服务端:Centos7`  `被监控端:win10`

## Cacti服务端搭建
> 安装Mysql<br>
> *rpm -Uvh http://repo.mysql.com/mysql57-community-release-el7.rpm*<br>
> *yum clean all && yum makecache*<br>
> 安装LAMP<br>
> *yum install httpd mysql-server php php-gd php-mysql -y*<br>
> 安装RRDtool<br>
> *yum install rrdtool rrdtool-devel.x84_64 rrdtool-perl.x86_64 -y*<br>
> 安装SNMP<br>
> *yum install net-snmp-utils -y*<br>
> 部署LAMP<br>
```c
/usr/bin/mysql –initialize –basedir=/usr/share/mysql –datadir=/var/lib/mysql/data/ 初始化Mysql
systemctl start mysqld
vim /etc/my.cnf 在末行添加 skip-grant-tables 跳过密码验证
systemctl restart mysqld
mysql -uroot -p
mysql>use mysql; 注意要写分号
mysql>update user set authentication_string=password(‘123456’) where user=‘root’;
exit退出到root vim /etc/php.ini  
date.timezone = “Asia/Shanghai”添加时区
systemctl start httpd
vi /var/www/html/phpinfo.php
添入测试文本<?php phpinfo(); ?> 
浏览器访问ip/phpinfo.php
```
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/2020-05-27%2014-41-45%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)


> 配置cacti<br>
```c
vim /etc/my.cnf添加
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
重启systemctl restart mysqld
mysql -uroot -p
mysql>create database cacti character set utf8;
mysql> ALTER DATABASE cacti CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
exit退出至root /etc/my.cnf 注释掉#skip-grant-tables 重启mysql
进入mysql
mysql>set global validate_password_policy=LOW;
mysql>set global validate_password_length=6;
mysql>alter user ‘root’@‘localhost’ identified by ‘123456’;
mysql>grant all privileges on cacti.* to cacti@localhost identified by ‘password’ ;
mysql>grant select on mysql.time_zone_name to cacti@localhost identified by ‘password’;
mysql>flush privileges;
```
>导入cacti数据库
