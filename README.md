# Cacti
>蛮麻烦的
## 环境
> `服务端:Centos7`  `被监控端:win7`

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
mysql>ALTER DATABASE cacti CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
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
mysql>source /root/cacti-1.2.1/cacti.sql ;
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

web初始化cacti
setenforce 0关闭防火墙 临时关闭或
vi /etc/selinux/config  将SELINUX=enforcing改为SELINUX=disabled 永久关闭
浏览器访问ip/cacti
log报错修改：chmod 777 /var/www/html/cacti/log
默认密码admin/admin
```
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/2020-05-27%2016-48-51%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

>一些报错修改（不修改不能继续安装）<br>
>1.报错：Your MySQL TimeZone database is not populated. Please populate this database before proceeding.<br>
>修改：mysql_tzinfo_to_sql /usr/share/zoneinfo/Asia/Shanghai Shanghai | mysql -u root -p mysql<br>
>2.报错：A valid timezone that matches MySQL and the system<br>
>修改：vim /etc/php.ini 将data.timezone前面的分号去掉<br>
>3.报错：某些php模块显示没有安装<br>
>修改：yum install -y php-缺少的模块名,安装之后systemctl restart httpd并刷新cacti网页<br>
>4.Set global max_allowed_packet=17700000;<br>
>5.权限问题（不可写入）chmod a+w /var/www/html/cacti/resource/snmp_queries/或777<br>
>6.cacti无法显示图像：chmod 777 -R /var/www/html/cacti/rra并service snmpd restart<br>
>7.监控本机localhost，snmp返回失败：修改主机名为127.0.0.1而不是localhost

## ok
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/2020-05-27%2017-45-41%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

## 被监控端Win7
>控制面板开启SNMP服务<br>
>设置服务团体public,允许任何主机访问<br>
>关闭防火墙<br>
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/2020-05-28%2010-08-41%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)
>yum install perl-devel perl-CPAN perl-YAML<br>
>perl -MCPAN -e shell<br>
>cpan> install LWP::UserAgent


## 被监控端Centos7
>yum -y install net-snmp net-snmp-utils<br>
>/etc/snmp/snmpd.conf<br>
>com2sec notConfigUser 192.168.106.201 public<br>
>access notConfigGroup '''' any noauth exact all none none<br>
>view all included .1 80<br>
>view systemview included .1.3.6.1.2.1.2<br>
>service snmpd start<br>
>测试 snmpwalk -v 2c-c public 192.168.106.

## 监控ESXI
>vsphere client打开esxi的ssh服务<br>
>putty上去配置snmp(/etc/vmware/snmp.xml）<br>
```c
<config>
<snmpSettings>
<enable>true</enable> #将标签false改为true.
<port>161</port>
<EnvEventSource>indications</EnvEventSource>
<loglevel>info</loglevel>
<communities>public</communities>  #社区名称
<targets></targets>
</snmpSettings>
</config>
```
>cacti添加esxi主机(同一网段)<br>
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/2020-06-02%2015-51-40%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

## 监控Cisco交换机
>交换机用com口转usb线连接到win，putty通过com3口登陆进行snmp的配置(网段ip)<br>
>服务器添加交换机<br>
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/2020-06-03%2010-30-37%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

## 配置邮件报警
>qq邮箱开启smtp并获得授权码<br>
>在cacti管理界面输入相应信息并发送测试邮件
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/2020-06-03%2014-01-00%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

## 升级Cacti
>因插件版本问题，将原本的Cacti1.2.1升级到最新版1.2.12(2020.5.3)
```c
备份数据库 mysqldump -uroot -p123456 -l --add-drop-table cacti > cacti_old.sql
备份cacti根目录 mv /var/www/html/cacti /var/www/html/cacti_old
官网下载最新版包mv cacti-1.2.12/ /var/www/html/cacti
vim /data/www/cacti/include/config.php 数据库配置同前
cp /var/www/html/cacti_old/rra/* /var/www/html/cacti/rra/
cp -u /var/www/html/cacti_old/scripts/* /var/www/html/cacti/scripts/
cp -u -R /var/www/html/cacti_old/resource/* /var/www/html/cacti/resource/
```
>刷新管理平台界面并安装
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/2020-06-03%2014-46-40%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

## 配置Thold警报机制
>从cacti的github界面下载插件thold包[跳转](https://github.com/Cacti/plugin_thold/archive/develop.zip)<br><br>
>将包解压后放在/var/www/html/cacti/plugins路径下并改名为thold<br>
>刷新控制台，安装插件<br>
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/2020-06-04%2010-31-42%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)
>根据图形模版来设置阈值
![avatar](https://github.com/Ricechips/Cacti/blob/master/PrtScn/TIM%E5%9B%BE%E7%89%8720200604104626.png)
