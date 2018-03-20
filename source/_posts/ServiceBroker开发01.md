title: ServiceBroker 开发
date: 2017/6/13 21:46:25
---

#概述	

​	ServiceBroker 是CloudFoundry中一个可插拔式的组件，它归属于CloudFoundry的服务市场（marketplace ），我们可以根据ServiceBroker开发规范自定义开发ServiceBroker注册到平台的服务市场中。根据这个特性，我们可以无限的扩展服务中间件的提供能力，如提供数据库服务，缓存服务，对象存储服务，非关系型数据库等等，下图是CloudFoundry官方文档中对ServiceBroker功能的示意图：

![managed-services.png](http://www.wailian.work/images/2018/02/23/managed-services.png)


<!--more-->

# 用途

​	ServiceBroker最重要的功能就是提供了中间件的服务能力。我们可以通过运行`cf marketplace `来获取得到平台中存在的服务，并通过`cf create-service`与`cf bind-service` 命令来创建服务实例与绑定服务实例到应用中，通过这种绑定服务实例的方式，应用就可以得到对应的中间件服务。

​	例如： 一个Web应用需要使用数据库来存储数据。那我们就可以在**marketplace** 中找到数据库相关的服务，如Mysql，创建Mysql服务的一个服务实例并绑定到对应的应用中，随后通过`cf restage appName`命令重置应用环境，那么在应用启动成功后，所需的数据库服务的信息如用户名密码等，就会写入到应用运行的容器环境变量中，如概述中的图左下角所示，应用可以通过读取环境获取配置的方式来使用数据库服务。

> 对于大多数开发语言与开发框架，如spring一系列框架，运行环境（buildpack）支持自动发现与配置，即能自动根据环境变量替换程序里面的DataSource连接数据库对象，达到无需更改代码即完成应用在云端的发布。

# 开发规范

​	ServiceBroker的本质就是一个实现了CloudFoundry定义接口的应用，提供对应接口的Rest接口。以下是CloudFoundry中规定的api接口列表（接口的详细要求与描述可参看[官方文档]([ServiceBroker API]:https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md) ）：
| 	url  | method  | 作用 |
| ---------- | ---- | :--------: |
| /v2/catalog | GET | 获取服务信息 |
| /v2/service_instances/:instance_id | PUT | 创建服务实例 |
| /v2/service_instances/:instance_id | PATCH | 更新服务实例 |
| /v2/service_instances/:instance_id/service_bindings/:binding_id | PUT | 绑定服务实例 |
| /v2/service_instances/:instance_id/service_bindings/:binding_id | DELETE | 解绑服务实例 |
| /v2/service_instances/:instance_id | DELETE | 删除服务实例 |

以上就是ServiceBroker所需提供的服务接口，我们的应用只要实现相应的Api接口，并且把Broker注册到平台上，CloudFoundry就会把我们的服务在服务市场中展示处理，提供给应用使用。

为了更好的扩展与使用，我们对于自定义的服务实例，也可以定义多以下的接口，方便进行服务的开发与使用

| url                                                   | method | 作用                 |
| ----------------------------------------------------- | ------ | -------------------- |
| /v2/service                                           | PUT    | 创建服务信息         |
| /v2/service/:service_id                               | PATCH  | 更新服务信息         |
| /v2/service/:service_id                               | DELETE | 删除服务信息         |
| /v2/service/:service_id/plans                         | PUT    | 创建服务计划         |
| /v2/service/:service_id/plans/:plan_Id/costs          | PUT    | 创建服务计划收费信息 |
| /v2/service/:service_id/plans/:plan_id/costs/:cost_id | DELETE | 删除服务计划收费信息 |
| /v2/service/:service_id/plans/:plan_id                | PATCH  | 更新服务计划信息     |
| /v2/service/:service_id/plans/:plan_id                | DELETE | 删除服务计划信息     |

ServiceBroker的插件式扩展，使我们能够有极大的操作空间，如能把Docker，k8s，Hadoop等封装成ServiceBroker，来提供对资源的的管控。

Spring团队对于ServiceBroker标准API已经开发出[Spring Cloud Open Service Broker](https://github.com/spring-cloud/spring-cloud-open-service-broker)框架，这套框架对于标准Api进行了封装，我们只需要在这个基础上实现其对应的实现，并扩展我们自定义的API，就能快速开发出一个自定义ServiceBroker。

这里提供github上Spring团队基于[Spring Cloud Open Service Broker](https://github.com/spring-cloud/spring-cloud-open-service-broker)框架实现的[MongoDB ServiceBroker例子](https://github.com/spring-cloud-samples/cloudfoundry-service-broker)，后期也会发布一个独立封装的Mysq lServiceBroker的文章，其中一步步的实现一个比较完善的ServiceBroker功能。



