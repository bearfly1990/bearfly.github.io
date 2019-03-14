---
layout: post
title: Update CSV Column Values
subtitle: Update all data in one column on csv
date: 2019-03-14
author: BF
header-img: img/bf/python_02.jpg
catalog: true
tags:
  - python
  - csv
---

# 背景

在一些 csv 文件中会有日期类型，而如果我们想按日期类型导入到数据库中，那么就需要在 insert 前以 date(datetime)的类型传入。

假设简单的 csv 文件(sample.csv)如下：

```csv
HolidayDate,Region
2019-01-01,China
2019-01-02,China
2019-04-01,China
2019-05-01,China
2019-07-07,China
2019-10-01,China
2019-01-05,US
2019-06-01,US
```

## 直接使用 cvs 库

读取到 csv 后，直接遍历完成：

```python
import csv
from datetime import datetime

def read_csv_rows_list(file_path):
    csv_row_list = []
    with open(file_path, newline='') as csvfile:
        csvReader = csv.reader(csvfile, delimiter=',', quotechar='|')
        for row in csvReader:
            csv_row_list.append(row)
    return csv_row_list

csv_row_list = read_csv_rows_list('sample.csv')
csv_row_list = csv_row_list[1:]
for row in csv_row_list:
    row[0] = datetime.strptime(row[0], '%Y-%m-%d')
[print(row) for row in csv_row_list]
```

output:

```bash
[datetime.datetime(2019, 1, 1, 0, 0), 'China']
[datetime.datetime(2019, 1, 2, 0, 0), 'China']
[datetime.datetime(2019, 4, 1, 0, 0), 'China']
[datetime.datetime(2019, 5, 1, 0, 0), 'China']
[datetime.datetime(2019, 7, 7, 0, 0), 'China']
[datetime.datetime(2019, 10, 1, 0, 0), 'China']
[datetime.datetime(2019, 1, 5, 0, 0), 'US']
[datetime.datetime(2019, 6, 1, 0, 0), 'US']
```

## 使用 pandas

如果使用 pandas 就很方便：

```python
import pandas as pd
import numpy as np

data = pd.read_csv('sample.csv',encoding='utf-8',)
data[u'HolidayDate'] = pd.to_datetime(data[u'HolidayDate'])
# data[u'HolidayDate'].astype(str)
# data[u'HolidayDate'] = data[u'HolidayDate'].apply(lambda x :datetime.strptime(x, '%Y-%m-%d'))
data = np.array(data)#np.ndarray()
data_list = data.tolist()
[print(row) for row in data_list]
#data.to_csv('sample.output.csv',index=False, encoding='utf-8')
```

```bash
[Timestamp('2019-01-01 00:00:00'), 'China']
[Timestamp('2019-01-02 00:00:00'), 'China']
[Timestamp('2019-04-01 00:00:00'), 'China']
[Timestamp('2019-05-01 00:00:00'), 'China']
[Timestamp('2019-07-07 00:00:00'), 'China']
[Timestamp('2019-10-01 00:00:00'), 'China']
[Timestamp('2019-01-05 00:00:00'), 'US']
[Timestamp('2019-06-01 00:00:00'), 'US']
```

---

参考：

- [pandas DataFrame数据转为list](https://blog.csdn.net/xiangxianghehe/article/details/72615711)
- [pandas处理时间和日期类型数据](https://www.jianshu.com/p/325ffa92654a)
- [Pandas修改csv文件某一列的值](https://blog.csdn.net/okm6666/article/details/81003397)
