---
layout: post
title: Not in replacement
subtitle: replace not in in sql
date: 2020-03-22
author: BF
header-img: img/bf/sql_01.jpg
catalog: true
tags:
  - sql
  - hive
---

# 背景

最近解决了一个 hive 中报错的问题。HY 从最底层的 view 开始检查，最后发现在当前的 Sandbox/UAT2 环境中，对于 not in 中使用子查询的支持有问题。

这个问题导致了后续引用的 view 在建立时解析出错，会出现 Range Error 和 hive sql 解析出错。

解决的办法是使用别的语句替换，比如 not exists。

今天就想记录一下一些可行的替换方法，主要还是 not exists 和用 join

# 样例

以下面简单的例子为例，有 Student 表和 Score 表， 想要得到分数不小于 60 分的名字
Table-Student

| id  | name |
| --- | ---- |
| 1   | cx   |
| 2   | xm   |
| 3   | zs   |
| 4   | lx   |

Table-Score

| id  | score |
| --- | ----- |
| 1   | 80    |
| 2   | 95.5  |
| 3   | 75    |
| 4   | 55    |

# not in

使用 `not in`的话，直接很好理解，小于60分的不要。
```sql
select name from Student where id not in (
	select id from Score where score < 60
)
```

# not exist
使用`not exist`需要id关联在一起
```sql
select name from Student a where not exists (
	select * from Score b where score < 60 and a.id = b.id
)
```

# join

可以直接使用反过来的逻辑使用join：
```sql
select name 
from Student a 
join Score b
on a.id = b.id and b.score >= 60
```

如果想要使用一样的not逻辑的话, 可以使用left join
```sql
select name 
from Student a 
left join Score b
on a.id = b.id and b.score < 60
where b.id is null
```

# 不建议使用not in

## 对null的处理不好
以下面为例：
```sql
select 'val' 
where 3 not in (1, 2, null);
```
Not In可以转换为条件对于每个值进行不等比对，并用逻辑与连接起来，Null值与任意其他值做比较时，结果永远为Null，在Where条件中也就是False，因此3<>null就会导致不返回任何行，导致Not In子句产生的结果在意料之外
```sql
select 'val'
where 3<> 1 and 3 <>2 and 3 <> null;
```
最后的返回结果为空，什么都没有，不是期望的，如果子查询里返回的结果里有Null,那么就会导致错误。

Note: 我在SqlServer, Mysql验证过，表现一样。

## Not In导致的查询性能低下

Not In需要对结果中可能的Null进行判断，有更多额外的开销，具体可以参考最后的那篇文章，分析的很详细，在一般情况下，都还是使用not exists替代更好。

# 参考
[在SQL Server中为什么不建议使用Not In子查询](https://www.cnblogs.com/CareySon/p/4955123.html)