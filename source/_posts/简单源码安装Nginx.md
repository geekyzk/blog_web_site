title: 简单源码安装Nginx
date: 2017/6/13 20:46:25
---

昨天，已经通过yum的方式来成功安装Nginx，由于yum这种方式只是适合红帽系，因此，我们可以通过使用源码安装的方式，来在不同的发行版本上成功安装Nginx，当然，例如ubuntu等也可以通过apt-get这种方式安装。
<!--more-->

## 一.准备环境

根据官网的说明，我们在编译前，需要配置一些列的编译环境，具体如下：

- pcre
- zlib
- openssl

我们在这里以centos为例来安装这些依赖，如下：

```shell

yum install -y pcre pcre-devel  
yum install -y zlib zlib-devel  
yum install -y openssl openssl-devel  
yum install -y gcc cmake

```

随后，我们下载当前nginx的稳定版本

```shell
wget http://nginx.org/download/nginx-1.12.0.tar.gz
```

解压

```shell
tar -xf nginx-1.12.0.tar.gz
```

完成以上步骤，我们的编译环境和源码包基本就绪，后面可以开始编译。



## 二.编译Nginx

编译Nginx和编译Hadoop等是差不多的，都是执行相同的命令，如下:

### 第一步.配置configure

我们在编译的适合，需要配置configure来生成编译所需的`Makefile` 文件，如下(在nginx源码目录下操作):

```shell
./configure
--sbin-path=/usr/local/nginx/nginx
--conf-path=/usr/local/nginx/nginx.conf
--pid-path=/usr/local/nginx/nginx.pid
```

当然，这里也可以配置更多的参数，参数说明可以参考帮助

- --sbin-path：指定了nginx运行文件所在位置
- --conf-path：指定了nginx配置文件所在位置
- --pid-path：指定了运行时产生的pid文件所在位置

### 第二步.make&make install

当生成了`Makefile` ，我们就可以执行以下命令，来编译nginx(基于前一步，在nginx源码目录下操作)：

```shell
$ make
$ make install
```

当执行完这两步以后，nginx就成功安装好了，我们可以进入到`/usr/local/nginx`目录下，执行`./nginx`，nginx就可以运行了，我们也可以通过访问localhost来查看nginx的欢迎界面，来确认是否运行成功

## 三.配置开机自动启动(非必需)

自己手动编译的一个问题就是，nginx并不会成为系统的一个服务，来开机自动启动，因此，我们需要把nginx添加到服务中，来让其在系统启动后就自动启动。

首先，我们新建并编辑`/etc/init.d/nginx`文件

```shell
vim /etc/init.d/nginx
```

随后，我们在新建的文件中，输入以下内容

```shell
#!/bin/bash
# chkconfig: 2345 85 15
# Startup script for the nginx Web Server
# description: nginx is a World Wide Web server. 
# It is used to serve HTML files and CGI.
# processname: nginx
# pidfile: /usr/local/nginx/nginx.pid
# config: /usr/local/nginx/nginx.conf

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DESC="nginx deamon"
NAME=nginx
DAEMON=/usr/local/nginx/$NAME
SCRIPTNAME=/etc/init.d/$NAME
PID_PATH=/usr/local/nginx/nginx.pid

test -x $DAEMON || exit 0

d_start(){
  $DAEMON || echo -n "already running"
}

d_stop(){
  $DAEMON -s quit || echo -n "not running"
}


d_reload(){
  $DAEMON -s reload || echo -n "can not reload"
}

d_status(){
	if [ -e $PID_PATH ];then
		echo "service is running"
	else
		echo "service not running"
	fi
}

case "$1" in
start)
  echo -n "Starting $DESC: $NAME"
  d_start
  echo "."
;;
stop)
  echo -n "Stopping $DESC: $NAME"
  d_stop
  echo "."
;;
reload)
  echo -n "Reloading $DESC conf..."
  d_reload
  echo "reload ."
;;
restart)
  echo -n "Restarting $DESC: $NAME"
  d_stop
  sleep 2
  d_start
  echo "."
;;
status)
  d_status
;;
*)
  echo "Usage: $ScRIPTNAME {start|stop|reload|restart|status}" >&2
  exit 3
;;
esac

exit 0
```

随后，wq保存，为这个nginx文件添加执行权限

```shell
chmod +x nginx
```

随后，添加到计划中

```shell
chkconfig --add nginx  
```

设置在指定范围内，都开机启动

```shell
chkconfig nginx on
```

通过上面的设置，我们的nginx就可以开机自动启动了。