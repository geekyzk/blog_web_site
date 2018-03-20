title: Consul介绍与使用
date: 2017/6/13 20:46:25
---

##一、简介

Consul 是一个基于GO语言开发的支持多数据中心分布式高可用的服务发现和配置共享服务软件，使用了**Raft算法**来保持一致性。 Consul可以分**Client端**与**Server端**，每个数据中心应运行3-5个Server端来保证Consul的高可用，Client端运行在每个node节点中，Client端不持久化数据，Server端负责持久化数据。

Consul拥有以下特性：

- **服务发现(Service Discovery)**：Consul的Client可以提供服务，如Mysql服务或提供RestAPI的WEB应用，其他客户端可用通过服务名来发现与利用提供的服务。Consul支持http协议与dns协议。
- **健康检查(Health Checking)**：Consul可以提供多种方式的健康检查，如检查所提供的服务是否正常、节点内存使用是否超过阈值、节点cpu使用情况是否超过阈值等等。通过健康检查，我们可以很好的监控节点的健康状态以及发现非正常原因。
- **键值对存储(KV Store)**：同其他类似的软件etcd等，Consul也提供键值对存储的功能以达到多种目的。
- **支持多数据中心(Multi Datacenter)**：Consul支持多数据中心，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障,而其部署则需要考虑网络延迟, 分片等情况等. zookeeper 和 etcd 均不提供多数据中心功能的支持

<!--more-->

##二、安装Consul

可以通过官方地址（https://www.consul.io/downloads.html）来下载到对应平台的二进制包。各个平台的运行方式类似，都是把Consul文件加压后的路径放到环境变量中即可。

添加完后，可运行`consul --version`来检查是否生效

```
$consul --version
Consul v1.0.6
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
```



## 三、基本命令使用

