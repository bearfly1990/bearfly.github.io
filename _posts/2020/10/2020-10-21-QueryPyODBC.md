---
layout: post
title: Query hive data
subtitle: query data by pyodbc for hive
date: 2020-10-21
author: BF
header-img: img/bf/hadoop.jpg
catalog: true
tags:
  - Hadoop
  - hive
  - pyodbc
---

# 背景

有Hue可以用来查询hive和impala的数据，但是使用起来不是特别方便，尤其想要同时把数据导出来的时候。

原来想尝试用java的方式，也向同事要了demo，但是需要keytab，而且还是倾向于用python。

最后尝试了一些python库失败(主要是这些module依赖的东西比较多，需要很多环境的配置，但是都没有权限，比较麻烦)，从同事那得知他们的Tableau本地用的是odbc的连接方式。

之前试过用pyodbc去连接sql server数据库，而且理论上使用odbc，只要数据源配置好就可以了，api的使用是共通的，不管什么语言都可以用odbc的方式去访问。

# 第三方软件安装
1. Cloudera hive odbc (用使用impala的话就安装 impala odbc)， 并配置好连接与数据源
2. MIT Kerberos (需要使用Kerberos认证)

# 主界面
![2020-10-21-QueryPyODBC-02.jpg](/img/post/2020/10/2020-10-21-QueryPyODBC-02.jpg)

yaml是用来配置基本信息，像最后文件输出的目录也可以配置，当前默认`output`，lib放了一些自己封装的类和方法.

主界面可以看到可以选择不同的数据源（在配置文件里配置初始的数据），输入最后要存的文件名字（默认是test.{current_datetime}.csv)，输出的文件类型(csv/excel)，以及你的sql语句。
![2020-10-21-QueryPyODBC-01.jpg](/img/post/2020/10/2020-10-21-QueryPyODBC-01.jpg)

为了更灵活，当start之前，你可以更新信息，重新query，即支持多线程查询，不过不建议一次查太多，而且要注意输出文件的名字，如果重名的话数据就会被覆盖。

主要查询的代码如下图所示，封装了`HiveUtils`做一些细节处理。
![2020-10-21-QueryPyODBC-03.jpg](/img/post/2020/10/2020-10-21-QueryPyODBC-03.jpg)


# 查询多个table和view

上面的工具满足了特殊的sql的需要，但是有时候，我们希望一次能把许多table/view的数据导出来查看。

所以也写了另外的脚本，通过把想要导出来的table/view名字列在配置文件里，一次性起多个线程去查询。

e.g. object.txt
```
db1.table1
db1.table2
db2.view1
db3.view2
```
把上面的所以数据都查出来，最后放在output目录中，文件名按table/view名字来取，方便查看。

# 最后

后续会继续改进，提高平时的工作效率。