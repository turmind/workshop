# emr poc

## vpc 和 s3 endpoint

- EMR部署在VPC的同一可用区中，避免产生跨区域访问流量
- VPC中增加S3 endpoint-gateway,增加方法为[s3 endpoint](https://docs.aws.amazon.com/zh_cn/vpc/latest/privatelink/vpc-endpoints-s3.html)，S3的endpoint type有两类，interface及gateway，选择gateway成本上更有优势。添加后，可通过以下第三方参考验证是否添加成功[verify](https://serverfault.com/questions/822979/how-to-verify-a-aws-vpc-s3-endpoint-works)
- 创建EMR时，可选择graviton类型实例以节省成本，graviton为实例类型名称中带有g的类型

## 使用外部存储

- 创建EMR时，选择使用glue存储元数据，可参考[hive metadata to glue](https://docs.aws.amazon.com/zh_cn/emr/latest/ReleaseGuide/emr-hive-metastore-glue.html)及[spark metadata to glue](https://docs.aws.amazon.com/zh_cn/emr/latest/ReleaseGuide/emr-spark-glue.html)，也可以选择使用mysql存储元数据，参考：[外部mysql](https://docs.aws.amazon.com/zh_cn/emr/latest/ReleaseGuide/emr-hive-metastore-external.html)
- 使用S3存储hbase数据，见链接：[hbase s3](https://docs.amazonaws.cn/emr/latest/ReleaseGuide/emr-hbase-s3.html)，对于hbase，在已有emr集群进行挂载的情况下，需要勾选只读模式

## flume sink to s3

- 可参考以下链接进行：[sink to s3](https://medium.com/inspiredbrilliance/upload-files-to-aws-s3-using-apache-flume-c3464f6a2092)
- 大概步骤如下
  - 到flume官网下载相关应用程序，如1.10:[flume user guide](https://flume.apache.org/releases/content/1.10.0/FlumeUserGuide.html)
  - 在下载包中增加pom.xml，使用命令行下载对应的依赖包
  - 配置参考如下：
  
core-site.xml

```linux
<configuration>
        <property>
                <name>fs.s3a.impl</name>
                <value>org.apache.hadoop.fs.s3a.S3AFileSystem</value>
        </property>
        <property>
                <name>fs.s3a.connection.ssl.enabled</name>
                <value>true</value>
        </property>
        <property>
                <name>fs.s3a.endpoint</name>
                <value>s3.us-east-1.amazonaws.com</value>
        </property>
</configuration>
```

demo.conf

```linux
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 4444

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = s3a://flume-sink-s3-8339/demo-sink/
a1.sinks.k1.hdfs.fileType = DataStream
a1.sinks.k1.hdfs.filePrefix = demo-sink
a1.sinks.k1.hdfs.writeFormat = TEXT
a1.sinks.k1.hdfs.rollCount = 0
a1.sinks.k1.hdfs.rollSize = 128
a1.sinks.k1.hdfs.batchSize = 100
a1.sinks.k1.hdfs.rollInterval = 60

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

## datax to s3

- 下载相应源码[datax](https://github.com/alibaba/DataX)及[datax s3 plugin](https://github.com/crazyoyo/datax-s3-plugin)
- datax最新源码不支持编译s3 plugin,需要切换到分支*dependabot/maven/ch.qos.logback-logback-classic-1.2.0*,切换分支方法可参考[swithc branch](https://devconnected.com/how-to-switch-branch-on-git/)

## s3distcp

- 使用s3distcp进行存储数据的迁移，可参考[official](https://docs.aws.amazon.com/zh_cn/emr/latest/ReleaseGuide/UsingEMR_s3distcp.html)，可参考demo:[s3distcp](https://aws.amazon.com/cn/blogs/china/seven-tips-for-using-s3distcp-on-amazon-emr-to-move-data-efficiently-between-hdfs-and-amazon-s3/)

## workshop地址

- 可参考workshop进行emr的操作，参考：[workshop](https://emr-etl.workshop.aws/hive_workshop/01-hive-cli.html)
