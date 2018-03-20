title: 构建自定义BoshRelease
date: 2017/7/13 20:46:25
---
一个Release中，可以只运行一个软件，也可以同时运行多个软件。例如，可以创建一个Release，其中运行3个部分：两个Mysql节点与一个Web应用。

一个标准的Release，包含有以下几个基本的部分：

- Jobs：包含了一个Release中的软件或服务
- Packages：包含了Job的源码或者依赖
- Sources：提供非二进制包给Packages
- Blob：提供二进制文件（除了签入源代码存储库的那些文件）给Packages

下面我们来讲解怎么定制化创建一个Release，一共分为6个步骤。

## 环境准备

在本地开发,我们可以通过官网的教程安装好最新的bosh cli，然后使用virtualbox，来装好一个本地的Bosh-lite环境。

安装好后，可以如下测试，显示当前版本

```
➜  ~ bosh -v
version 2.0.48-e94aeeb-2018-01-09T23:08:08Z

Succeeded
```

随后，我们通过命令`bosh init-release --dir <release_name>`创建一个基本的工程骨架,在创建的时候，可以添加`--git`参数，这样在初始化工程时，也会同时初始化git仓库。

当初始化好后，我们可以有如下的结构：

```
➜ tree .
.
├── config
│   ├── blobs.yml
│   └── final.yml
├── jobs
├── packages
└── src

4 directories, 2 files
```

当我们发布Release时，Bosh将会编译代码或其他资源到相关的虚拟机系统中的`/var/vcap`目录下，例如：上面的创建的目录，当发布后，将会是以下路径：

`/var/vcap/jobs`、`/var/vcap/packages`等

<!--more-->


## 一、初始化job

bosh提供了初始化job的命令`bosh generate-job job_name`,我们可以使用这个命令来初始化job文件结构，如下所示:

```
$bosh generate-job bg_worker
$tree .
.
├── config
│   ├── blobs.yml
│   └── final.yml
├── jobs
│   ├── bg_worker
│   │   ├── monit
│   │   ├── spec
│   │   └── templates
```

如上所示，初始化命令后再jobs文件夹下创建出**monit**，**spec**两个文件与**templates**文件夹，这三个文件也就是job结构所依赖的。

### 1.1、创建控制脚本(Controller scripts)

对于每个job来说，都必须拥有启动和停止的方法，因此，我们就必须编写提供实现这些功能的控制脚本。

> 注意：
>
> - 脚本必须包含start、stop命令
> - 编写的脚本放在对应job文件夹下的templates目录下，以erb结尾，一般命名为ctl.erb
> - 对于脚本的日志输出，应该在脚本中定义输出的路径为`/var/vcap/sys/log/JOB_NAMEu`
> - 控制脚本可以是多个，而不是只能只有一个

以下是官方的一个实例：

```
#!/bin/bash

RUN_DIR=/var/vcap/sys/run/web_ui
LOG_DIR=/var/vcap/sys/log/web_ui
PIDFILE=${RUN_DIR}/pid

case $1 in

  start)
    mkdir -p $RUN_DIR $LOG_DIR
    chown -R vcap:vcap $RUN_DIR $LOG_DIR

    echo $$ > $PIDFILE

    cd /var/vcap/packages/ardo_app

    export PATH=/var/vcap/packages/ruby_1.9.3/bin:$PATH

    exec /var/vcap/packages/ruby_1.9.3/bin/bundle exec \
      rackup -p <%= properties.web_ui.port %> \
      >>  $LOG_DIR/web_ui.stdout.log \
      2>> $LOG_DIR/web_ui.stderr.log

    ;;

  stop)
    kill -9 `cat $PIDFILE`
    rm -f $PIDFILE

    ;;

  *)
    echo "Usage: ctl {start|stop}" ;;

esac
```

### 1.2、更新monit文件

当创建完控制脚本后，我们就需要更新对应job的文件夹下的monit文件。

**monit**文件主要包含以下内容

- 说明Job的进程ID(PID)
- 引用模板文件里提供的每一个命令
- 指定job为**vcap**组

在部署的Release中，bosh会运行代理(Agent)在每个运行job的虚拟机中，而bosh通过与每个代理(Agent)进行沟通进而执行脚本中控制脚本的命令。

