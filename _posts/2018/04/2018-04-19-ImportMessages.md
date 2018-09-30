---
layout:     post
title:      Import Messages
subtitle:   Progress to import messages by python
date:       2018-04-19
author:     BF
header-img: img/bf/flower_01.jpg
catalog: true
tags:
    - python
    - test
---
## Import Messages
工作中需要导Message的性能测试作为benchmark，最早的时候有同事使用运行批处理的方式，一些配置和环境都是固定的，也不够灵活。

所以我趁着之前准备数据的基础，我搭建了一个简单的相对灵活的自动化流程。到后期稳定之后，可以做到run at everywhere。
当然，目前回滚数据库这一块还没有做到代码里，最后有需要的话可以加进来。

整个流程其实很简单：
* 读取配置文件
* Check环境，将一些需要自定义的配置文件指向一个固定的公共folder
* 启动基于java的Message Service，来接收messages
* 按配置文件中的文件夹信息，根据其下的txt文件名顺序依次导入message，并check数据库和import log，输出到log文件中

更具体的信息参考：[README](https://github.com/bearfly1990/PowerScript/blob/master/Private/Engineering/ImportMessages/README.md)

主要代码：[Import Messages](https://github.com/bearfly1990/PowerScript/blob/master/Private/Engineering/ImportMessages)
