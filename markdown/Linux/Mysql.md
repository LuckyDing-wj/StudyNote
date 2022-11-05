[TOC]

# 一、CentOS7安装Mysql 8.0

## 1. 下载镜像

```shell
yum install -y wget

wget https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-8.0/mysql-8.0.23-el7-x86_64.tar.gz
```

## 2. 检查是否安装过mysql

如果以前用yum安装过，先用yum卸载，如果不是此方式或者没安装过，则跳过。

```shell
[root@VM-0-2-centos data]# yum remove mysql
```

查看是否有mysql依赖，如果有则卸载

```shell
[root@VM-0-2-centos data]# rpm -qa | grep mysql

//普通删除模式
# rpm -e xxx(mysql_libs)
//强力删除模式,如果上述命令删除时，提示有依赖其他文件，则可以用该命令对其进行强力删除
# rpm -e --nodeps xxx(mysql_libs)
```

## 3. 检查是否有mariadb

```shell
[root@VM-0-2-centos data]# rpm -qa | grep mariadb
mariadb-libs-5.5.68-1.el7.x86_64
```

如果有则卸载

```shell
[root@VM-0-2-centos data]# rpm -e --nodeps mariadb-libs
# 如果有 devel 也删除
[root@VM-0-2-centos data]# rpm -e --nodeps mariadb-devel-5.5.65-1.el7.x86_64
```

## 4. 安装mysql依赖包

```shell
[root@VM-0-2-centos data]# yum install libaio
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * centos-sclo-rh: ftp.sjtu.edu.cn
 * centos-sclo-sclo: ftp.sjtu.edu.cn
centos-sclo-rh                                         | 3.0 kB  00:00:00
centos-sclo-sclo                                       | 3.0 kB  00:00:00
docker-ce-stable                                       | 3.5 kB  00:00:00
epel                                                   | 4.7 kB  00:00:00
extras                                                 | 2.9 kB  00:00:00
mongodb-org-4.4                                        | 2.5 kB  00:00:00
os                                                     | 3.6 kB  00:00:00
updates                                                | 2.9 kB  00:00:00
(1/4): epel/7/x86_64/updateinfo                        | 1.0 MB  00:00:00
(2/4): epel/7/x86_64/primary_db                        | 6.9 MB  00:00:01
(3/4): updates/7/x86_64/primary_db                     | 8.8 MB  00:00:01
(4/4): mongodb-org-4.4/7/primary_db                    |  49 kB  00:00:01
Package libaio-0.3.109-13.el7.x86_64 already installed and latest version
Nothing to do
[root@VM-0-2-centos data]#
```

## 5. 解压

```sh
[root@VM-0-2-centos data]# tar -zxvf mysql-8.0.23-el7-x86_64.tar.gz
```

将文件重命名并移动

```sh
[root@VM-0-2-centos data]# mv mysql-8.0.23-el7-x86_64 /usr/local/mysql
[root@VM-0-2-centos data]# cd /usr/local/mysql/
[root@VM-0-2-centos mysql]# ls
bin  docs  include  lib  LICENSE  man  README  share  support-files
```

创建数据库文件存放文件夹

```sh
[root@VM-0-2-centos data]# cd /usr/local/mysql/
[root@VM-0-2-centos mysql]# mkdir mysqldb
[root@VM-0-2-centos mysql]#
```

安装目录赋予权限

```sh
[root@VM-0-2-centos local]# chmod -R 777 /usr/local/mysql/
[root@VM-0-2-centos local]#
```

## 6. 创建mysql组和用户

创建组

```sh
[root@VM-0-2-centos mysql]# groupadd mysql
```

创建用户(-s /bin/false参数指定mysql用户仅拥有所有权，而没有登录权限)

```sh
[root@VM-0-2-centos mysql]# useradd -r -g mysql -s /bin/false mysql
```

将用户添加到组中

```sh
[root@VM-0-2-centos mysql]# chown -R mysql:mysql ./
```

## 7. 修改配置文件

```sh
# 如果没有此文件，先创建
[root@VM-0-2-centos mysql]# touch /etc/my.cnf
[root@VM-0-2-centos mysql]#
[root@VM-0-2-centos mysql]# vi /etc/my.cnf
```

设置如下：

```tex
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=/usr/local/mysql
# 设置mysql数据库的数据的存放目录
datadir=/usr/local/mysql/mysqldb
# 允许最大连接数
max_connections=10000
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
```

## 8. 安装mysql

进入mysql目录进行安装

```sh
[root@VM-0-2-centos bin]# pwd
/usr/local/mysql/bin
[root@VM-0-2-centos bin]# ./mysqld --initialize --console
./mysqld: error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory
[root@VM-0-2-centos bin]#
```

出现错误，缺少依赖，安装依赖

```sh
yum -y install numactl
```

重新安装即可正常

```sh
[root@VM-0-2-centos bin]#
[root@VM-0-2-centos bin]# ./mysqld --initialize --console
2021-06-19T03:51:38.252153Z 0 [System] [MY-013169] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.23) initializing of server in progress as process 17064
2021-06-19T03:51:38.253453Z 0 [Warning] [MY-013242] [Server] --character-set-server: 'utf8' is currently an alias for the character set UTF8MB3, but will be an alias for UTF8MB4 in a future release. Please consider using UTF8MB4 in order to be unambiguous.
2021-06-19T03:51:38.259503Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2021-06-19T03:51:39.305222Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2021-06-19T03:51:41.967937Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: mqjgo99gk2.W
[root@VM-0-2-centos bin]#
[root@VM-0-2-centos bin]#
[root@VM-0-2-centos bin]#
```

