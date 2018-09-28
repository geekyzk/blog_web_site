title: 基于Vsphere平台使用cf-deployment部署CloudFoundry
date: 2018/04/02 23:46:25
---
&emsp; &emsp;本文主要讲解在vsphere平台上，开源PaaS平台(Cloudfoundry)的安装，本文使用了[cf-deployment](https://github.com/cloudfoundry/cf-deployment.git)部署CloudFoundry，原有的[cf-realease](https://github.com/cloudfoundry-attic/cf-release)部署版本已停止维护。

## 1、部署涉及主要工具与说明

- **vsphere环境**
- Bosh命令行工具(bosh-cli)
- Bosh部署执行代码(bosh-deployment)
- CloudFoundry部署执行代码(cf-deployment)
- CloudFoundry命令行工具(cf-cli)

<!--more-->
## 2、部署流程

&emsp;&emsp;下面开始从0开始介绍如何一步步的使用cf-deployment部署CloudFoundry。

### 2.1 vsphere环境准备

&emsp;&emsp;由于本次安装是基于vsphere平台，因此，下面介绍一些vsphere平台所必须满足的因素:

- vsphere平台管理员账号
- vsphere所管理的虚拟机需能连接互联网
- 部署所用的vsphere网段必须能够互通

&emsp;&emsp;上述的列表就是我们vsphere所需满足的部署条件，下面假设本次安装的vsphere环境信息如下：

- vsphere账号：root/123456
- vsphere管理地址: 10.100.3.11
- 数据中心名称: DataCenter
- 集群名称：PaaS
- 资源池名称：PaaS
- 网卡名称：NetPaaS
- 部署网段：192.168.7.1/24
- 网关地址为: 192.168.7.2
- DNS服务器地址为: 192.168.7.1 
- 存储名称：StorePaaS

&emsp;&emsp;假设本次可用ip为整个7的网段。

### 2.2 Bosh环境部署

 &emsp;&emsp;PaaS平台是使用Bosh这个工具进行部署。因此，我们必须先安装Bosh工具，以下是安装步骤：

1. 虚拟机准备

   首先，我们必须准备一台虚拟机来安装bosh-cli这个工具，此次安装的虚拟机信息如下:

   - ip : 192.168.7.2
   - 操作系统: Centos7

   > 注意：本次部署直接ip从192.168.7.2开始的原因是方面后面设置dns服务器为192.168.7.1，个人可根据实际情况来进行ip的设置与分配

2. Bosh-cli 安装

   本次使用的版本为`3.0.1`.最新版本可到[bosh官网](https://bosh.io/docs/cli-v2.html)处下载，操作步骤如下：

   - 下载命令行工具(`wget https://s3.amazonaws.com/bosh-cli-artifacts/bosh-cli-3.0.1-linux-amd64`)
   - 更改工具的权限(`chmod +x bosh-cli-*`)
   - 移动到系统默认执行文件存放目录并重命名为bosh(`mv bosh-cli-* /usr/local/bin/bosh`)

&emsp;&emsp;上述的步骤中的第三步，也可以把其移动到我们自定义的文件夹中，但必须把此文件夹的路径添加到系统环境变量`PATH`中。

3. 环境依赖安装

&emsp;&emsp;下述为在Centos中安装依赖的方式，如果是其他linux系统，如Ubuntu，请参考[官方手册](https://bosh.io/docs/cli-env-deps.html)。

   ```shell
   $ sudo yum install gcc gcc-c++ ruby ruby-devel mysql-devel postgresql-devel postgresql-libs sqlite-devel libxslt-devel libxml2-devel patch openssl git
   $ gem install yajl-ruby
   ```

4. 获取Bosh director环境创建执行代码

&emsp;&emsp;Bosh管控平台部署生成的虚拟机，主要是通过Bosh director，他为独立一台虚拟机。Bosh的最新代码我们可以在bosh官方的[github仓库](https://github.com/cloudfoundry/bosh-deployment)中获取，下面为获取的步骤

   ```Shell
   $ git clone https://github.com/cloudfoundry/bosh-deployment
   ```

5. 创建Bosh director

&emsp;&emsp;我们本次分配Bosh director这台虚拟机ip为`192.168.7.3`。通过创建环境的命令，bosh会自动给我们创建此虚拟机。

   ```shell
   $ bosh create-env bosh-deployment/bosh.yml \
       --state=state.json \
       --vars-store=creds.yml \
       -o bosh-deployment/vsphere/cpi.yml \
       -v director_name=paas \
       -v internal_cidr=192.168.7.1/24 \
       -v internal_gw=192.168.7.254 \
       -v internal_ip=192.168.7.3 \
       -v network_name="NetPaaS" \
       -v vcenter_dc=DataCenter \
       -v vcenter_ds=StorePaaS \
       -v vcenter_ip=10.100.3.11 \
       -v vcenter_user=root \
       -v vcenter_password=123645 \
       -v vcenter_templates=paas-1-templates \
       -v vcenter_vms=paas-1-vms \
       -v vcenter_disks=paas-1-disks \
       -v vcenter_cluster=PaaS
   ```

&emsp;&emsp;通过上述命令，我们就可以创建完成Bosh director。在创建的过程中，会需要从外网下载创建依赖，因此，需要保证网络的可用性，最好能连接到国外aws的s3服务器。

6. 检验bosh环境

&emsp;&emsp;在上一步的操作中，我们可以看到一个参数`—vars-store=creds.yml`。在部署的过程中，生成部署的证书，bosh的`admin`密码等都会存储到此文件中，我们可以通过以下步骤来登录`Bosh director`.

```shell
#为我们创建的Bosh director指定一个别名，并指定所用的证书
$ bosh alias-env bosh -e 192.168.7.3 --ca-cert <(bosh int ./creds.yml --path /director_ssl/ca)
# 设置Bosh登录的用户名
$ export BOSH_CLIENT=admin
# 设置Bosh登录的密码
$ export BOSH_CLIENT_SECRET=`bosh int ./creds.yml --path /admin_password`
# 检查环境
$ bosh -e bosh env
```

7. 创建与更新cloud-config配置

&emsp;&emsp;从`Bosh V2`版本开始，Bosh就支持通过配置cloud-config信息的方式来配置通过其部署的软件的部署信息，把部署的配置能做到统一管理，简化软件的部署配置。下面列举出部分cloud-config能配置的信息，更多信息可到[官方cloud-config说明](https://bosh.io/docs/cloud-config.html)中查看。下面是本次部署的cloud-config配置文件配置:

```yml
---
azs:
- name: z1
  cloud_properties:
    datacenters:
    - name: DataCenter
      clusters:
      - { paas: {resource_pool: paas }}
- name: z2
  cloud_properties:
    datacenters:
    - name: DataCenter
      clusters:
      - { paas: {resource_pool: paas }}
- name: z3
  cloud_properties:
    datacenters:
    - name: DataCenter
      clusters:
      - { paas: {resource_pool: paas }}
networks:
- name: default
  subnets:
  - azs: [z1, z2, z3]
    gateway: 192.168.7.254
    dns: [192.168.7.1]
    range: 192.168.7.0/24
    reserved:
      - 192.168.7.1
      - 192.168.7.2
      - 192.168.7.3
      - 192.168.7.4
      - 192.168.7.5
      - 192.168.7.6
      - 192.168.7.7
      - 192.168.7.8
      - 192.168.7.9
      - 192.168.7.10
      - 192.168.7.254 
      - 192.168.7.255
    static:
      - 192.168.7.11 - 192.168.7.40
    cloud_properties:
      name: NetPaaS
vm_types:
- name: minimal
  cloud_properties:
    cpu: 1
    ram: 4096
    disk: 10240
- name: default
  cloud_properties:
    cpu: 2
    ram: 4096
    disk: 10240
- name: small
  cloud_properties:
    cpu: 2
    ram: 8192
    disk: 10240
- name: small-highmem
  cloud_properties:
    cpu: 4
    ram: 32768
    disk: 10240

disk_types:
- disk_size: 5120
  name: 5GB
- disk_size: 10240
  name: 10GB
- disk_size: 100240
  name: 100GB

vm_extensions:
- name: load_balancer_for_mysql_proxy
- name: cf-router-network-properties
- name: cf-tcp-router-network-properties
- name: diego-ssh-proxy-network-properties
- name: 50GB_ephemeral_disk
  cloud_properties:
    disk: 51200
- name: 100GB_ephemeral_disk
  cloud_properties:
    disk: 102400

compilation:
  workers: 5
  reuse_compilation_vms: true
  az: z1
  vm_type: small-highmem
  network: default
```

&emsp;&emsp;随后，我们通过下述命令来更新bosh的cloud-config配置

```
$ bosh -e bosh update-cloud-config cloud-config.yml
```

&emsp;&emsp;至此，我们已经顺利完成Bosh的安装。

### 2.3、CloudFoundry部署

&emsp;&emsp;经过上述的步骤，部署所需的基本环境已经安装完成，下面就可以开始使用cf-deployment来部署CloudFoundry平台。

> 注意：在部署之前，需要完成Bosh director的登录，以及cloud-config的配置。

1. 获取cf-deployment部署代码

   如部署`Bosh director`一样，我们也是通过获取`cf-deployment`来通过其部署CloudFoundry，操作如下：

   ```shell
   $ git clone https://github.com/cloudfoundry/cf-deployment.git
   ```

2. 上传stemcell代码

&emsp;&emsp;基于bosh安装的软件，都需要指定运行其的`stemcell`。`stemcell`就是bosh中干细胞，也就是代表运行的基本系统，各平台的`stemcell`我们可以到[官方列表](http://bosh.io/stemcells)中查看。下面是本次部署的操作步骤：

```shell
bosh -e bosh upload-stemcell https://s3.amazonaws.com/bosh-core-stemcells/vsphere/bosh-stemcell-3541.10-vsphere-esxi-ubuntu-trusty-go_agent.tgz
```

3. 部署CloudFoundry

   &emsp;&emsp;经过前面的铺垫，我们就可以正式开始部署CloudFoundry了，操作如下:

   ```shell
   #设置CloudFoundry所使用的域名，在部署时会自动生成对应的证书
   $ export SYSTEM_DOMAIN=paas.example.com
   $ bosh -e bosh -d cf deploy cf-deployment/cf-deployment.yml \
     --vars-store env-repo/deployment-vars.yml \
     -v system_domain=$SYSTEM_DOMAIN
   ```

   &emsp;&emsp;在安装的过程中，需要网络下载所需的依赖，必须保证网络的可用性。

   &emsp;&emsp;上述的部署默认为有3个集群可用`[z1,z2,z3]`，如果我们的cloud-config配置中，只有一个可用的集群，我们可以在创建的命令中加上`-o operations/scale-to-one-az.yml`参数。

   &emsp;&emsp;通过上面的操作，使用cf-deployment部署CloudFoundry就已经成功，我们可以通过`bosh -e bosh -d cf instances`这个命令来查看部署CloudFoundry所生产的虚拟机及其基本信息。

4. 安装CloudFoundry命令行(cf-cli)

&emsp;&emsp;[cf-cli](https://github.com/cloudfoundry/cli)是一个CloudFoundry官方推出的命令行操作工具，我们可以通过此工具来管理与使用平台的各项功能，每个命令的详细功能说明，我们可以到[命令行操作说明](http://cli.cloudfoundry.org/en-US/cf/)中查看。下面是具体的操作步骤

```shell
# 获取CloudFoundry api地址
$ bosh -e bosh -d cf instances
# 在输出的信息中，我们找到api对应的虚拟机的ip
# 若有多个，只需其中的一个即可，如本次为:192.168.7.30
# 设置域名解析,需要在电脑的hosts文件中，输入以下内容:
# 192.168.7.30 api.paas.example.com
# 192.168.7.30 login.paas.example.com
# 192.168.7.30 uaa.paas.example.com
$ vim /etc/hosts
# 安装命令行
$ sudo wget -O /etc/yum.repos.d/cloudfoundry-cli.repo https://packages.cloudfoundry.org/fedora/cloudfoundry-cli.repo
$ sudo yum install cf-cli
```

通过上述步骤，我们就设置好了域名的解析与安装好了`cf-cli`。

5. 检验环境

&emsp;&emsp;当前面的操作都完成后，我们就可以使用cf-cli来登录到CloudFoundry环境当中,其中admin的密码我们可以根据部署时的参数`--vars-store env-repo/deployment-vars.yml`中指定的`deployment-vars.yml`中查看。

```
$ cf l --skip-ssl-validation -a https://api.paas.example.com -u admin -p password
```

&emsp;&emsp; 通过上述命令，我们就可以连接到CloudFoundry环境当中，后面的话，可以通过发布应用的方式来检验平台的功能与可用性。

&emsp;&emsp;至此，我们通过上述的步骤，成功的使用cf-deployment部署CloudFoundry平台。