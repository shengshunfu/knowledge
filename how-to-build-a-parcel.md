# cloudera manager 基本概念

![enter image description here](http://blog.cloudera.com/wp-content/uploads/2013/07/ClouderaManagerVocabulary.png)

# parcel

## 什么是parcel?

parcel是cloudera manager用来在集群中分发软件的一种机制.  它的格式其实就是一个tar格式的的文件, 里面包含了软件的二进制代码, 以及一些元数据. cloudera manager会自动的将它分发到集群中的每一台机器上, 然后根据parcel的元信息设置目录链接, 命令链接等等.

以Cloudera提供的Kafka的parcel为例整个tar的内容结构大致如下:

```PLAIN
KAFKA-2.0.0-1.kafka2.0.0.p0.12
├── bin
│   ├── kafka-acls
│   ├── kafka-configs
│   ├── kafka-connect-distributed
│   ├── kafka-connect-standalone
│   ├── kafka-console-consumer
│   ├── kafka-console-producer
│   ├── kafka-consumer-offset-checker
│   ├── kafka-preferred-replica-election
│   ├── kafka-reassign-partitions
│   ├── kafka-run-class
│   └── kafka-topics
├── etc
│   ├── default
│   │   └── kafka
│   ├── init.d
│   │   ├── kafka-mirror-maker
│   │   └── kafka-server
│   └── kafka
│       └── conf.dist
├── lib
│   └── kafka
│       ├── bin
│       ├── cloudera
│       ├── config -> /etc/kafka/conf
│       ├── libs
│       ├── LICENSE
│       ├── logs
│       ├── NOTICE
│       └── site-docs
└── meta
    ├── alternatives.json
    ├── filelist.json
    ├── kafka_env.sh
    └── parcel.json
```

其中meta这个目录下的文件的定义如下:

**`parcel.json`**

 包含了软件的元信息, 比如说软件的名称, 版本, 提供的组件, 与其他软件的依赖关系等等. **这个文件是必选项**, 其余的都是可选项.

详细介绍参见: https://github.com/cloudera/cm_ext/wiki/The%20parcel.json%20file

**`filelist.json`**

这个parcel中的包含的文件 (除去meta目录), 结果时以JSON的形式展现的.

**`kafka_env.sh`**

用于设置关于这个软件部署时需要设置的环境变量, 也可以与其他以parcel方式打包的软件互操作, 为其注入一些变量. 比如说, 我们可以制作一个phoenix的parcel包, 在它的`parcel.json`指定一些参数, 让HBase启动时会执行我们指定的(比如说)`phoenix_env.sh`文件从而将phoenix 的jar添加到HBase的CLASSPATH中. 这个文件的名字取决于你在`parcel.json`的参数.

详细内容参见: https://github.com/cloudera/cm_ext/wiki/The%20Parcel%20Defines%20Script

**`alternatives.json`**

主要的功能是定义需要注册到系统的一些文件, 可以简单的理解为一些软连接的定义, 例如需要将`bin`目录下的命令软链接到`/usr/bin`下面, 这样方便我们使用这个软件,

详细介绍参见: https://github.com/cloudera/cm_ext/wiki/The%20alternatives.json%20file

 **`permissions.json`**

 定义了软件的二进制文件的用户组以及权限

详细内容参见: https://github.com/cloudera/cm_ext/wiki/The%20permissions.json%20file

## 如何制作parcel

首先我们从下载https://github.com/cloudera/cm_ext/, 这里面提供了一些验证工具, 可以帮助我们验证上面提到的这些定义文件, 以及可以验证最终生成的`.parcel`文件是不是有效的, 简单的步骤如下

```BASH
git clone https://github.com/cloudera/cm_ext/
cd cm_ext
mvn install  # 确保你已经有安装了maven 版本在3以上

cd validator/target
ls
# 在validator/target下面可以看到编译好的validator.jar

# 执行
java -jar validator.jar

# 方便起见我们可以定义一个命令

validate () {
    java -jar $PWD/validator.jar $@
}

# 等价于执行java -jar $PWD/validator.jar
validate
```

我们还要准备一个cloudera manager用来测试我们的parcel

安装方式可以参考: [Cloudera Manager Quick Start Guide](http://www.cloudera.com/documentation/manager/5-1-x/Cloudera-Manager-Quick-Start/Cloudera-Manager-Quick-Start-Guide.html)

详细制作步骤参考: https://github.com/cloudera/cm_ext/wiki/Building-a-parcel

然后将parcel上传的cloudera manager对应的主机上, 默认的目录是`/opt/cloudera/parcel-repo`

记得要为parcel计算一个sha1的校验, 文件名是`parcel文件的全名 + .sha`

然后在cloudera manager的parcel页面 (`http:/<host>:<port>/cmf/parcel/status`)可以看到我们构建好的软件包, 试着分配/激活以下, 看看能不能正常工作.

如果在cloudera manager没有显式我们的软件包, 可以通过日志来检查:

```
sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```
