title: MongoDb使用
date: 2017/6/13 20:46:25
---

## Mongodb 安装

本次安装的Mongodb类型为MongoDB Community Edition on Linux 的ubuntu（16.04）版本，其他版本的安装方式可以参照官网地址 https://docs.mongodb.com/manual/administration/install-on-linux/ 。


1. 在实际中，我们可以根据需要，安装Mongodb不同的package包，以下是官方提供的几种package包：
   * mongodb-org
   * mongodb-org-server
   * mongodb-org-mongos
   * mongodb-org-shell
   * mongodb-org-tools
<!--more-->
2. 安装的过程和官网介绍的是一样的，就是以下几个步骤

   * 导入公钥 

     ```shell
     sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
     ```

   * 创建list文件

     ```shell
     echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
     ```

   * 更新源

     ```shell
     sudo apt-get update -y
     ```

   * 安装mongodb

     ```shell
     sudo apt-get install -y mongodb-org
     ```


   自此，Mongodb已经成功的安装在ubuntu 16.04上。



## Mongodb的启动和运行







## Mongodb基本操作 

下面记录一些mongodb的基本命令，mongodb服务没有开启用户认证，在真实的生产环境中开启认证后，有部分命令是需要有相关权限才能使用。

### 1. 显示所有数据库信息

```shell
> show dbs
admin  0.000GB
local  0.000GB
```

### 2. 创建和选择数据库

   在mongodb中，创建和选择数据库都是相同的命令 use databasename

   ```shell
   > use testdb
   switched to db testdb

   ```

   当 testdb数据库不存在时，会自动创建testdb数据库，但是，如果在新建的数据库中不存储信息，在后续若使用` show dbs` 命令时，新建的数据库是不会显示的

### 3. 删除数据库

   首先，切换到需要删除的数据库，随后执行删除数据库操作

   ```shell
   > use testdb
   switched to db testdb
   > db.dropDatabase()
   { "dropped" : "testdb", "ok" : 1 }
   ```


### 4. 插入文档

   mongodb插入文档的格式为`db.COLLECTION_NAME.insert(document)`

   其中COLLECTION_NAME为插入的集合名，document为插入的数据

   ```shell
   > db.testcol.insert({name:'tian'})
   WriteResult({ "nInserted" : 1 })
   > show tables
   testcol
   > db.testcol.insert({name:'tian',age:12})
   WriteResult({ "nInserted" : 1 })
   > db.testcol.find()
   { "_id" : ObjectId("58cb9f6219907121a6e70b22"), "name" : "tian" }
   { "_id" : ObjectId("58cba03c19907121a6e70b23"), "name" : "tian", "age" : 12 }
   ```

   在上面的示例中，就可以看到mongodb非常灵活，我们可以根据业务的需要，添加字段，而不必如关系行数据中一样，一开始就限制了表的格式，修改比较繁琐

### 5. 查看文档

   可以使用`db.COLLECTION_NAME.find()`查看当前集合中的所有文档信息

   ```shell
   > db.testcol.find()
   { "_id" : ObjectId("58cb9f6219907121a6e70b22"), "name" : "tian" }
   { "_id" : ObjectId("58cba03c19907121a6e70b23"), "name" : "tian", "age" : 12 }
   ```

   可以加多`.pretty()`，来格式化输出信息

   ```shell
   > db.testcol.find().pretty()
   { "_id" : ObjectId("58cb9f6219907121a6e70b22"), "name" : "tian" }
   {
           "_id" : ObjectId("58cba03c19907121a6e70b23"),
           "name" : "tian",
           "age" : 12
   }

   ```

   使用`findOne()`来获取一条信息，默认返回最早的一条记录

   ```shell
   > db.testcol.findOne()
   { "_id" : ObjectId("58cb9f6219907121a6e70b22"), "name" : "tian" }
   ```

   查询参数详解：

