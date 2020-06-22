---
layout: post
title: HDFS Space Usage
subtitle: Hadoop command to check space usage
date: 2020-06-22
author: BF
header-img: img/bf/hadoop.jpg
catalog: true
tags:
  - Hadoop
  - HDFS
---

# 背景

需要统计HDFS的空间使用情况，有命令`hadoop fs -du /path`支持查看(hdfs dfs等效)。

但这样子只能看一个目录的情况，如果想要遍历所有的目录没有直接支持的。

# 方案

默认`-du` 是查看所给目录下子目录的占用空间大小，而加上`-s`则是列出给定的目录所占的空间。
所以如果给定所有已知的目录，那么就可以使用类似下面的语法
```cmd
hadoop fs -du -s /path1 /path2 /path3
```

所以我们可以想办法先得到想要的目录,通过`-ls -R`遍历所有目录和文件:
```cmd
[root@hadoop00 ~]# hadoop fs -ls -R /
drwxr-xr-x   - root supergroup          0 2020-06-22 23:25 /cx
drwxr-xr-x   - root supergroup          0 2020-03-01 21:25 /cx/test
-rw-r--r--   3 root supergroup         66 2020-03-01 21:25 /cx/test/salary.csv
drwxr-xr-x   - root supergroup          0 2020-06-22 23:28 /cx/test2
-rw-r--r--   3 root supergroup  879999912 2020-06-22 23:28 /cx/test2/data.csv
drwxr-xr-x   - root supergroup          0 2019-02-17 21:23 /data
drwxr-xr-x   - root supergroup          0 2019-02-17 21:25 /data/input
-rw-r--r--   3 root supergroup         68 2019-02-17 21:25 /data/input/word-count-data.txt
drwxr-xr-x   - root supergroup          0 2019-02-17 21:23 /data/output
drwxr-xr-x   - root supergroup          0 2019-02-17 22:10 /folder1
-rw-r--r--   3 root supergroup         20 2019-02-17 22:10 /folder1/test.txt
drwx-wx-wx   - root supergroup          0 2020-03-01 23:20 /user
drwx-wx-wx   - root supergroup          0 2020-03-01 23:27 /user/hive
drwx-wx-wx   - root supergroup          0 2020-03-01 23:20 /user/hive/tmp
drwx--x--x   - root supergroup          0 2020-03-01 23:51 /user/hive/tmp/root
drwx-wx-wx   - root supergroup          0 2020-03-01 23:27 /user/hive/warehouse
drwx-wx-wx   - root supergroup          0 2020-03-01 23:30 /user/hive/warehouse/myhive.db
```

之后通过`grep ^d`筛选出目录，再使用`awk`去掉不想要的值(带hive)。

```cmd
[root@hadoop00 ~]# hdfs dfs -ls -R / | grep ^d | awk '$8!~/.*hive.*/ {print $8}'
/cx
/cx/test
/cx/test2
/data
/data/input
/data/output
/folder1
/user
```

最后，把上面的结果作为参数传回给`-du`命令, 可以得到想要的结果。
```cmd
[root@hadoop00 ~]# hdfs dfs -du -h -s `hdfs dfs -ls -R / | grep ^d | awk '$8!~/.*hive.*/ {print $8}'`
839.2 M  /cx
66  /cx/test
839.2 M  /cx/test2
68  /data
68  /data/input
0  /data/output
20  /folder1
0  /user
```
`-h`是把size转成好理解的格式。

# 参考

- [Hadoop fs -du -s -h 输出三列数据的含义](https://blog.csdn.net/weixin_39457142/article/details/102611594)
- [Linux awk 命令](https://www.runoob.com/linux/linux-comm-awk.html)
- [Linux三剑客之awk命令](https://www.cnblogs.com/ginvip/p/6352157.html)
