---
layout: post
title: Single Node Hadoop
subtitle: Setup single node Hadoop in linux
date: 2018-12-12
author: BF
header-img: img/bf/mountain_03.jpg
catalog: true
tags:
  - java
  - hadoop
  - big data
---

# 背景

今天在阿里云机器上试着搭了一个单结点的 Hadoop, 记录一下过程。

# 资源准备

使用的 jdk 和 Hadoop 版本分别为：

- jdk-8u40-linux-x64.gz
- hadoop-2.7.3.tar.gz

系统是阿里云的 CentOS Linux release 7.4.1708 (Core)

# 配置环境

## JDK

jdk 解压在`/usr/local/jdk1.8.0_40`目录下。

编辑`/etc/profile`, 将 jdk 添加到环境变量中。

```bash
JAVA_HOME=/usr/local/jdk1.8.0_40
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

然后执行`source /usr/profile` 使配置生效。

## Hadoop

设置环境变量`vim ~/.bash_profile`：

```bash
HADOOP_HOME=/usr/local/hadoop/hadoop-2.7.3
PATH=$PATH:$HADOOP_HOME/bin
export HADOOP_HOME PATH
```

使配置生效`source ~/.bash_profile`

创建为 Hadoop 之后使用的目录

```bash
mkdir -p /usr/local/hadoop/tmp
mkdir -p /usr/local/hadoop/hdfs/name
mkdir -p /usr/local/haddop/data
```

## 设置 ssh 免密登录

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

## 设置主机

将主机名设置成 Master `vim /etc/hostname`:

```bash
Master
```

然后让 IP 和主机名绑定 `vim /etc/hosts`：

```bash
xxx.xxx.xxx.xxx Master
```

# Hadoop 配置与运行

Hadoop 的解压路径为 `/usr/local/hadoop/ hadoop-2.7.3`

在`/usr/local/hadoop/ hadoop-2.7.3/hadoop/etc`目录下有许多配置文件需要我们配置。

## hadoop-env.sh

主要是修改使用我们自己的 jdk 路径

```
export JAVA_HOME=/usr/local/jdk1.8.0_40
```

## yarn-env.sh

同样是修改 JAVA_HOME 路径：

```
export JAVA_HOME=/usr/local/jdk1.8.0_40
```

## core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://Master:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/tmp</value>
    </property>
</configuration>
```

## hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/hdfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

## mapred-site.xml

这个文件从 mapred-site.xml.template 拷过来改：

```xml
<configuration>
    <property>
        <name>mapreduce.framwork.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

## yarn-site.xml

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

## 格式化 namenode

```
cd /usr/local/hadoop/hadoop-2.7.3/bin/
./hdfs namenode -format
```

## 启动

运行 sbin 目录下的 start-all.sh

```
cd /usr/local/hadoop/hadoop-2.7.3/sbin/
./start-all.sh
```

然后使用 jps 查看 java 进程`jps`,可以看到我们想要的都起来了。

```bash
[root@izbp162mggaelq3xw6nk65z sbin]# jps
1061 Application
7765 jar
4903 ResourceManager
4475 NameNode
5003 NodeManager
5292 Jps
4750 SecondaryNameNode
```

## 通过网址访问

如果启动成功，我们便能成功访问下列的网址：

[http://www.bearfly.fun:8088/cluster](http://www.bearfly.fun:8088/cluster)

[http://www.bearfly.fun:50070/dfshealth.html#tab-overview](http://www.bearfly.fun:50070/dfshealth.html#tab-overview)

# 最后

还是要想办法真正搭个集群的环境，不行只能虚拟机上了，最好能再搞到一台机器。

---

参考：网络视频
