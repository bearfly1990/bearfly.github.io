---
layout: post
title: Performance Test Training (1)
subtitle: Performance Test Training Day 1 at SH
date: 2018-08-25
author: BF
header-img: img/bf/basketball_01.jpg
catalog: true
tags:
  - test
  - training
---

# Performance Test Training (1)

今天是性能测试培训的第一天，主要内容是性能测试涉及到的概念，以及讲师自己的一些实践，个人感觉还是有些收获的。

下面就简单总结下一点知识。

## Performance Idicator

针对目前大多数的系统，都可以发起请求，来得到结果。尤其是`REST API`这种形式的，比如之前针对`Publisher Rest API`，就有对应的`SoupUI`的cases，也有基于`Jmeter`对api的反应时间(Response Time)进行的测试。

总的来说，有下面三个主要的指标:

* TPS (Transaction Per Second)
* Response Time
* Error Rate

### TPS

`Transaction per second`指是一秒钟时间内能完成的事务数，是一个最基本最重要的指标。一般来说，在制定性能测试计划的时候，会设定目标TPS的目标值，从具体的需求中抽离出来。

`TPS`的计算方式为1/RT*users，比如一个用户实例时，假设RT为0.2,则`TPS = 1/0.2=20`

在理想情况下，假设RT不变，则`TPS`如下所示：

| Users(Threads) | RT  | TPS        |
| -------------- | --- | ---------- |
| 1              | 0.2 | 1/0.2*1=5  |
| 2              | 0.2 | 1/0.2*2=10 |
| 3              | 0.2 | 1/0.2*3=15 |
| ...            | ... | ...        |

### Response Time(RT)

系统反应时间一般不会一成不变的，当系统请求越来越多的时候，反应时间会逐渐延长。所以结合上请求数，`TPS`一般为一个曲线，并且有峰值。

当超过之后，便会开始达到系统处理的瓶颈，使反应时间增加，反而使`TPS`减小。所以一般我们会从小变大线程数来得到`TPS`的结果，示意图如下，实际没有这么平滑。

![TPS](/img/post/2018-08-25-PerfTestTraining-TPS.jpg)

### Error Rate

当线程增加的时候，便会增加应用的负荷，便可能会有error产生，这个时候便有一个指标便是`Error Rate`，即一批线程中有错误的比率，比如一个正常的请求返回繁忙的错误。

尤其是在达到最大的`TPS`的时候，如果错误率很高的话，那也是没有意义的。

### Others

一般来说，性能指标能否完成，主要看以下：

* 速度相关(TPS / RT)
* 容量相关(吞吐, Hit, PV)
* 资源相关(CPU, Memory, IO)

我们很多时候会先定义性能指标，而最后的测试是为了估算应用最后能否达到预期的数值，给予项目人员及时的反馈，并帮助分析性能的瓶颈，甚至找到最根本的原因。

## Performance Test Roles

上面讲的是性能测试中的表象和指标，其实只是性能测试一个最基本的一个方面，理论上来说，性能测试工作可以划分为以下几个角色：

* 性能脚本工程师
* 性能场景设计工程师
* 性能分析工程师

### Performance Scripts Engineer

这个最好理解，同时也是我最近的角色，负责Messaage Import的性能测试环境的搭建，性能相关数据的采集。

相对来说，大部分人都能做到这个程度，毕竟技术门槛不高，而且目标比较明确。

### Performance Scenario Design Engineer

这一块的话，更像是目前David和陶总担任的角色。因为从用户需求角度和项目理解熟悉程度，我还没有那么高，同时我更多的精力是在完成测试脚本和执行测试计划。

当然，在一些情况下，我也有自己的一些测试方向和想法，但因为资源有限的情况下，还是要优先以客户需求为导向的测试。

希望之后我也能更多承担起这块的角色，但这个前提是我对业务还有系统的功能有更深入的了解。

因为测试数据的建立对我来说就有一定的难度，需要大家的support.

### Performance Analysis Engineer

这个角色是性能测试最核心也是最难的一个角色。需要对应用，业务都了解的情况下，掌握非常好的分析方法和经验，会使用各种分析工具和系统级命令。

从最直接的性能结果开始，最终达到代码级别的诊断与分析。

一般来说都是一步一步精细化下去：

现象 -> OS -> CPU -> Processor -> Thread ->Java/C/...-> dump -> code

## Performance Test Knowledge

性能测试要做的好，需要掌握了解许多的知识，也是一个很庞大的知识体系：

* 压力工具(Load Runner/Jmeter/...)
* 监控分析工具
* 中间件(MQ/Tuxedo/...)
* 网络(Network)，网络性能分析
* OS(Windows/Linux/...)
* 应用服务器(Apache/Nginx/...)
* DB
* 存储
* Code
* ...

## Last

今天还介绍了许多实用的知识，比如Linux系统的架构，常用的一些网络协议，系统自己的性能查看工具等。

之后有时候，再自己多实践与总结。

晚安。