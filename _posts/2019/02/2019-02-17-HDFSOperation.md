---
layout: post
title: Hadoop HDFS Operation
subtitle: HDFS的基本操作
date: 2019-02-17
author: BF
header-img: img/bf/hadoop.jpg
catalog: true
tags:
  - java
  - hadoop
  - big data
---

# 背景

之前搭好了 Hadoop 环境，但是在使用的过程中还是有一些问题，现在终于解决了，至少最基础的环境没有问题了。

# 基本环境

Hadoop-2.7.3 / Java7

四台机子如下，**hadoop00** 为**mater**, 其余为 slave

```
192.168.137.100 hadoop00
192.168.137.101 hadoop01
192.168.137.102 hadoop02
192.168.137.103 hadoop03
```

# HDFS 的相关命令

```
-mkdir            在HDFS创建目录    hdfs dfs -mkdir /data
-ls               查看当前目录      hdfs dfs -ls /
-ls -R            查看目录与子目录
-put              上传一个文件      hdfs dfs -put data.txt /data/input
-moveFromLocal    上传一个文件，会删除本地文件：ctrl + X
-copyFromLocal    上传一个文件，与put一样
-copyToLocal      下载文件  hdfs dfs -copyToLocal /data/input/data.txt
-get              下载文件  hdfs dfs -get /data/input/data.txt
-rm               删除文件  hdfs dfs -rm /data/input/data.txt
-getmerge         将目录所有的文件先合并，再下载
-cp               拷贝： hdfs dfs -cp /data/input/data.txt  /data/input/data01.txt
-mv               移动： hdfs dfs -mv /data/input/data.txt  /data/input/data02.txt
-count            统计目录下的文件个数
-text、-cat       查看文件的内容  hdfs dfs -cat /data/input/data.txt
-balancer         平衡操作
```

# 使用 Java 进行 HDFS 基本操作

下面是一个非常简单的例子，需要说明的是，"root"是那台 Linux 上有权限的用户名，不然会报权限错误，有多种解决方案，可以参考：[HDFS JAVA 客户端的权限错误：Permission denied](https://www.cnblogs.com/bkylry/p/8072132.html)

```java
package org.bearfly.fun.hadooplearn;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

/**
* @Description: App
* @author bearfly1990
* @date Feb 17, 2019 10:18:42 PM
*/
public class App {
    public static void main(String[] args) throws IOException, InterruptedException, URISyntaxException {
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop00:9000"), conf, "root");
        fs.mkdirs(new Path("/folder1"));

        Path src = new Path("d:/test.txt");
        Path dst = new Path("/folder1");
        fs.copyFromLocalFile(src, dst);
        fs.close();
    }
}

```

网上有更多详细的例子。

最后的运行结果如下（加上上面创建的文件）：

```bash
[root@hadoop00 ~]# hdfs dfs -ls -R /
drwxr-xr-x   - root supergroup          0 2019-02-17 21:23 /data
drwxr-xr-x   - root supergroup          0 2019-02-17 21:25 /data/input
-rw-r--r--   3 root supergroup         68 2019-02-17 21:25 /data/input/word-count-data.txt
drwxr-xr-x   - root supergroup          0 2019-02-17 21:23 /data/output
drwxr-xr-x   - root supergroup          0 2019-02-17 22:10 /folder1
-rw-r--r--   3 root supergroup         20 2019-02-17 22:10 /folder1/test.txt

```

# WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

在使用的过程中一直有报这样的 Warning,虽然可以正常运行，但是看着很烦。

网上有许多的问题和对应解决方案，而我这个编译的版本什么都是一样的，只要在环境变量中添加 HADOOP_OPTS 即可。

具体分析的时候可以把 Hadoop 的 debug log 打开`$ export HADOOP_ROOT_LOGGER=DEBUG,console`，找到对应的信息。

```bash
# /etc/profile
export HADOOP_OPTS="-Djava.library.path=${HADOOP_HOME}/lib/native"
```

# DataNode Crashed

在使用的过程中，我还遇到了DataNode失效了。我找不到原因，所以stop hdfs 和 yarn 之后重新format了一下就好了。

注：在这过程中我关闭了防火墙，删除原始的目录，排除了可能的影响。

更多信息可以参考最后的链接。

---

参考：

* [HDFS JAVA 客户端的权限错误：Permission denied](https://www.cnblogs.com/bkylry/p/8072132.html)
* [Hadoop 出现错误：WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable，解决方案](https://www.cnblogs.com/likui360/p/6558749.html)
* [Hadoop之—— WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform...](https://blog.csdn.net/l1028386804/article/details/51538611)
* [Hadoop之——重新格式化HDFS的方案](https://blog.csdn.net/l1028386804/article/details/72857449)
* [java使用FileSystem上传文件到hadoop文件系统](https://www.cnblogs.com/lizhonghua34/p/6437878.html)