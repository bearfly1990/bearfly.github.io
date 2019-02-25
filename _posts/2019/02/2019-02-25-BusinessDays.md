---
layout: post
title: Generate Business Days
subtitle: Calculate and count the business days
date: 2019-02-25
author: BF
header-img: img/bf/sql_01.jpg
catalog: true
tags:
  - sql
---

# 背景与需求

这两天有个小的需求，想要使用 Stored Procedure 根据 Holiday 的信息，将新的一年每月的 Business Day 数量计算出来并插入一张表中。

这个功能我已经实现，现在就记录一下一些觉得有意思的信息，简单的介绍下自己的思路。

# 简化需求

假设有一张`Holiday`表，表示每个国家的假期计划，其中假日可能和周末两天重合，那么那天假期其实是没用了，等于没放假。

现在假设需要构造 2019 年每个月的 Business Days(每个月除了周末和假期，假期与周末重合就算一天)

现有的 Holiday 表如下：

```sql
create table Holiday(
HolidayDate date,
Region varchar(15)
)
```

```sql
HolidayDate	Region
2019-01-01	China
2019-01-02	China
2019-04-01	China
2019-05-01	China
2019-07-07	China
2019-10-01	China
2019-01-05	US
2019-06-01	US
```

并有一张 Region 表：

```sql
create table Region(
Region varchar(15)
)
```

```sql
Region
China
US
```

# 构造 2019 月份数据

下面使用临时表来构造 2019 年的月份数据，每年的每个月的一号是确定的。

```sql
declare @Table_WorkDays table (StartDate Date, EndDate Date, WorkDayNums int);

insert into @Table_WorkDays(StartDate) values('2019-01-01');
insert into @Table_WorkDays(StartDate) values('2019-02-01');
insert into @Table_WorkDays(StartDate) values('2019-03-01');
insert into @Table_WorkDays(StartDate) values('2019-04-01');
insert into @Table_WorkDays(StartDate) values('2019-05-01');
insert into @Table_WorkDays(StartDate) values('2019-06-01');
insert into @Table_WorkDays(StartDate) values('2019-07-01');
insert into @Table_WorkDays(StartDate) values('2019-08-01');
insert into @Table_WorkDays(StartDate) values('2019-09-01');
insert into @Table_WorkDays(StartDate) values('2019-10-01');
insert into @Table_WorkDays(StartDate) values('2019-11-01');
insert into @Table_WorkDays(StartDate) values('2019-12-01');
```

```sql
StartDate	EndDate	WorkDayNums
2019-01-01	NULL	NULL
2019-02-01	NULL	NULL
2019-03-01	NULL	NULL
2019-04-01	NULL	NULL
2019-05-01	NULL	NULL
2019-06-01	NULL	NULL
2019-07-01	NULL	NULL
2019-08-01	NULL	NULL
2019-09-01	NULL	NULL
2019-10-01	NULL	NULL
2019-11-01	NULL	NULL
2019-12-01	NULL	NULL
```

接下来计算 EndDate，通过加一个月的时间再减去一天，即月末。

```sql
update @Table_WorkDays set EndDate = DATEADD(day, -1, DATEADD(month, 1, StartDate))
```

如果 sql server 版本 >= SQL Server 2012，可以使用`EOMonth`函数

```sql
update @Table_WorkDays set EndDate = EOMonth(StartDate)
```

```sql
StartDate	EndDate	        WorkDayNums
2019-01-01	2019-01-31	NULL
2019-02-01	2019-02-28	NULL
2019-03-01	2019-03-31	NULL
2019-04-01	2019-04-30	NULL
2019-05-01	2019-05-31	NULL
2019-06-01	2019-06-30	NULL
2019-07-01	2019-07-31	NULL
2019-08-01	2019-08-31	NULL
2019-09-01	2019-09-30	NULL
2019-10-01	2019-10-31	NULL
2019-11-01	2019-11-30	NULL
2019-12-01	2019-12-31	NULL
```

下面再计算出 workday 的天数，把一个月中的周末两天去掉：

```sql
update @Table_WorkDays set WorkDayNums =
	(DATEDIFF(dd, StartDate, EndDate) + 1)
	- (DATEDIFF(wk, StartDate, EndDate) * 2)
	- (case when DATENAME(dw, StartDate) = 'Sunday' then 1 else 0 end)
	- (case when DATENAME(dw, EndDate) = 'Saturday' then 1 else 0 end)
```