| 等于    | `{<key>:<value>`}        | `db.col.find({"title":"tian"}).pretty()` |
| ----- | ------------------------ | ---------------------------------------- |
| 小于    | `{<key>:{$lt:<value>}}`  | `db.col.find({"likes":{$lt:50}}).pretty()` |
| 小于或等于 | `{<key>:{$lte:<value>}}` | `db.col.find({"likes":{$lte:50}}).pretty()` |
| 大于    | `{<key>:{$gt:<value>}}`  | `db.col.find({"likes":{$gt:50}}).pretty()` |
| 大于或等于 | `{<key>:{$gte:<value>}}` | `db.col.find({"likes":{$gte:50}}).pretty()` |
| 不等于   | `{<key>:{$ne:<value>}}`  | `db.col.find({"likes":{$ne:50}}).pretty()` |

使用例子如下

```shell
> db.testcol.find({'age':{$gt:18}})
{ "_id" : ObjectId("58cba1f519907121a6e70b25"), "name" : "chen", "age" : 19 }
```

可以传入多个值判断，即实现and效果

```shell
> db.testcol.find({'age':{$gt:18},'name':'chen'})
{ "_id" : ObjectId("58cba1f519907121a6e70b25"), "name" : "chen", "age" : 19 }
```

可以使用$or 来实现or效果

```shell
> db.testcol.find({$or:[{'name':'tian'},{'age':15}]})
{ "_id" : ObjectId("58cb9f6219907121a6e70b22"), "name" : "tian" }
{ "_id" : ObjectId("58cba03c19907121a6e70b23"), "name" : "tian", "age" : 12 }
{ "_id" : ObjectId("58cba1e619907121a6e70b24"), "name" : "liu", "age" : 15 }
```

实现and和or一起的效果

```shell
> db.testcol.find({$or:[{'name':'tian'},{'age':19}],title:'mongodb'})
{ "_id" : ObjectId("58cba9b919907121a6e70b26"), "name" : "chen", "age" : 19, "title" : "mongodb" }

```

### 6. 更新文档

Mongodb 可以使用update()和save()方法来更新存在数据库中的数据，一般的格式为:

```shell
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

- **query **: update的查询条件，类似sql update查询内where后面的。

- **update **: update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的

- **upsert **: 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。

- **multi **: 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。

- **writeConcern **:可选，抛出异常的级别。


下面举一些例子

1. 更新一条记录(更新查找到的第一条记录)

   ```shell
   > db.testcol.update({'name':'liu'},{$set:{'age':22}})
   WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
   ```

2. 如果查询的属性返回值是多条记录，而且需要都修改，则可以将multi设为true

   ```shell
   > db.testcol.update({'name':'tian'},{$set:{'age':23}},{multi:true})
   WriteResult({ "nMatched" : 2, "nUpserted" : 0, "nModified" : 2 })
   ```


##Mongodb 权限设置

MongoDb有下面几种权限

| 权限                   | 效果                                       |
| -------------------- | ---------------------------------------- |
| Read                 | 允许用户读取指定数据库                              |
| readWrite            | 允许用户读写指定数据库                              |
| dbAdmin              | 允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile |
| userAdmin            | 允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户 |
| clusterAdmin         | 只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。     |
| readAnyDatabase      | 只在admin数据库中可用，赋予用户所有数据库的读权限              |
| readWriteAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的读写权限             |
| userAdminAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的userAdmin权限      |
| dbAdminAnyDatabase   | 只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。       |
| root                 | 只在admin数据库中可用。超级账号，超级权限                  |
|                      |                                          |

### 创建用户

根据上述权限表，只有拥有创建用户的权限，才能新创建用户。

**注意:**为指定数据库创建用户时，必须先切换到对应的数据库，再创建用户！！！！

例如下面的例子，首先，切换到test数据库，随后，再test数据库中创建数据库的用户admin，下面的例子只是一个实例，权限是可以一次赋予一个或多个的。

```json
use test
db.createUser(  
  {  
    user: "admin",  
    pwd: "admin",  
    roles: [ { role: "readWrite", db: "admin" },{ role: "dbAdmin" ,db: "admin" } ] 
  }  
)

```

