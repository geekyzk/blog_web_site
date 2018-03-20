---
title: Centos7 源码编译 hadoop
layout: post 
---

最近心血来潮，想着重新搞一下hadoop，因此，就有了这篇文章，以下是环境的基本配置：

1. CentOS-7-x86_64-DVD-1611.iso
2. hadoop-2.7.3-src.tar.gz
3. jdk-8u121-linux-x64.tar.gz

## 一.下载hadoop源码包
`wget http://apache.fayea.com/hadoop/common/hadoop-2.7.3/hadoop-2.7.3-src.tar.gz`
通过这个命令，我们就可以在官网下载hadoop2.7.3的源码包
解压文件
`tar -xf hadoop-2.7.3-src.tar.gz`
查看编译所需要的配置信息，可以查看源码包里面的BUILDING.txt文件，里面有详细的说明，这里列出本次使用centos编译所需的。
```
* Unix System
* JDK 1.7+
* Maven 3.0 or later
* Findbugs 1.3.9 (if running findbugs)
* ProtocolBuffer 2.5.0
* CMake 2.6 or newer (if compiling native code), must be 3.0 or newer on Mac
* Zlib devel (if compiling native code)
* openssl devel ( if compiling native hadoop-pipes and to get the best HDFS encryption performance )
* Linux FUSE (Filesystem in Userspace) version 2.6 or above ( if compiling fuse_dfs )
* Internet connection for first build (to fetch all Maven and Hadoop dependencies)
```
下面，就通过以上信息，来配置系统的编译环境
<!--more-->

## 二.配置环境(直接使用的是root用户，无烦恼)

### 配置jdk1.8

首先，官网下载好jdk-8u121-linux-x64.tar.gz后，如下操作

1. 解压到指定目录，若指定没有，则必须提前创建。

   `tar -xf jdk-8u121-linux-x64.tar.gz -C /usr/local/software/`

2. 配置环境变量
  `vim /etc/profile.d/java.sh`
  在新建的文件里面，输入相关的配置,如下:
```shell
   export JAVA_HOME=/usr/local/software/jdk1.8.0_121
   export PATH=$JAVA_HOME/bin:$PATH
   export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
随后，执行命令，使环境变量生效
`source /etc/profile`
通过命令来检验，配置的java环境是否生效，如下
```shell
[root@localhost jdk1.8.0_121]# java -version & javac -version
[1] 13168
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
javac 1.8.0_121
[1]+  Done           
```
如上所示，则表明java环境变量已经成功生效了，即java环境已经成功配置好。

### 配置maven
通过maven的官网，下载maven的gz包，随后,如下操作:
1. 解压到指定目录
  `tar -xf apache-maven-3.5.0-bin.tar.gz -C /usr/local/software/`
2. 配置环境变量
- 创建环境变量文件
  `vim /etc/profile.d/maven.sh`
- 配置环境变量内容
```shell
export MAVEN_HOME=/usr/local/software/apache-maven-3.5.0
export PATH=$MAVEN_HOME/bin:$PATH
```
- 更新环境变量操作
  `source /etc/profile`
- 检验是否成功
```shell
[root@localhost apache-maven-3.5.0]# mvn -v
Apache Maven 3.5.0 (ff8f5e7444045639af65f6095c62210b5713f426; 2017-04-04T03:39:06+08:00)
Maven home: /usr/local/software/apache-maven-3.5.0
Java version: 1.8.0_121, vendor: Oracle Corporation
Java home: /usr/local/software/jdk1.8.0_121/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-514.16.1.el7.x86_64", arch: "amd64", family: "unix"
```
如上所示，则maven已经成功配置

### 配置findbugs
本次使用的使findbugs-3.0.1.tar.gz
- 解压
  ` tar -xf findbugs-3.0.1.tar.gz -C /usr/local/software/ `
- 配置环境变量
  ` vim /etc/profile.d/findbugs.sh`
  输入以下内容
  `export PATH=/usr/local/software/findbugs-3.0.1/bin:$PATH`
- 更新环境变量
  `source /etc/profile`
- 检验是否成功
```shell
[root@localhost findbugs-3.0.1]# findbugs -version
3.0.1
```
如上所示，则findbugs已安装成功

### 配置protobuf
下载protobuf2.5.0版本，随后，解压准备编译,这里提供一个下载地址:
> http://pan.baidu.com/s/1pJlZubT

- 解压
- 下载安装编译所需的环境
  `yum install autoconf automake libtool curl make g++ gcc-c++ unzip`
- 配置安装路径
  进入到解压后的目录，执行以下命令，设置安装路径
  `./configure --prefix=/usr/local/software/protobuf`
- 编译安装
```shell
$ make
$ make check
$ make install
```
- 配置环境变量
  `vim /etc/profile.d/protobuf.sh`
  输入以下内容
```
export PATH=/usr/local/software/protobuf/bin:$PATH
```
- 更新环境变量
  ` source /etc/profile`
- 检查是否生效
```shell
[root@localhost ~]# protoc --version
libprotoc 2.5.0
```
如上，则安装成功.

### 编译所需的包
```
yum install cmake openssl-dev zlib
```

## 编译开始
进入hadoop源码目录，执行以下命令
`mvn clean package -DskipTests -Pdist,native -Dtar`
则开始编译hadoop，编译非常依赖网，而且有时候会因为网络的问题，直接中断编译，则可以再次运行编译命令来重新编译(最好翻墙).
当下你是如下内容内容时，则证明编译成功了
```shell
[INFO] Apache Hadoop Distribution ......................... SUCCESS [ 26.246 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 35:31 min
[INFO] Finished at: 2017-05-14T02:29:25+08:00
[INFO] Final Memory: 206M/548M
```
在源码目录下则会生产一个hadoop-dist文件夹，里面就时编译后生成的东西，编译出的hadoop包就在target里面


