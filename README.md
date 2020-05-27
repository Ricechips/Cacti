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

