title: Centos7 安装mysql5.7
date: 2017/6/13 20:46:25
---

**MySQL**是一种基于GNU（通用公共许可证）发布的开源免费关系数据库管理系统（RDBMS）。它用于通过为每个创建的数据库提供多用户访问，在任何单个服务器上运行多个数据库。

## 一.添加官方yum源

首先，下载官方Centos7的yum源:

`wget http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm`  

随后，安装官方yum源:

`yum localinstall mysql57-community-release-el7-7.noarch.rpm`

最后，更新yum:

`yum update -y`
<!--more-->
检查yum库，是否已经存在最新的mysql:

```shell
[root@localhost ~]# yum repolist | grep "mysql*"
mysql-connectors-community/x86_64       MySQL Connectors Community           36
mysql-tools-community/x86_64            MySQL Tools Community                47
mysql57-community/x86_64                MySQL 5.7 Community Server          187

```

如上所示，则成功添加了官方的yum源

## 二.安装mysql

通过上面的步骤，我们已经成功的添加和更新了yum源，现在我们则可以安装mysql。

`yum install mysql-community-server`

通过上面的命令，将会安装mysql-community-server，mysql-community-client，mysql-community-common和mysql-community-libs所需的所有软件包。

## 三.启动mysql

现在，我们可以通过**`service mysqld start`**命令来启动mysql。随后，使用**`service mysqld status`**来检验mysql的状态，如下:

```shell
[root@localhost ~]# service mysqld status
Redirecting to /bin/systemctl status  mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2017-05-14 17:41:31 CST; 52min ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 2607 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 1215 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 2610 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─2610 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

May 14 17:41:28 localhost.localdomain systemd[1]: Starting MySQL Server...
May 14 17:41:31 localhost.localdomain systemd[1]: Started MySQL Server.

```

当启动成功后，我们可以在`/var/log/mysqld.log`里面查看系统生成的随机密码(mysql5.7或更高版本以上才有),如下:

```shell
[root@localhost ~]# grep 'temporary password' /var/log/mysqld.log
2017-05-14T08:48:02.531831Z 1 [Note] A temporary password is generated for root@localhost: prkOkfWX=9Ty
```

如果不确定安装版本，我们也可以使用**`mysql --version`**来查看安装的版本，如下:

```shell
[root@localhost ~]# mysql --version
mysql  Ver 14.14 Distrib 5.7.18, for Linux (x86_64) using  EditLine wrapper
```

## 四.设置mysql

通过上面三个步骤，我们就已经成功安装好了mysql并运行了起来,现在，就可以设置一些mysql的属性

> 注意：一定要记住mysql生成的随机密码，方便后面登陆后修改，这个密码只是临时的

现在，可以运行mysql_secure_installation来设置一些基本的属性,如下

`[root@localhost ~]# mysql_secure_installation`

可以根据提示来设置一些mysql的基本设置，如是否运行远程登陆，是否删除test数据库.

> 注意:在重设密码使，设置的密码应包含大小写字母数字和特殊字符，不然不给通过。

## 五.连接mysql

通过上面步骤四，我们就已经设置好基本的配置，当然，也可以使用临时密码来直接跳过第四步，从第五步开始，成功连接上mysql后通过sql语句来配置mysql，这些具体的不在这里讨论，以下展示如何连接上安装好的mysql.

```shell
[root@localhost ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.18 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

```

如上所示，我们已经成功连接上mysql，即mysql的安装和配置也已经完成了，至于更详细的配置和优化，就不在这里讨论了。