Bosh的代理是使用开源的监控软件Monit。

以下为官方的一个示例

```
check process web_ui
  with pidfile /var/vcap/sys/run/web_ui/pid
  start program "/var/vcap/jobs/web_ui/bin/ctl start"
  stop program "/var/vcap/jobs/web_ui/bin/ctl stop"
  group vcap
```

> 注意：对于每个job，**monit**这个文件是必须的。

### 1.3、更新spec文件

在编译阶段，BOSH将templates中的模板转换成文件，然后将它复制到作业虚拟机上。

因此，我们必须在对应job文件夹下的specq文件中的templates区域中编写相关的数据，以键值对的方式书写，规则如下：

- Key为模板的名字，机templates文件夹下的模板文件名
- value为job对应的虚拟机上的路径

以下为一个例子：

```
templates:
  ctl.erb: bin/ctl
```

对于每个job而言，其在生成的虚拟机的路径都为`/var/vcap/jobs/<jobs_name>`，因此，上面的例子中**bin/ctl**的实际路径则为`/var/vcap/jobs/<jobs_name>/bin/ctl`

> 一般约定，把运行文件放到job的bin目录下

## 二、构造依赖

对于BOSH release中的依赖，有两种类型：

- 运行时依赖。当程序运行时所依赖的package，例如当运行web_ui时，这个任务依赖Ruby
- 编译时依赖。当前package在编译的时候，依赖其他package，例如Ruby依赖YAML library

在我们设计依赖时，应该遵循以下三个原则:

- job不能依赖其他job
- job可以依赖package
- Package可以依赖其他pakage




## 三、创建Package骨架

在Release中，Package为Job提供所需的二进制包已经其他的依赖信息。

我们可以通过`bosh generate-package <dependency_name>`命令来为对应的依赖来分别创建对应的依赖描述

如官方例子所示：依赖`libyaml_0.1.4`, `ruby_1.9.3`和`ardo_app`

```
➜ tree packages
packages
├── ardo_app
│   ├── packaging
│   └── spec
├── libyaml_0.1.4
│   ├── packaging
│   └── spec
└── ruby_1.9.3
    ├── packaging
    └── spec
```

>将每个依赖关系放在一个单独的包中提供了最大的可重用性以及清晰的模块化结构,但是不是必须的


### 3.1、更新packaging与specs文件

在每个package文件夹中，都包含一个spec文件，这个文件主要描述以下几点

- package名称
- package的依赖
- 编译时，所需的二进制文件或其他包的本地路径

我们可以根据第二步构建的依赖图来去顶哪些依赖属于哪个对应的spec。

> 我们应该尽量不依赖底层系统的package，这样能够为我们的Release提供最大的可用性

我们在spec文件中，应该在files内容块中描述二进制文件的路径，步骤如下：

- 查找二进制软件包的下载地址，如Ruby为：`http://cache.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p484.tar.gz`.
- 下载二进制包，并验证二进制包的Hash值是否一样。
- 记录包含版本号的二进制名称，并用斜杠和二进制文件名连接起来。在同一行中引用官方网址是一个好主意。

Bosh会根据spec文件配置的文件路径在`src`或`blobs`文件夹中查找对应的二进制包（**首先查看的是`src`文件夹**）。当我们添加实际的二进制包到Bosh的Blobstore中时，Bosh将会把正确的信息放到`blobs`文件夹中。

对于依赖自身源码的的packages，可以使用通配符`<package_name>/**/*`来对其源码进行一个深度的遍历。下面提供几个例子：

```
---
name: libyaml_0.1.4

dependencies: []

files:
- libyaml_0.1.4/yaml-0.1.4.tar.gz # From http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz
```

```
---
name: ruby_1.9.3

dependencies:
- libyaml_0.1.4

files:
- ruby_1.9.3/ruby-1.9.3-p484.tar.gz # http://cache.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p484.tar.gz
- ruby_1.9.3/rubygems-1.8.24.tgz    # http://production.cf.rubygems.org/rubygems/rubygems-1.8.24.tgz
- ruby_1.9.3/bundler-1.2.1.gem      # https://rubygems.org/downloads/bundler-1.2.1.gem
```

```
---
name: ardo_app

dependencies:
- ruby_1.9.3

files:
- ardo_app/**/*
```

