title: Centos安装OpenJDK
date: 2017/6/13 20:46:25
---

# Centos 安装jdk

取来自:http://stackoverflow.com/questions/20901442/how-to-install-jdk-in-centos

OpenJDK Runtime Environment (Java SE 6)

```
yum install java-1.6.0-openjdk
```

OpenJDK Runtime Environment (Java SE 7)

```
yum install java-1.7.0-openjdk
```

OpenJDK Development Environment (Java SE 7)

```
yum install java-1.7.0-openjdk-devel
```
<!--more-->
OpenJDK Development Environment (Java SE 6)

```
yum install java-1.6.0-openjdk-devel
```

------

**Update for Java 8**

In CentOS 6.6 or later, Java 8 is available. Similar to 6 and 7 above, the packages are as follows:

OpenJDK Runtime Environment (Java SE 8)

```
yum install java-1.8.0-openjdk
```

OpenJDK Development Environment (Java SE 8)

```
yum install java-1.8.0-openjdk-devel
```

There's also a 'headless' JRE package that is the same as the above JRE, except it doesn't contain audio/video support. This can be used for a slightly more minimal installation:

OpenJDK Runtime Environment - Headless (Java SE 8)

```
yum install java-1.8.0-openjdk-headless
```