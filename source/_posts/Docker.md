title: Docker一般操作
date: 2017/6/13 20:46:25
---

### 1. 拉取镜像

默认格式为:`docker pull name[:tag]`

Example:

```Shell
docker pull ubuntu
docker pull ubuntu:14.04
```

实际上的操作是从默认docker hub仓库下载镜像，即上面的命令等价于下面的命令

```shell
docker pull registry.hub.docker.com/ubuntu:latest
```



### 2.查看本地镜像信息

1. 获取本地所有的镜像信息

   ```shell
   docker images
   ```

2. 获取指定镜像的具体信息

   ```
   docker inspect image_id
   ```

   获取具体信息中的具体某一项信息

   ```
   docker inspect -f {{".Architecture"}} image_id
   ```



### 3. 搜索Docker镜像

```
docker search image_name
```

可以支持参数，来指定星级等,如下

- `-s,--starts=0`指定仅显示评价为指定星级以上的镜像
- `--no-trunc=false`输出信息不截断显示
- `--automated=false`仅显示自动创建的镜像



### 4.删除Docker镜像

```
docker rmi image_id
```

可以使用`-f`来强制删除镜像

```
docker rmi -f image_id
```

### 5.创建Docker镜像

创建Docker的方式有三种，分别如下：

- 基于已有镜像的容器创建
- 基于本地模版导入
- 基于Dockerfile创建

#### 5.1 基于已有镜像的容器创建

首先，拉取一下镜像并运行

```
docker run -ti ubuntu /bin/bash
```

运行上面的命令后，会打开创建容器的bash窗口，随后，修改镜像里面的内容

```
touch test
exit
```

exit后，就会退出镜像，退出后，容器就停止运行了

随后创建镜像

```
docker commit -m "add a file" -a "geekyzk" abcd geekyzk/ubuntu
```

- -m 镜像说明


- -a 作者
-  abcd 为容器的id
- geekyzk/ubuntu为镜像名

#### 5.2 基于本地模版导入



### 6.容器启动和停止

```
docker run -t -i ubuntu /bin/bash
```

- `-t`表示让docker分配一个伪终端并绑定到容器的标准输入上
- `-i`表示让容器的标准输入保持打开

#### 6.1 守护态运行

```
docker run -d ubuntu /bin/bash
```

容器启动后，返回一个id，并在后台运行程序，不会关闭

#### 6.2 终止／停止容器

```
docker stop image_id
```

#### 6.3 启动容器

处于停止的容器，可以通过以下命令来启动

```
docker start image_id
```

 #### 6.4 重启容器

```
docker restart image_id
```



### 7.获取容器输出

```
docker logs image_id
```

### 8. 查看容器

查看正在运行的容器

```
docker ps
```

查看所有容器，包括已停止的

```
docker ps -a
```

### 9.进入容器中

#### 9.1 通过attach进入

```
docker attach image_id
```

但是进入后退出容器，容器也会随着停止

#### 9.2 通过exec命令进入

```
docker exec -ti image_Id /bin/bash
```



### 10.挂载卷

```
docker run -d -P --name web -v /webapp:/opt/webapp training/webpapp python app.py
```

- -v 指定挂载的文件，宿主机:容器,可以多个-v共同使用



### 11.数据卷容器共享数据

多个容器中共享数据，最好的方法是使用数据卷容器，数据卷容器就是一个普通的容器，专门用它提供数据卷供其他容器挂载

```
docker run -it -v /dbdata --name dbdata ubuntu
```

```
docker run -it --volumes-from dbdata --name db1 ubuntu
```



### 12.端口映射

```
docker run -itd -P tomcat 
```

