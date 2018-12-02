---
layout: post
title: DB For Test Result
subtitle: DB Desgin for test result of message import
date: 2018-12-02
author: BF
header-img: img/bf/lily_03.jpg
catalog: true
tags:
  - database
  - sql
  - E-R
  - work
  - json
---

# 背景

之前做的自动化测试，最后的测试结果都是通过邮件发送给相关人员，没有做持久化，所以有时候想找一个版本的结果要去翻邮件列表，而且很有可能找不到。

接下来我希望能把最后的测试结果存放在数据库中，这样就可以方便对历史结果进行查询，充分利用数据库的特性。

# 需求分析

目前一些测试用例已经固定，变化不会太大，像 Benchmark 和 Message Import 都有一批固定的数据会重复跑。不过目标还是能创建相对通用的数据结构， 让大多数的结果都能存放。

希望最后达到的效果：

- 历次测试结果都能保存下来
- 能区别出测试的有效性，有时候测试因为环境或者别的原因失败了，那么是没有参考价值的
- 可以方便查询某一版本，某一参数或者某一天的测试结果（最好能直接通过 sql 查询到）

## 现状分析

测试最直接的结果是生成在 testResult.csv 文件中，主要结构类似如下：

| TestCase      | Msg Nums | CostTime(s) | Failed to Load/Post |
| :------------ | -------: | ----------: | ------------------: |
| TC01_MsgType1 |    30000 |        1200 |                 0/0 |
| TC02_MsgType2 |    45000 |        1652 |                 0/0 |
| TC03_MsgType3 |    45000 |        2320 |                 0/0 |

测试完成后，会根据 TestResult.xlsx.template 生成一份 excel 报告，额外包含一些比如 CPU/Memory 及使用的参数等信息。最后通过 SMTP 服务器发送到全组邮件列表。

# 设计方案

## 方案一：将测试结果文件直接保存在数据库中

这个方案是简单的，做的工作也最少，只要在数据库中建立一张表，建立几个必要的字段(e.g. date, version)再加上文件 IO 流就好了，比如：

| TestVersion     | APPVersion | TotalTime | FileResult |
| --------------- | ---------- | --------- | ---------- |
| 20181202_121232 | P01        | 2450      | xxx        |
| 20181202_134630 | P02        | 3600      | xxx        |
| 20181202_151256 | Po3        | 1660      | xxx        |

当然，直接把文档存在数据库也不太好，可以直接存在一个目录下，而在数据库里只存文件路径：

```sql
create table testresult(
    id int primary key auto_increment,
    datetimes datetime,
    testVersion char(15),
    APPVersion varchar(20),
    TotalTime int,
    FilePath varchar(30)
    -- File MediumBlob
);
```

但明显的，这个方案并不理想，比如我想查询一个 case 在一个时间段的平均花费时间，我没有方法用这个方式得到，打开几个对应的文档自己查看，那是不现实的。

## 方案二：将测试结果构造成 JSON 等数据结构

这个方案其实只是方案一的改进版，将测试结果数据抽象之后，可以做为 JSON 或者别的数据库支持的格式存储，可以使用 sql 直接查询其中的数据。

| TestVersion     | APPVersion | TotalTime | JsonResult |
| --------------- | ---------- | --------- | ---------- |
| 20181202_121232 | P01        | 2450      | {xxx}      |
| 20181202_134630 | P02        | 3600      | {xxx}      |
| 20181202_151256 | Po3        | 1660      | {xxx}      |

所以如果数据库支持的话，那也是相对方便的。 比如 JSON 可以类似如下：

```json
[
  {
    "TestCase": "TC01_MsgType1",
    "MsgNums": 30000,
    "CostTime": 1200,
    "FailedLoad": 0,
    "FailedPost": 0
  },
  {
    "TestCase": "TC01_MsgType1",
    "MsgNums": 30000,
    "CostTime": 1200,
    "FailedLoad": 0,
    "FailedPost": 0
  }
]
```

```sql
create table testresult(
    id int primary key auto_increment,
    datetimes datetime,
    testVersion char(15),
    APPVersion varchar(20),
    TotalTime int,
    JsonResult json DEFAULT NULL
    -- File MediumBlob
);
```

比如通过如下的语句就能找到一些想要的数据：

```sql
select JsonResult->'$[*].CostTime'  from testresult;
```

更多的 mysql 的操作请看最后参考中的一些文档。

## 方案三：使用传统数据格式来存储

因为公司里不方便用 mysql 之类的数据库，现在的有 sqlserver，那么就直接利用吧。

这样的话就要设计更多的字段了，我画了一个简单的 E-R 图：

![ER Map](/img/post/2018/12/2018-12-02-TestResultDBDesign.jpg)

像这样如果想要查询某一段时间一个 case 的平均时间就很方便了：

```sql
select AVG(TD.CostTime) from TestResult TR, TestDetails TD, TestCases TC
where TD.TestResultID = TA.ID and TD.TestCaseID = TC.ID
and TC.TestCaseName = 'ImportMsg01'
and TR.TestTime > '2018-12-01'
group by TD.CostTime
```

# 最后

主要思路这样差不多了，真的在做的时候应该会有许多细节，到时完善。
这个是接下来这周的一个小目标，加油。

---

参考：

- [mysql 对 json 数据的使用](https://blog.csdn.net/qq_36213352/article/details/83054993)
- [对 MySQL 中 JSON 数据类型的操作和分析](https://blog.csdn.net/manongpengzai/article/details/77200399)
- [MySQL5.7 新特性之 JSON 类型](https://blog.csdn.net/zhaowen25/article/details/52938004)
- [mysql5.7Json 数组解析](https://blog.csdn.net/u013329580/article/details/77096630)
