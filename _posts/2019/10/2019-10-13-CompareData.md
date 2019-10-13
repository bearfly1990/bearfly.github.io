---
layout: post
title: Compare data by pandas
subtitle: Compare data in two excel files
date: 2019-10-13
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - python
  - pandas
  - excel
  - db
---

# 背景

最近的项目会根据一些数据的值来得到一定的结果，用户原来使用 python 直接生成 excel 文件，我们相当于移植到数据库中，使用 SP 去做业务逻辑。

所以理论上最后的结果需要一致，之前没有做全部数据的对比，这次相关 features 由我在改动，所以无论如何还是要保证数据能对比上，所以就写了相关的脚本来处理。

# 需求

情况是这样的，假设：

1. User 有一份 output 文件(output_user.xlsx)，数据库中有我们处理后对应的 output 数据。
2. User 的 output 文件是有多余的数据，我们在数据库中已经剔除，所以对比前需要把 output 文件的冗余数据剔除掉。
3. 需要把数据库中我们的 output 数据也输出到文件中(output_db.xlsx)
4. 将 output_user.xlsx 与 output_db.xlsx 中的需要对比的字段进行比较，最后将不同的数据行出来到 compare_result.xlsx 文件中。

# 假设

User Output 文件中的 output_user.xlsx 数据为：

| id  | UserName | Age | Country | Status  |
| --- | -------- | --- | ------- | ------- |
| 1   | cx       | 29  | China   | Success |
| 2   | xm       | 27  | China   | Pending |
| 3   | ll       | 30  | China   | Failed  |
| 4   | zz       | 20  | China   | Success |
| 5   | yy       | 21  | USA     | Success |
| 5   | yy       | -21 | USA     | Success |
| 6   | xx       | -21 | USA     | Success |

数据库中的数据为：

| id  | UserName | Age | Country | Status  |
| --- | -------- | --- | ------- | ------- |
| 1   | cx       | 29  | China   | Success |
| 2   | xm       | 27  | China   | Success |
| 3   | ll       | 30  | China   | Failed  |
| 4   | zz       | 20  | China   | Pending |

剔除的 logic:

- id 不能重复
- Age 不能为负数(value<0)

所以在 output_user.xlsx 中 id 为 5，6 的需要被去除

# step01 剔除冗余数据

```python
import pandas as pd

def get_exception_indexes(df):
    indexes_removed = set()
    df_age = df[df['Age'].apply(lambda x: x < 0 or str(x) in ('', 'nan'))]['Age']
    indexes_removed.update(list(df_age.index.values))
    df_duplicated_ids = df[df.duplicated(subset=['id'], keep=False)]
    df_duplicated_group = df_duplicated_ids.groupby('id')
    for id in df_duplicated_group.groups.keys():
        indexes_removed.update(df_duplicated_group.groups[id])
    return list(indexes_removed)

if __name__=='__main__':
    df_output_user = pd.read_excel('output_user.xlsx', sheet_name='Data')
    index_removed = get_exception_indexes(df_output_user)

    df_output_user_removed = df_output_user.iloc[index_removed]
    df_output_user_keeped = df_output_user.drop(index=index_removed)

    writer = pd.ExcelWriter('output_user.removed.xlsx', engine='openpyxl')
    df_output_user_keeped.to_excel(writer, sheet_name='Data', index=False)
    df_output_user_removed.to_excel(writer, sheet_name='Removed', index=False)
    writer.save()
```
得到结果：

| id  | UserName | Age | Country | Status  |
| --- | -------- | --- | ------- | ------- |
| 1   | cx       | 29  | China   | Success |
| 2   | xm       | 27  | China   | Pending |
| 3   | ll       | 30  | China   | Failed  |
| 4   | zz       | 20  | China   | Success |

# step02 从数据库中导出到Excel中

```python
import pandas as pd

if __name__=='__main__':
    conn=pyodbc.connect(r'DRIVER={SQL Server};SERVER=PC-CX\SQLEXPRESS;UID=test;PWD=test')
    sql_str = 'select * from [BFTest].[dbo].[Test_Output]'

    sql_query = pd.read_sql_query(sql_str, conn)
    df_output_db = pd.DataFrame(sql_query)

    writer = pd.ExcelWriter('output_db.xlsx', engine='openpyxl')
    df_output_db.to_excel(writer, sheet_name='Data', index=False)
    writer.save()
```

# step03 对比数据

最后将没有匹配的数据分别放到sheet User和DB中。
```python
import pandas as pd
import pyodbc

def get_key_value(x):
    key_value = [x['id'], x['UserName'], x['Country'], x['Status']]
    return ','.join([str(x).replace(' ', '').lower() for x in key_value])

if __name__=='__main__':

    df_output_user = pd.read_excel('output_user.removed.xlsx', sheet_name='Data')
    df_output_db = pd.read_excel('output_db.xlsx', sheet_name='Data')

    df_output_user['key_value'] = df_output_user.apply(lambda x: get_key_value(x), axis=1)
    df_output_user['source'] = 'User'
    df_user_compare = df_output_user[['key_value', 'source']]

    df_output_db['key_value'] = df_output_db.apply(lambda x: get_key_value(x), axis=1)
    df_output_db['source'] = 'DB'
    df_db_compare = df_output_db[['key_value', 'source']]

    df_compare = pd.concat([df_user_compare, df_db_compare])
    df_compare = df_compare.drop_duplicates(subset=['key_value'], keep=False)

    indexes_user = df_compare.index[df_compare['source'] == 'User'].tolist()
    indexes_db = df_compare.index[df_compare['source'] == 'DB'].tolist()

    df_output_user = df_output_user.iloc[indexes_user]
    df_output_db = df_output_db.iloc[indexes_db]
    writer = pd.ExcelWriter('compare_result.xlsx', engine='openpyxl')
    df_output_user.to_excel(writer, sheet_name='User', index=False)
    df_output_db.to_excel(writer, sheet_name='DB', index=False)
    writer.save()
```

最后的代码在[demo03_get_data_from_db](https://github.com/bearfly1990/PowerScript/tree/master/Python3/pandas/demo03_get_data_from_db)