记录初始化密码：mqjgo99gk2.W

## 9. 启动mysql服务

进入mysql.server服务目录下并启动服务

```sh
[root@VM-0-2-centos support-files]# pwd
/usr/local/mysql/support-files
[root@VM-0-2-centos support-files]#
[root@VM-0-2-centos support-files]# ./mysql.server start
Starting MySQL.Logging to '/usr/local/mysql/mysqldb/VM-0-2-centos.err'.
 ERROR! The server quit without updating PID file (/usr/local/mysql/mysqldb/VM-0-2-centos.pid).
[root@VM-0-2-centos support-files]#
```

第一次启动，初始化时出现错误，重新给mysql目录赋予权限，再次执行

```sh
[root@VM-0-2-centos support-files]# chmod -R 777 /usr/local/mysql/
[root@VM-0-2-centos support-files]# ./mysql.server start
Starting MySQL.. SUCCESS!
[root@VM-0-2-centos support-files]#
```

启动成功

## 10. 添加到系统进程

```sh
[root@VM-0-2-centos support-files]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
[root@VM-0-2-centos support-files]#
```

设置mysql自启动

```sh
[root@VM-0-2-centos support-files]# chmod +x /etc/init.d/mysqld
[root@VM-0-2-centos support-files]# systemctl enable mysqld
mysqld.service is not a native service, redirecting to /sbin/chkconfig.
Executing /sbin/chkconfig mysqld on
[root@VM-0-2-centos support-files]#
```

## 11. 修改root用户登录密码

登录mysql，输入第8步中记录的初始化密码

```sh
[root@VM-0-2-centos bin]# pwd
/usr/local/mysql/bin
[root@VM-0-2-centos bin]# ./mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.23

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

修改密码

```sh
mysql> alter user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Your Password';
Query OK, 0 rows affected (0.00 sec)
mysql>
```

## 12. 设置允许远程登陆

```sh
mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>
mysql>
mysql> update user set user.Host='%' where user.User='root';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql>
mysql>
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql>
mysql>
mysql>quit
```

## 13. 重启服务

```sh
# 重启
[root@VM-0-2-centos bin]# systemctl restart mysql
# 查看状态
[root@VM-0-2-centos bin]# systemctl status mysql
● mysqld.service - LSB: start and stop MySQL
   Loaded: loaded (/etc/rc.d/init.d/mysqld; bad; vendor preset: disabled)
   Active: active (exited) since Sat 2021-06-19 12:12:14 CST; 53s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 20573 ExecStart=/etc/rc.d/init.d/mysqld start (code=exited, status=0/SUCCESS)

Jun 19 12:12:14 VM-0-2-centos systemd[1]: Starting LSB: start and stop MySQL...
Jun 19 12:12:14 VM-0-2-centos mysqld[20573]: Starting MySQL SUCCESS!
Jun 19 12:12:14 VM-0-2-centos systemd[1]: Started LSB: start and stop MySQL.
Jun 19 12:12:14 VM-0-2-centos mysqld[20573]: 2021-06-19T04:12:14.250239Z mysqld_safe A mysqld process...ists
Hint: Some lines were ellipsized, use -l to show in full.
[root@VM-0-2-centos bin]#
```

查看linux防火墙开放端口

```sh
firewall-cmd --list-all
```

如果没有开放3306，则开放

```sh
[root@centos7 bin]# firewall-cmd --zone=public --add-port=3306/tcp --permanent
[root@centos7 bin]# firewall-cmd --reload
//--permanent为永久生效，没有此参数 服务器重启后配置失效
```

## 14. 重置root密码

如果忘记了root账号密码，按以下步骤重置：

在my.cnf第二行添加如下配置，表示不检查权限表，即任何人都可以登录数据库。

```tex
[mysqld]
skip-grant-tables
```

重启mysql服务

```sh
[root@VM-0-2-centos bin]# systemctl restart mysqld.service
```

登录mysql，password直接回车

```sh
[root@VM-0-2-centos bin]# 
[root@VM-0-2-centos bin]# systemctl restart mysqld.service
[root@VM-0-2-centos bin]# ./mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.23 MySQL Community Server - GPL

```

将root账号，密码字段置空

```sh
mysql> update user set authentication_string='' where user='root';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql>
```

将my.cnf中添加的skip-grant-tables去除，重启mysql

```sh
[mysqld]
# skip-grant-tables
```

```sh
[root@VM-0-2-centos bin]# systemctl restart mysqld.service
```

无需密码登录mysql

```sh
[root@VM-0-2-centos bin]# pwd
/usr/local/mysql/bin
[root@VM-0-2-centos bin]# ./mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.23 MySQL Community Server - GPL

```

修改root密码，在上面的步骤中，设置远程登录时将localhost修改为%，此处也要用%

```sh
mysql> alter user 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'YOU PASSWORD';
Query OK, 0 rows affected (0.01 sec)

mysql> 
```

修改完成，测试登录。

test

