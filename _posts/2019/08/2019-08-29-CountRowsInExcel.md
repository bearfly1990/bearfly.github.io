---
layout: post
title: Count data rows in excel
subtitle: Calcuate data rows for excels in the same folder
date: 2019-08-29
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - python
  - pandas
  - excel
---

# 背景

今天手动给用户提供了 Report，其中需要对 excel 中的数据与数据库中的进行对比。数据文件不少，我要一个个打开去数，真的很费时间。

想要写一个小脚本，但发现没那么快，还是先手动给做了，写一个还是挺快的。

# 需求

情况是这样的，假设： 
1. 都是有效的文件 
2. 有多个 excel 文件在同一个目录中 
3. 有 xls 和 xlsx 后缀的 
4. 统计每个文件的文件名，Sheet，及 Sheet 中有多少有效数据 
5. 将结果写到 output 文件中

最后的代码在[这里
](https://github.com/bearfly1990/PowerScript/tree/master/Python3/pandas/demo02_get_data_rows)
# 读取一个 excel，并输出相关信息

需求我们一个个实现，首先，读取一个 excel，遍历每个 sheet，得到数据的条数。

下面是一个简单的样例,可以得到想要的结果

```python
import pandas as pd

df_sheet_map = pd.read_excel("./Test1.xlsx", None)

sheets = list(df_sheet_map.keys())

for sheet in sheets:
    df_sheet = df_sheet_map[sheet]
    print(sheet, len(df_sheet.index))
```

output:

```
Sheet1 6
Test 4
Demo 7
```

# 遍历目录下所有的 excel 文件

可以使用 glob 去遍历目录及子目录下，所有后缀为 xls 或者 xlsx 的文件，再直接读取：

```python
import pandas as pd
import glob

files = glob.glob('**/*.xlsx', recursive=True)
files = files + glob.glob('**/*.xls', recursive=True)
print(files)
for file in files:
    print(file)
    df_sheet_map = pd.read_excel(file, None)
    sheets = list(df_sheet_map.keys())
    for sheet in sheets:
        df_sheet = df_sheet_map[sheet]
        print('--',sheet, ':', len(df_sheet.index))
```

output:

```
['Test1.xlsx', 'Test2.xlsx', 'Test3.xls', 'sub_folder\\Test4.xls']
Test1.xlsx
-- Sheet1 : 6
-- Test : 4
-- Demo : 7
Test2.xlsx
-- Sheet1 : 16
-- Test : 8
-- Demo : 11
Test3.xls
-- Sheet1 : 10
-- Test : 5
-- Demo : 8
sub_folder\Test4.xls
-- Sheet1 : 10
-- Test : 5
-- Demo : 8
```

# 将结果写到 output 文件中

信息都有了，想要写到文件中，最简单写到 txt，这里我们用了 pandas，就直接写到 excel 中好了。

```python
import pandas as pd
import glob


dict_output = {}
dict_output['fileName'] = []
dict_output['sheet'] = []
dict_output['rows'] = []

files = glob.glob('**/*.xlsx', recursive=True)
files = files + glob.glob('**/*.xls', recursive=True)
print(files)
for file in files:
    print(file)
    df_sheet_map = pd.read_excel(file, None)
    sheets = list(df_sheet_map.keys())

    for sheet in sheets:
        df_sheet = df_sheet_map[sheet]
        print('--',sheet, ':', len(df_sheet.index))
        dict_output['fileName'].append(file)
        dict_output['sheet'].append(sheet)
        dict_output['rows'].append(len(df_sheet.index))

df_output = pd.DataFrame(dict_output)

df_output.to_excel("output.xlsx", sheet_name='details', index=False)

# with pd.ExcelWriter('output.xlsx') as writer:  # doctest: +SKIP
#     ...     df1.to_excel(writer, sheet_name='Sheet_name_1')
#     df2.to_excel(writer, sheet_name='Sheet_name_2'
```
output in excel:

| fileName             | sheet   | rows |
| :------------------- | :------ | ---: |
| Test1.xlsx           | Sheet1  |    6 |
| Test1.xlsx           | Test    |    4 |
| Test1.xlsx           | Demo    |    7 |
| Test2.xlsx           | Sheet1  |   16 |
| Test2.xlsx           | Test    |    8 |
| Test2.xlsx           | Demo    |   11 |
| Test3.xls            | Sheet1  |   10 |
| Test3.xls            | Test    |    5 |
| Test3.xls            | Demo    |    8 |
| sub_folder\Test4.xls | Sheet1  |   10 |
| sub_folder\Test4.xls | Test    |    5 |
| sub_folder\Test4.xls | Demo    |    8 |

参考:

- [python 获取 excel 文件的所有 sheet 名字](https://www.cnblogs.com/qingyuanjushi/p/8449151.html)
- [pandas DataFrame的创建方法](https://www.cnblogs.com/datasnail/p/9675410.html)