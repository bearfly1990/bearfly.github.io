---
layout: post
title: pyodbc unicode issue
subtitle: 数据插入失败
date: 2019-12-24
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - python
  - pyodbc
  - unicode
---

**这两天**SRD这边有文件中存在乱码，导致那一行数据插入失败。

下面是我晚上在家里电脑重现的时候信息,所以是中文的：
```
pyodbc.Error: ('HY090', '[HY090] [Microsoft][ODBC 驱动程序管理器] 无效的字符串或缓冲区长度 (0) (SQLBindParameter)')
```
我一开始以为是这列数据的字符串长度过长，有1万多个字符，但是我用别的值试了一下，是没有问题的。

然后我尝试着使用print输出来看一下，竟然直接error了。
下面是例子：
```python
text = "you are right \udef6 thanks"
print(text)
# UnicodeEncodeError: 'utf-8' codec can't encode character '\udef6' in position 14: surrogates not allowed
```

目前连接Sql Server数据库用的是pyodbc，而现在在插数据的时候挂了。

我想到这个问题可能与pyodbc处理unicode的编码有关系，就去找官方的资料。

最后发现在pyodbc的github库的issue list里，已经存在类似的问题 [#617](https://github.com/mkleehammer/pyodbc/issues/617)。

(在这里吐槽一下，现在公司把github都封掉了，不让访问，怕有人把公司代码上传上去。。。
现在查到一些链接只能用google的快照打开。。。)

**就像**issue里描述的，对于一些特殊字符(e.g. 🎥, ☯)，当使用`fast_executemany`，
并且要插的数据类型是varchar(max)的时候， pyodbc不能很好的处理。

如果不使用这个模式(默认fast_executemany=False)，就能正常的插入到数据库中（虽然还是乱码），不会报错了。

但如果不使用这个模式，执行插入的操作就会很慢。

于是我又发现了一个孪生的库[pypyodbc](https://github.com/jiangwen365/pypyodbc)。
可以认为他是pyodbc的python版本的实现，API使用基本一样，可以直接替换：
```python
import pypyodbc as pyodbc
```

**当然**， 有时候我不想换个库，只能等pyodbc更新修复，或者自己编译一个作者修复过的版本[v-makouz@606b4a9](https://github.com/v-makouz/pyodbc/commit/606b4a959a8531e8cf1349da0f828ee8bc2e46f0),
 又或者自己预处理一下字符串。

乱码是处理数据的时候常会遇到的，最好有统一的处理方式。

PS：对于纯英文环境，我们完全可以把编码值>255的都当成特殊字符去掉。