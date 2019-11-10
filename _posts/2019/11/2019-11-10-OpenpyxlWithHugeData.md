---
layout: post
title: openpyxl with big data
subtitle: read/write big data with excel by openpyxl
date: 2019-11-10
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - python
  - openpyxl
  - bigdata
  - excel
---

# 背景

最近写了一个脚本，用 openpyxl 从 sql server 数据库中读取数据，使用 template 文件，将数据填充进去，生成最后的 daily report。

使用 openpyxl 来操作 excel，很方便,但发现当要写入大量数据的时候，时间非常慢，而且非常占内存。

最直接的原因是 openpyxl 会将读写过的 cell 都加载在内存中方便后面 update，不会释放这些 cell 而会一直驻留在内存中。

所以如果单纯写的话建议使用 xlrd，而且 pandas 有 to_csv/to_excel 这样的直接的方法。

但是因为我需要使用模板，在研究后发现 openpyxl 有 read_only/write_only 的模式，可以满足我的需求。
(一定要保证安装了 lxml 库，不然就算使用 write_only 模式，也一样会点用大量占存。）

Note：针对 office2007 以后的版本，xlsx 文件上限行数大约为 100 多万条的样子。

# 需求

1. 从数据库中得到想要的数据
2. 读取 template 文件，保留 template 的样式
3. 写入新的 output 文件中

## 数据库中的数据

![2019-11-10-OpenpyxlWithHugeData_DB01.png](/img/post/2019/11/2019-11-10-OpenpyxlWithHugeData_DB01.png)

## Template

![2019-11-10-OpenpyxlWithHugeData_TPL01.png](/img/post/2019/11/2019-11-10-OpenpyxlWithHugeData_TPL01.png)

# 直接使用 oponpyxl

```python
import pandas as pd
import pyodbc
import openpyxl
from datetime import datetime

conn=pyodbc.connect(r'DRIVER={SQL Server};SERVER=PC-CX\SQLEXPRESS;UID=test;PWD=test')

def logger_info(message):
    print(f'{datetime.now()}[INFO]', message)


def query_data_from_db(conn):
    sql_str = 'select UserName, Age, Country, [Status] from [BFTest].[dbo].[Test_Output]'
    sql_query = pd.read_sql_query(sql_str, conn)
    df_output_db = pd.DataFrame(sql_query)
    return df_output_db


def write_to_excel(template, report):
    workbook = openpyxl.load_workbook(template)
    ws_report = workbook['report']
    row_start = 2
    for idx_row, row in df_output_db.iterrows():
        for idx_col in range(len(row)):
            ws_report.cell(column = idx_col+1, row = row_start + idx_row).value = row[idx_col]
    workbook.save(report)


if __name__=='__main__':
    start_time = datetime.now()

    logger_info('query data from db')
    df_output_db = query_data_from_db(conn)

    logger_info('write to excel')
    write_to_excel('template.xlsx', 'report.xlsx')

    end_time = datetime.now()
    logger_info(f'cost time:{end_time-start_time}')
```

最后的结果：
![2019-11-10-OpenpyxlWithHugeData_Report01.png](/img/post/2019/11/2019-11-10-OpenpyxlWithHugeData_Report01.png)

## 增加数据量

现在让我们把数据量加上去一些，再看结果：

```python
def large_data(df):
    df_large = df.copy()
    for i in range(15):
        df = df_large.copy()
        df_large=pd.concat([df_large,df])
    return df_large


if __name__=='__main__':
    start_time = datetime.now()

    logger_info('query data from db')
    df_output_db = query_data_from_db(conn)

    logger_info('large data')
    df_output_db = large_data(df_output_db)
    df_output_db = df_output_db.reset_index(drop=True)
    logger_info(f'count data:{len(df_output_db)}')

    logger_info('write to excel')
    start_time_excel = datetime.now()
    write_to_excel('template.xlsx', 'report.xlsx')
    logger_info(f'write to excel cost time:{datetime.now() - start_time_excel}')

    logger_info(f'cost time:{datetime.now()-start_time}')
```

数据量加到了 131072,一共使用了 26 秒。

```
PS C:\Users\mayn\Desktop\GitSpace\PowerScript\Python3\openpyxl\generate_report> python .\generate_report.hugedata.py
2019-11-10 10:50:02.332573[INFO] query data from db
2019-11-10 10:50:02.334600[INFO] large data
2019-11-10 10:50:02.370501[INFO] count data:131072
2019-11-10 10:50:02.370501[INFO] write to excel
2019-11-10 10:50:28.605598[INFO] write to excel cost time:0:00:26.234127
2019-11-10 10:50:28.606598[INFO] cost time:0:00:26.274025
```

# 使用 write_only

先写 template 的表头，再写内容：

```python

def write_to_excel(template, report):
    wb_tpl = openpyxl.load_workbook(template)
    ws_report_tpl = wb_tpl['report'] #workbook.get_sheet_by_name('report')
    workbook = openpyxl.Workbook(write_only=True)
    ws_report = workbook.create_sheet('report')
    for row in ws_report_tpl.rows:
        row_tpl = []
        for cell in row:
            cell_tpl = openpyxl.cell.WriteOnlyCell(ws_report)
            cell_tpl.value = cell.value
            cell_tpl.font = cell.font.copy()
            cell_tpl.fill = cell.fill.copy()
            row_tpl.append(cell_tpl)
        ws_report.append(row_tpl)
    for idx_row, row in df_output_db.iterrows():
        if idx_row % 10000 == 0:
            print(indx_row)
        ws_report.append(row.to_list())
    workbook.save(report)

if __name__=='__main__':
    start_time = datetime.now()

    logger_info('query data from db')
    df_output_db = query_data_from_db(conn)

    logger_info('large data')
    df_output_db = large_data(df_output_db)
    df_output_db = df_output_db.reset_index(drop=True)
    logger_info(f'count data:{len(df_output_db)}')

    logger_info('write to excel')
    start_time_excel = datetime.now()
    write_to_excel('template.xlsx', 'report.xlsx')
    logger_info(f'write to excel cost time:{datetime.now() - start_time_excel}')

    logger_info(f'cost time:{datetime.now()-start_time}')
```

```
PS C:\Users\mayn\Desktop\GitSpace\PowerScript\Python3\openpyxl\generate_report> python .\generate_report.hugedata.writeonly.py
2019-11-10 10:54:58.253268[INFO] query data from db
2019-11-10 10:54:58.255265[INFO] large data
2019-11-10 10:54:58.292595[INFO] count data:131072
2019-11-10 10:54:58.292595[INFO] write to excel
2019-11-10 10:55:18.039439[INFO] write to excel cost time:0:00:19.745817
2019-11-10 10:55:18.039439[INFO] cost time:0:00:19.786171
```

可以看到时间花费上普通用法比 write_only 慢了 36%左右，在数据量 volumn 更大的时候更明显，内存上也使用更不用说。

完整的代码在[demo04_compare_csv](https://github.com/bearfly1990/PowerScript/tree/master/Python3/openpyxl/generate_report)
