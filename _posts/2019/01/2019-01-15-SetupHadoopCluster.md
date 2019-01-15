---
layout: post
title: Setup Hadoop Cluster
subtitle: 搭建Hatoop集群环境
date: 2019-01-15
author: BF
header-img: img/bf/hadoop.jpg
catalog: true
tags:
  - java
  - hadoop
  - big data
---

# 背景

最近终于在虚拟上搭好了 Hadoop 的集群环境，记录一下。

# 资源准备

- jdk-8u40-linux-x64.gz
- hadoop-2.7.3.tar.gz
- CentOS Linux release 7.4.1708 (Core)

四台机子如下，hadoop00 为**mater**, 其余为**slave**

```
192.168.137.100 hadoop00
192.168.137.101 hadoop01
192.168.137.102 hadoop02
192.168.137.103 hadoop03
```

# 配置环境

- jdk: `/usr/local/apps/jdk1.8.0_40`
- hadoop: `/usr/local/apps/hadoop-2.7.3`

编辑`/etc/profile`, 将 jdk 和 hadoop 相关 key 添加到环境变量中。

```bash
export JAVA_HOME=/usr/local/apps/jdk1.8.0_40
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export HADOOP_HOME=/usr/local/apps/hadoop-2.7.3
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HADOOP_HOME/lib

export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
#export HADOOP_OPTS="-Djava.library.path=$HADOOP_PREFIX/lib:$HADOOP_PREFIX/lib/native"
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native
```

然后执行`source /usr/profile` 使配置生效。

创建为 Hadoop 之后使用的目录

```bash
mkdir -p /usr/local/hadoop/tmp
mkdir -p /usr/local/hadoop/hdfs/name
mkdir -p /usr/local/haddop/data
```

## 设置 ssh 免密登录

下面是生成密钥对的方式。
生成公钥：

```bash
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
```

将公钥加入验证：

```bash
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
```

尝试登录验证：

```bash
ssh localhost
```

最后我们需要每一台都能互相访问，所以需要将所有的公钥加入**authorized_keys**

那我们就要每次都通过`scp`将私钥传递下去，最后再将最后的**authorized_keys**传回每一台：

```
scp /root/.ssh/authorized_key hadoop00:/root/.ssh
scp /root/.ssh/authorized_key hadoop01:/root/.ssh
scp /root/.ssh/authorized_key hadoop02:/root/.ssh
```

# Hadoop 配置与运行

Hadoop 的解压路径为 `/usr/local/apps/hadoop/hadoop-2.7.3`

在`/usr/local/apps/hadoop/hadoop-2.7.3/hadoop/etc`目录下有许多配置文件需要我们配置。

## hadoop-env.sh

主要是修改使用我们自己的 jdk 路径

```
export JAVA_HOME=/usr/local/apps//jdk1.8.0_40
```

## yarn-env.sh

同样是修改 JAVA_HOME 路径：

```
export JAVA_HOME=/usr/local/apps/jdk1.8.0_40
```

## core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop00:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/apps/hadoop-2.7.3/tmp</value>
    </property>
    <!--
    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>
    -->
</configuration>
```

## hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/apps/hadoop-2.7.3/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/apps/hadoop-2.7.3/hdfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop00:9001</value>
    </property>
    <property>
        <name>dfs.namenode.servicerpc-address</name>
        <value>hadoop00:10000</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
</configuration>
```

## mapred-site.xml

这个文件从 mapred-site.xml.template 拷过来改：

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop00:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop00:19888</value>
    </property>
</configuration>
```

## yarn-site.xml

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop00</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>hadoop00:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>hadoop00:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>hadoop00:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>hadoop00:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>hadoop00:8088</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>1024</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>1</value>
    </property>
</configuration>
~
```

## 格式化 namenode

```shell
hdfs namenode -format
```

## 启动

将 HDFS 和 YARN 都起来：

```shell
# start HDFS
start-dfs.sh
# start YARN
start-yarn.sh
```

然后使用 jps 查看 java 进程`jps`,可以看到我们想要的都起来了。

```shell
[root@hadoop00 hadoop]# jps
9057 ResourceManager
9318 Jps
8631 NameNode
8860 SecondaryNameNode
```

```shell
[root@hadoop01 ~]# jps
7844 NodeManager
7689 DataNode
7978 Jps
```

## 通过网址访问

如果启动成功，我们便能成功访问下列的网址：

（注意）我们需要将防火墙关闭（当然也可以把用到的端口配置一下）

[http://hadoop00:8088](http://hadoop00:8088/)

![Yarn](/img/post/2019/01/2019-01-15-SetupHadoopCluster.01.jpg)

[http://hadoop00:50070](http://hadoop00:50070)

![Yarn](/img/post/2019/01/2019-01-15-SetupHadoopCluster.02.jpg)

# 最后

目前搭的只是最基础的环境，后继用到其它组件时还需要继续添加。

---

参考：《Hadoop构建数据仓库实践》等
