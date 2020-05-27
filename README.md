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
>我的配置过程
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/2020-05-27%2014-53-38%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)
>导入cacti数据库
官网下载包[跳转](https://www.cacti.net/download_cacti.php)
解压到目录下
```c
mysql -uroot -p123456
mysql>use cacti ;
mysql > source /root/cacti-1.2.1/cacti.sql ;
在/var/www/html/下创建mkdir cacti
cp -r /root/cacti-1.2.1/* /var/www/html/cacti

vi /var/www/html/cacti/include/config.php修改配置信息
$database_type = ‘mysql’;
$database_default = ‘cacti’;
$database_hostname = ‘localhost’;
$database_username = ‘cacti’;
$database_password = ‘password’;
$database_port = ‘3306’;
$database_ssl = false;

增加cacti用户权限
useradd -s /sbin/nologin cacti
mkdir /var/www/html/cacti/rra/log
chown -R cacti /var/www/html/cacti/rra/log/

配置定时任务crontab
crontab –e
添入内容*/5 * * * * /usr/bin/php /var/www/html/cacti/poller.php > /dev/null 2>&1
crontab -l有输出即正常
systemctl enable crond
systemctl start crond

安装其他相关组件
https://oss.oetiker.ch/rrdtool/pub/rrdtool-1.7.0.tar.gz
https://www.cacti.net/downloads/spine/cacti-spine-1.2.1.tar.gz
编译时需要的软件包
yum install glib2-devel cairo-devel libxml2-devel pango pango-devel help2man

解压后cd rrdtool-1.7.0
./configure --prefix=/usr/local/rrdtool
make
make install

cd cacti-spine-1.2.1
./configure --prefix=/usr/local/spine
make
make install
编辑/usr/local/spine/etc/spine.conf
vi /usr/local/spine/etc/spine.conf修改配置信息
DB_Host localhost
DB_Database cacti
DB_User cacti
DB_Pass password
DB_Port 3306