### 3.2、创建Packaging脚本

在编译阶段, Bosh将会应用package中spec定义文件, 并将它们呈现为部署作业所需的可执行二进制文件和脚本。

我们需要写脚本告诉Bosh应该怎么做这些操作，说明可能涉及复制，编辑和相关程序的某些组合，例如：

- 对于类似官网提供的Ruby应用`ardo_app`，Bosh必须复制对应的源码文件和安装Ruby gems。
- 对于Ruby自身。Bosh必须编译源码成二进制文件。
- 对于Python应用。Bosh必须复制对应的源码文件和安装Python等。

Bosh一切都根据我们书写的packaging script。编写packaging script时应遵守下面的原则：

- 使用您的依赖关系图确定哪些依赖关系属于哪个Packaging脚本。
- 在每个脚本开头添加`set -e -x`这一行，这有助于在编译时调试脚本，如果命令以非零退出代码退出，则立即退出脚本
- 确保任何复制，安装或编译都将结果代码传送到安装目标目录（表示为BOSH_INSTALL_TARGET环境变量）。对于make命令，使用configure或其等价物来完成此操作
- 注意，BOSH确保在部署时应确保`spec`文件中的`dependencies`区域中的依赖时可用的。例如，在Ruby包的spec文件中，我们引用libyaml作为依赖项。这确保了在编译VM上，Ruby的打包脚本可以访问编译好的libyaml包。

如果您在打包脚本中提供的指令无法将编译后的代码传递到BOSH_INSTALL_TARGET，他的工作无法运行，因为VM无法找到要运行的代码。

如控制脚本(Control scripts),编写Packaging script是创建release所需要的较重任务之一。下面展示了官方的一些例子：

```
set -e -x

tar xzf libyaml_0.1.4/yaml-0.1.4.tar.gz
pushd yaml-0.1.4
  ./configure --prefix=${BOSH_INSTALL_TARGET}

  make
  make install
popd
```

```
set -e -x

tar xzf ruby_1.9.3/ruby-1.9.3-p484.tar.gz
pushd ruby-1.9.3-p484
  ./configure \
    --prefix=${BOSH_INSTALL_TARGET} \
    --disable-install-doc \
    --with-opt-dir=/var/vcap/packages/libyaml_0.1.4

  make
  make install
popd

tar zxvf ruby_1.9.3/rubygems-1.8.24.tgz
pushd rubygems-1.8.24
  ${BOSH_INSTALL_TARGET}/bin/ruby setup.rb
popd

${BOSH_INSTALL_TARGET}/bin/gem install ruby_1.9.3/bundler-1.2.1.gem --no-ri --no-rdoc
```

```Shell
set -e -x

cp -a ardo_app/* ${BOSH_INSTALL_TARGET}

cd ${BOSH_INSTALL_TARGET}

/var/vcap/packages/ruby_1.9.3/bin/bundle install \
  --local \
  --deployment \
  --without development test
```

### 3.3、根据依赖更新job specs文件

依赖关系图显示需要添加到对应job sepcs文件中packages中的运行时依赖关系。例如：

```Yaml
packages:
- ardo_app
- ruby_1.9.3
```

## 四、Add Blobs

在创建Release时，有可能会使用到源码仓库，如（git repository），但是release通常使用的是tar文件或者其他二进制文件。

通过执行以下操作，BOSH可以避免将二进制文件检入存储库。

- 对于开发版本，可以使用本地二进制文件进行复制
- 对于最终版本，上传二进制文件到blobstore,并告诉Bosh应该到blobstore哪获取

### 4.1、配置blobstore

在config目录中，配置了BOSH需要关于blobstore的信息：

- final.yml包含其文件名与其类型，该类型可以是本地的，也可以是指定blobstore提供程序的其他几种类型之一。
- private.yml文件指定了blobstore路径以及一个秘钥（secret）。

private.yml包含用于访问blobstore的密钥，不应将其检入到存储库中.

config目录还包含两个文件，其内容是自动生成的：blobs.yml文件和dev.yml文件。下面列举了官方的一些配置

##### final.yml:

```
---
blobstore:
  provider: local
  options:
    blobstore_path: /tmp/ardo-blobs
final_name: ardo_app
```