title: CloudFoundry部署
date: 2017/6/13 20:46:25
---

本文主要介绍通过bosh工具，在vsphere平台上部署CloudFoundry

##  一、工具代码说明

- Bosh-cli

  Bosh的命令行工具

- Bosh-deployment

  Bosh的部署配置代码

- cf-deployment

  CloudFoundry的部署配置代码

- vsphere

  Vsphere管理工具的账号密码及可用集群(默认使用最高权限账号，也可根据需要创建单独的账号


## 二、安装步骤

下面讲解工具的安装步骤与代码的下载与配置，默认操作环节为Centos7。

### 1、Bosh-Cli

通过官方下载地址下载最新版本的命令行二进制文件。