```sql
StartDate	EndDate	        WorkDayNums
2019-01-01	2019-01-31	23
2019-02-01	2019-02-28	20
2019-03-01	2019-03-31	21
2019-04-01	2019-04-30	22
2019-05-01	2019-05-31	23
2019-06-01	2019-06-30	20
2019-07-01	2019-07-31	23
2019-08-01	2019-08-31	22
2019-09-01	2019-09-30	21
2019-10-01	2019-10-31	23
2019-11-01	2019-11-30	21
2019-12-01	2019-12-31	22
```

结合 Region 表简单 join，得到每个 Region 2019 年的初始 WorkDayNums:

```sql
select * from @Table_WorkDays, Region
```

```sql
StartDate	EndDate	        WorkDayNums	Region
2019-01-01	2019-01-31	23	        China
2019-01-01	2019-01-31	23	        US
2019-02-01	2019-02-28	20	        China
2019-02-01	2019-02-28	20	        US
2019-03-01	2019-03-31	21	        China
2019-03-01	2019-03-31	21	        US
2019-04-01	2019-04-30	22	        China
2019-04-01	2019-04-30	22	        US
2019-05-01	2019-05-31	23	        China
2019-05-01	2019-05-31	23	        US
2019-06-01	2019-06-30	20	        China
2019-06-01	2019-06-30	20	        US
2019-07-01	2019-07-31	23	        China
2019-07-01	2019-07-31	23	        US
2019-08-01	2019-08-31	22	        China
2019-08-01	2019-08-31	22	        US
2019-09-01	2019-09-30	21	        China
2019-09-01	2019-09-30	21	        US
2019-10-01	2019-10-31	23	        China
2019-10-01	2019-10-31	23	        US
2019-11-01	2019-11-30	21	        China
2019-11-01	2019-11-30	21	        US
2019-12-01	2019-12-31	22	        China
2019-12-01	2019-12-31	22	        US
```

# 优化 Holiday 数据

一个月中可能有许多天假期，所以我们需要最后算一个月的有效假期（不和周末重合）。

首先得出月份

```sql
select CONVERT(date, DATEADD(month, DATEDIFF(month, 0, HolidayDate), 0)) as TheMonth, * from Holiday;
```

```sql
TheMonth	HolidayDate	Region
2019-01-01	2019-01-01	China
2019-01-01	2019-01-02	China
2019-04-01	2019-04-01	China
2019-05-01	2019-05-01	China
2019-07-01	2019-07-07	China
2019-10-01	2019-10-01	China
2019-01-01	2019-01-05	US
2019-06-01	2019-06-01	US
```

然后 group by `TheMonth` 和 `Region`的时候计算有效假期（不和周末重合）：

```sql
--#HolidayNew
select	TheMonth, Region,
        sum(case when DATENAME(dw, HolidayDate) in ('Saturday', 'Sunday') then 0 else 1 end) as ValidHolidayDays
from
        (select CONVERT(date, DATEADD(month, DATEDIFF(month, 0, HolidayDate), 0)) TheMonth,* from Holiday) as HolidayNew
group by TheMonth, Region
order by Region, TheMonth;
```

```sql
TheMonth	Region	ValidHolidayDays
2019-01-01	China	2
2019-04-01	China	1
2019-05-01	China	1
2019-07-01	China	0
2019-10-01	China	1
2019-01-01	US	0
2019-06-01	US	0
```

# 合并 WorkDay 与 Valid Holiday Day

将上面的#HolidayNew和最上面的初始workday结合一下，就可以得到最后的Business Day Nums。

```sql
select  temp.startDate as TheMonth, 
        temp.Region,
        WorkDayNums - isnull(hn.ValidHolidayDays, 0) as BusinessDays
from (select * from @Table_WorkDays, Region) as temp 
left join #HolidayNew hn 
on temp.StartDate = hn.TheMonth and temp.Region = hn.Region
```
```sql
TheMonth	Region	BusinessDays
2019-01-01	China	21
2019-01-01	US	23
2019-02-01	China	20
2019-02-01	US	20
2019-03-01	China	21
2019-03-01	US	21
2019-04-01	China	21
2019-04-01	US	22
2019-05-01	China	22
2019-05-01	US	23
2019-06-01	China	20
2019-06-01	US	20
2019-07-01	China	23
2019-07-01	US	23
2019-08-01	China	22
2019-08-01	US	22
2019-09-01	China	21
2019-09-01	US	21
2019-10-01	China	22
2019-10-01	US	23
2019-11-01	China	21
2019-11-01	US	21
2019-12-01	China	22
2019-12-01	US	22
```

# 最后

这次又回顾学习了下一些日期函数的学习，包括类型转换等，还可以。

---


