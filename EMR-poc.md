# emr workshop

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

### 相关步骤

下载DataX, 下载S3插件

```linux
git clone https://github.com/alibaba/DataX
git clone https://github.com/crazyoyo/DataXS3Plugin
```

复制数据到DataX

```linux
cp -r DataXS3Plugin/s3* DataX/
```

切换到相应分支

```linux
git branch
* dependabot/maven/ch.qos.logback-logback-classic-1.2.0
```

修改pom.xml，增加s3reader及s3writer，如下显示的73、103行

```linux
 73         <module>s3reader</module>
 74
 75         <!-- writer -->
 76         <module>mysqlwriter</module>
 77         <module>drdswriter</module>
 78         <module>odpswriter</module>
 79         <module>txtfilewriter</module>
 80         <module>ftpwriter</module>
 81         <module>hdfswriter</module>
 82         <module>streamwriter</module>
 83         <module>otswriter</module>
 84         <module>oraclewriter</module>
 85         <module>sqlserverwriter</module>
 86         <module>postgresqlwriter</module>
 87         <module>kingbaseeswriter</module>
 88         <module>osswriter</module>
 89         <module>mongodbwriter</module>
 90         <module>adswriter</module>
 91         <module>ocswriter</module>
 92         <module>rdbmswriter</module>
 93         <module>hbase11xwriter</module>
 94         <module>hbase094xwriter</module>
 95         <module>hbase11xsqlwriter</module>
 96         <module>hbase11xsqlreader</module>
 97         <module>elasticsearchwriter</module>
 98         <module>tsdbwriter</module>
 99         <module>adbpgwriter</module>
100         <module>gdbwriter</module>
101         <module>cassandrawriter</module>
102         <module>clickhousewriter</module>
103         <module>s3writer</module>
```

修改package.xml，分别增加s3write及s3reader

```linux
<fileSet>
        <directory>s3writer/target/datax/</directory>
        <includes>
        <include>**/*.*</include>
        </includes>
        <outputDirectory>datax</outputDirectory>
</fileSet>

<fileSet>
        <directory>s3reader/target/datax/</directory>
        <includes>
        <include>**/*.*</include>
        </includes>
        <outputDirectory>datax</outputDirectory>
</fileSet>
```

编译环境及编译命令如下

```linux
mvn --version

Apache Maven 3.0.5 (Red Hat 3.0.5-17)
Maven home: /usr/share/maven
Java version: 1.8.0_342, vendor: Red Hat, Inc.
Java home: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.342.b07-1.amzn2.0.1.x86_64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "5.10.135-122.509.amzn2.x86_64", arch: "amd64", family: "unix"

mvn clean install -Dmaven.test.skip=true
```

### job示例

从mysql导出数据到s3

```linux
{
    "job": {
        "setting": {},
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "column": [
                            "a",
                            "b",
                            "c"
                        ],
                        "connection": [
                            {
                                "jdbcUrl": [
                                    "jdbc:mysql://rds-mysql.xxxxx.us-east-1.rds.amazonaws.com:3306/test?characterEncoding=utf8"
                                ],
                                "table": [
                                    "t"
                                ]
                            }
                        ],
                        "password": "xxxxxxx",
                        "username": "xxxx"
                    }
                },
                "writer": {
                    "name": "s3writer",
                    "parameter": {
                        "endpoint": "s3.us-east-1.amazonaws.com",
                        "region": "us-east-1",
                        "accessId": "",
                        "accessKey": "",
                        "bucket": "emr-demo-xxx",
                        "object": "datax",
                        "encoding": "UTF-8",
                        "fieldDelimiter": ",",
                        "writeMode": "append"
                    }
                }
            }
        ]
    }
}
```

## s3distcp

- 使用s3distcp进行存储数据的迁移，可参考[official](https://docs.aws.amazon.com/zh_cn/emr/latest/ReleaseGuide/UsingEMR_s3distcp.html)，可参考demo:[s3distcp](https://aws.amazon.com/cn/blogs/china/seven-tips-for-using-s3distcp-on-amazon-emr-to-move-data-efficiently-between-hdfs-and-amazon-s3/)

- 使用s3distcp进行存储数据的迁移，可参考[official](https://docs.aws.amazon.com/zh_cn/emr/latest/ReleaseGuide/UsingEMR_s3distcp.html)，可参考demo:[s3distcp](https://aws.amazon.com/cn/blogs/china/seven-tips-for-using-s3distcp-on-amazon-emr-to-move-data-efficiently-between-hdfs-and-amazon-s3/)

对于从hdfs中转到S3中的hive表数据，对分区的支持需要进行修复，执行如下命令，test2对应需要修复的表名

```linux
msck repair table test2;
```

## workshop地址

- 可参考workshop进行emr的操作，参考：[workshop](https://emr-etl.workshop.aws/hive_workshop/01-hive-cli.html)