- -P 会随机映射一个端口对应容器内开放的网络端口

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
e5a35854de91        fa6a                "catalina.sh run"   3 minutes ago       Up 3 minutes        0.0.0.0:32768->8080/tcp   tomcat
```



也可以使用`-p`来指定映射的端口

```
docker run -itd -p 5000:5000 tomcat
```

- -p支持的格式：
  - Ip:hostPort:containerPort
  - Ip::containerPort 绑定本地的任意端口映射容器的端口
  - hostPort:containerPort

#### 12.1 查看端口配置

```
docker port image_id container_port
```



### 13.容器互联

```
docker run -d --name db postgres
```

```
docker run  -d -P --name web --link db:db tomcat 
```

这样，web容器和db容器之间就能建立连接了

- `--link` 格式为`name:alias`其中name是要连接的容器名，alias是这个连接的别名

随后，可以查看到web容器中，就已经配置有相关db容器的环境变量

Example:

`docker run -d -P --link tomcat:tomcat ubuntu /bin/bash`

```json
TOMCAT_PORT=tcp://172.17.0.2:8080
TOMCAT_PORT_8080_TCP=tcp://172.17.0.2:8080
TOMCAT_PORT_8080_TCP_ADDR=172.17.0.2
TOMCAT_PORT_8080_TCP_PORT=8080
TOMCAT_PORT_8080_TCP_PROTO=tcp
TOMCAT_NAME=/boring_engelbart/tomcat
TOMCAT_ENV_LANG=C.UTF-8
TOMCAT_ENV_JAVA_HOME=/docker-java-home/jre
TOMCAT_ENV_JAVA_VERSION=7u131
TOMCAT_ENV_JAVA_DEBIAN_VERSION=7u131-2.6.9-2~deb8u1
TOMCAT_ENV_CATALINA_HOME=/usr/local/tomcat
TOMCAT_ENV_TOMCAT_NATIVE_LIBDIR=/usr/local/tomcat/native-jni-lib
TOMCAT_ENV_LD_LIBRARY_PATH=/usr/local/tomcat/native-jni-lib
TOMCAT_ENV_OPENSSL_VERSION=1.1.0f-3
TOMCAT_ENV_GPG_KEYS=05AB33110949707C93A279E3D3EFE6B686867BA6 07E48665A34DCAFAE522E5E6266191C37C037D42 47309207D818FFD8DCD3F83F1931D684307A10A5 541FBE7D8F78B25E055DDEE13C370389288584E7 61B832AC2F1C5A90F0F9B00A1C506407564C17A3 713DA88BE50911535FE716F5208B0AB1D63011C7 79F7026C690BAA50B92CD8B66A3AD3F4F22C4FED 9BA44C2621385CB966EBA586F72C284D731FABEE A27677289986DB50844682F8ACB77FC2E86E29AC A9C5DF4D22E99998D9875A5110C01C5A2F6059E7 DCFD35E0BF8CA7344752DE8B6FB21E8933C60243 F3A04C595DB5B6A5F1ECA43E3B7BBB100D811BBE F7DA48BB64BCB84ECBA7EE6935CD23C10D498E23
TOMCAT_ENV_TOMCAT_MAJOR=8
TOMCAT_ENV_TOMCAT_VERSION=8.0.45
TOMCAT_ENV_TOMCAT_TGZ_URL=https://www.apache.org/dyn/closer.cgi?action=download&filename=tomcat/tomcat-8/v8.0.45/bin/apache-tomcat-8.0.45.tar.gz
TOMCAT_ENV_TOMCAT_ASC_URL=https://www.apache.org/dist/tomcat/tomcat-8/v8.0.45/bin/apache-tomcat-8.0.45.tar.gz.asc
TOMCAT_ENV_TOMCAT_TGZ_FALLBACK_URL=https://archive.apache.org/dist/tomcat/tomcat-8/v8.0.45/bin/apache-tomcat-8.0.45.tar.gz
TOMCAT_ENV_TOMCAT_ASC_FALLBACK_URL=https://archive.apache.org/dist/tomcat/tomcat-8/v8.0.45/bin/apache-tomcat-8.0.45.tar.gz.asc
```





### 创建标签

```
docker tag registry.em248.com:5000/ubuntu:latest ubuntu:latest
```

> 注意：新创建的标签的id会跟源镜像的id一致。如果使用docker rmi 删除创建的标签镜像，不会删除镜像，而是只会删除这个创建的标签

