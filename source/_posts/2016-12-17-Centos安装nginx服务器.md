---
title: Centos安装nginx服务器
layout: post 
---

## 一.Nginx简介


引用官方的话来说，Nginx是一个免费的，开源的，高性能的HTTP服务器和反向代理，以及一个IMAP / POP3代理服务器。NGINX以其高性能，稳定性，丰富的功能集，简单的配置和低资源消耗而闻名。若想了解更多，可以访问https://www.nginx.com/resources/wiki/ 。



## 二.Nginx安装

### 第一步：添加yum源

根据官方的说明，我们可以添加yum的配置文件，来通过yum安装Nginx，如下：

`vim /etc/yum.repos.d/nginx.repo`

使用vim打开后，在里面输入以下内容：

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```
<!--more-->
使用wq来保存并退出。

这里顺便说一下，redhat系统配置也是和centos差不多的，只是yum文件的内容略有区别，如下:

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/rhel/$releasever/$basearch/
gpgcheck=0
enabled=1
```

### 第二步：更新yum源

通过上面添加yum源的步骤后，我们就可以通过以下命令来更新yum源

**`yum update -y`**

当更新后，我们可以使用以下命令来查看是否存在生效

**`yum repolist nginx*`**

若显示的内容里面包含有nginx，则更新成功

### 第三步：安装，运行Nginx

接下来，我们就开始安装Ngxin了，如下：

**`yum install nginx`**

当安装成功，我们可以通过以下命令来启动Nginx：

**`service nginx start`**

随后，可通过以下命令来检验Nginx的运行状态:

**`service nginx status`**

一般来说，通过以上步骤，Nginx就已经成功安装了。

### 注意事项

当我们成功安装运行后，可以访问localhost来再检查以下是否成功，若显示Welcome to nginx!的页面，则成功，但是，若是系统开启了防火墙，其他电脑还是访问不了本机的Nginx服务器的，要通过一下配置，更新一下防火墙的规则，才能是其他电脑访问，如下：

```shell
firewall-cmd --permanent --zone=public --add-service=http 
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```

当更新成功后，其他电脑就可以访问本机的nginx了

##　三.Nginx配置了解

通过上述步骤，一个nginx已经安装好，但是我们还不知道Nginx是怎么配置监听端口的，为什么默认就是80端口呢？默认显示的页面是在哪配置的呢？这就要涉及到Nginx的配置文件了。

安装成功后，nginx的默认配置文件是在**/etc/ningx/**文件夹里面,而说明默认端口之类的，则是在**/etc/nginx/conf.d/default.conf**文件中，由于文件内容太占篇幅，下面就举出几个属性说明:

- location / {}配置的就是页面显示的路径，其中里面的root就是html存放的路径,index就是默认显示哪个页面
- listen 80 则是说明了监听的端口。

这里就简单说明这两个配置，通过这两个配置，我们也可以解决了我们前面的疑惑了。更详细的配置，请阅读相关文章。