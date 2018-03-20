title: ServiceBroker 学习
date: 2017/6/13 20:46:25
---

## 管理ServiceBroker

### 管理前提

必须满足以下条件之一：

- 必须是cloudfoundry平台的admin用户

- 必须是空间的developer


### 注册Broker

​	注册broker的原因是因为cloudfoundry会取broker的数据，并且验证数据设置的正确性，在验证成功后，会保存broker的catalog到cloudfoundry的数据库中。在创建成为broker时，需要设置访问broker的用户名和密码，broker程序也应该做相应的用户验证，防止其他用户恶意删除相关数据。如果注册时使用的url使用了https，cloudfoundry controller也会相应的使用https请求

<!--more-->
**注意**： cf-release 229, CC API 2.47.0, cloudfoundry支持两种类型的broker，即标准的broker（全局作用域）

和空间作用域的broker。

#### 注册标准Broker(standard Brokers)

- 需要由admin来创建和管理

- 可以发布到特定的组织或者所有组织

- 可以保持plan不可用或私有

- 刚创建的时候，默认为私有，如果想使用，必须使其明确指出是哪个空间可用，或者所有空间都可以用

创建命令使用如下:

```shell
$ cf create-service-broker mybrokername someuser somethingsecure https://mybroker.example.com/
```

#### 空间作用域Broker(**Space-Scoped Brokers**)

- 只对空间内的用户有效，空间外的用户是不可用的

- 必须要由空间的开发者用户创建，即必须要有developer权限

- 由空间的开发者进行管理

- broker一经发布，在所发布的空间内会自动变成可用。

创建命令例子如下:

```shell
$ cf create-service-broker mybrokername someuser somethingsecure https://mybroker.example.com/ --space-scoped
```

**注意**:空间开发者创建broker，必须加上--space-scoped参数，如果没加上，在创建的时候是会收到错误信息的。

### Broker 访问控制(Access Control)

​	所有标准broker刚发布的时候，默认的访问权限是私有的，那就是意味着如果我们修改broker相关的信息，例如添加新broker或者是添加新plan在已存在的catalog信息中，用户是不能立即获取最新的数据的，因此，必须使用admin用户来管理broker的访问，以展现给用户最新的消息。

​	空间作用域的broker一发布，默认已经是自动运行空间内所有用户使用，因此已经没有集成访问控制。

#### 使用前提

- 必须拥有admin管理员用户

- CLI v6.4.0

- Cloud Controller API v2.9.0 (cf-release v179)

#### 显示所有访问权限信息

可以使用`cf service-access`命令来获取所有访问权限信息，如下：

```shell
$ cf service-access
getting service access as admin...
broker: p-riakcs
   service    plan        access    orgs
   p-riakcs   developer   limited

broker: p-mysql
   service   plan        access   orgs
   p-mysql   100mb-dev   all
```

access属性有三个值，三个值的定义分别如下：

- **all** : 服务对所有用户来说是可用的
- **none** : 服务对所有用户来说是不可用的
- **limited** : 服务仅对在列出的组织下的用户可用

大致使用方式如下:
```shell
$ cf help service-access
NAME:
   service-access - List service access settings

USAGE:
   cf service-access [-b BROKER] [-e SERVICE] [-o ORG]

OPTIONS:
   -b     access for plans of a particular broker
   -e     access for plans of a particular service offering
   -o     plans accessible by a particular org
```
#### 设置broker允许使用

可以使用`cf enable-service-access SERVICE-NAME`命令来使broker变成可用，如果没有任何参数，即默认设置成对所有用户都可用。

大致参数如下:

- -p Plan:授权所有用户只能访问service指定的plan
- -o Org:授权指定的组织下的用户能使用service的所有plan
- -p PLAN -o ORG：授权特定组织下的用户只能访问service指定的plan

也可以使用`cf help enable-service-access`来获取更多帮助信息

#### 设置broker禁止使用

使用的命令为`cf disable-service-access SERVICE-NAME`，其大致用法也是如`enable-service-access`一样的，可以参考enable-service-access的使用。

#### 注意

如果当前broker是允许所有用户访问的，则不能再使用`disable-service-access`来指定组织不可用。应该先设置broker为所有组织都不可用，再设置可访问的组织。



### 多个Brokers, Services, Plans注意事项

- 不允许broker有相同的名字

- 不允许broker有相同的url

- service id和plan id必须是唯一的，建议使用guid

### 列出所有brokers

可以使用如下命令来获取所有broker

```shell
$ cf service-brokers
Getting service brokers as admin...Cloud Controller
OK

Name            URL
my-service-name https://mybroker.example.com
```

### 更新指定broker信息

​	由于cloudfoundry不能自动时时更新broker的catalog信息，如果我们更新了broker的catalog信息，则应该使用更新命令，来让cloudfoundry重新验证和更新在其数据库中的数据。

​	更新命令也提供了改变broker认证用户名和密码的信息。

​	使用例子如下:

```shell
$ cf update-service-broker mybrokername someuser somethingsecure https://mybroker.example.com/

```

### 重命名broker

可以使用如下命令来更改broker的名字

```shell
$ cf rename-service-broker mybrokername mynewbrokername
```

**注意**：重命名只是改变其在cloudfoundry数据库的标识，即创建broker使指定的名字，对broker内的catalog信息是不会有任何改变的。

### 删除broker

删除broker会删除所有services和plan，使用如下：

```shell
$ cf delete-service-broker mybrokername
```

**注意**：如果在cloudfoundry中，还有用此broker信息创建出来的服务实例，是会删除失败的，当计划删除broker时，应该第一时间删除其所有实例。如果broker已经停止使用，但是没有删除实例，也可以使用purge相关的命令来清理。

### 清理service

如果broker已经停止使用，但是没有删除实例，则我们应该使用以下命令来删除相关的plan和实例信息，来清理cloudfoundry的数据库。

这个操作不会发送任何请求到broker中，只是清楚cloudfoundry的数据库中的信息。

当所有service的信息被清理后，我们就可以删除相关的broker了，例子如下:

```shell
cf purge-service-offering v1-test 
```

### 清理 Service Instances

可以使用如下命令来清理，机制跟清理service是一样的。

```shell
cf purge-service-instance mysql-dev
```

