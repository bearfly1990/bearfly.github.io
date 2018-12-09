---
layout: post
title: Test Result DB Implement
subtitle: Implement test result db and operation
date: 2018-12-08
author: BF
header-img: img/bf/night_02.jpg
catalog: true
tags:
  - python
  - sqlalchemy
  - orm
  - db
---

# 背景

上周提到想要把测试的结果保存到数据库中，下面主要介绍实现的思路。

注意：在定义表名和字段的时候，最好使用小写开头，我下面的例子并不规范。

# 数据结构

最后的数据表结构设计成大致如下：
![TestResultDBDesign](/img/post/2018/12/2018-12-08-TestResultDBDesign.svg)

# [sqlalchemy](https://www.sqlalchemy.org/)

在 Python 中，最有名的 ORM 框架是 SQLAlchemy，它能满足绝大多数对数据库的映射与操作。

网上有许多相关的资料，但是如果不翻墙的话，官网好像访问不了...

下面是使用到的一些类库：

```python
from sqlalchemy import Column, DateTime, Float, String, create_engine, Integer, ForeignKey, exists, exc
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship
```

# TestResult

下面是对测试结果的 TestResult 表的映射，需要关联的是 TestDetail 那张表。

TestResult 是单次测试的总结记录，将测试的总体时间，测试的一些环境信息都保存下来，可以根据自己的需求扩充。

```python
class TestResult(Base):
    __tablename__ = 'TestResult'
    id = Column(Integer, primary_key=True)
    TestTime = Column(DateTime)
    TestVersion = Column(String(40))
    APPVersion = Column(String(100))
    Computer = Column(String(20))
    TotalTimeMinutes = Column(Float)
    CPU = Column(String(20))
    Memory = Column(String(20))
    TestDetails = relationship('TestDetail')
```

# TestCase

每一轮测试中，都会有分为不同的 Test Case，而在多次测试时，许多 Case 都是重复的，所以将相关的信息都保存在这个表中。

因为一般性能测试中，与数据条数有关，所以这边我加了一个 Rows，可以根据实际需求更改与添加字段。

```python
class TestCase(Base):
    __tablename__ = 'TestCase'
    id = Column(Integer, primary_key=True)
    TestCaseName = Column(String(50))
    Rows = Column(Integer)
    TestDetails = relationship('TestDetail')
```

# TestDetail

每次测试中，具体的测试结果便存在 TestDetail 这张表中，针对 case 关注的信息不同，可以扩展相应的字段。

可以看到，我在这边建立了他与TestResult, TestCase的外键关联。

```python
class TestDetail(Base):
    __tablename__ = 'TestDetail'
    id = Column(Integer, primary_key=True)
    CostTimeSeconds = Column(Float)
    CostTimeMinutes = Column(Float)
    TestResultID = Column(Integer, ForeignKey('TestResult.id'))
    TestCaseID = Column(Integer, ForeignKey('TestCase.id'))
```

# TestUtils

下面是设计的一个操作类：

```python
class TestResultUtils():
    def __init__(self, db_sqlnet='xxx', db_name='xxx', db_user='xxx', db_password='xxx'):
        self.engine = create_engine('mssql+pymssql://{0}:{1}@{2}/{3}'.format(db_user, db_password, db_sqlnet, db_name))
        DBSession = sessionmaker(bind=self.engine, autoflush=False)
        self.session = DBSession()

    def save_test_result(self, test_result = None, test_case_list = [], test_detail_list = []):
        count = self.session.query(TestResult).filter(TestResult.TestTime == test_result.TestTime).count()
        if(count > 0):
            test_result = self.session.query(TestResult).filter(TestResult.TestTime == test_result.TestTime).first()

        if(len(test_case_list) != len(test_detail_list)):
            print("test case list and test detail list size is not matched.")
            return

        for index, test_detail in enumerate(test_detail_list):
            test_case = test_case_list[index]
            count = self.session.query(TestCase).filter(TestCase.TestCaseName == test_case_list[index].TestCaseName and TestCase.Rows == test_case_list[index].TestCaseName.Rows).count()
            if(count > 0):
                test_case = self.session.query(TestCase).filter(TestCase.TestCaseName == test_case_list[index].TestCaseName and TestCase.Rows == test_case_list[index].TestCaseName.Rows).first()
            test_case_list[index] = test_case
            test_case.TestDetails.append(test_detail)
            test_result.TestDetails.append(test_detail)

        for test_case in test_case_list:
            self.session.add(test_case)
        self.session.add(test_result)

        try:
            self.session.commit()
            self.session.close()
        except exc.SQLAlchemyError as e:
            print(str(e))
```

# 最后

今天杭州的雪真的挺大的，晚安zzz

---
参考：

- [使用 SQLAlchemy](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0014021031294178f993c85204e4d1b81ab032070641ce5000)
- [cmutils_test.py](https://github.com/bearfly1990/PowerScript/blob/master/Python3/mylib/cmutils_test.py)
- [DB for TestResult](https://bearfly1990.github.io/2018/12/02/TestResultDBDesign/)