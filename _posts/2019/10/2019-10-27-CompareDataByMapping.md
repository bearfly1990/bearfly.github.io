---
layout: post
title: Compare csv file
subtitle: Compare csv column values by mapping file
date: 2019-10-27
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - python
  - pandas
  - csv
---

# 背景

最近有个需求是对比 CSV 文件，其中一个为 Source， 另一个是系统生成的，理论上要满足 mapping 关系。

现在想要用 python 来验证一下生成的文件对应关系是正确的。

# 需求

情况如下：

1. source_old.csv 与 source_new.csv 为需要对面的文件, 需要对比的字段为 csv 中的 Country 与 City

2. Country 的 mapping 关系在 mapping1.txt 中，City 的 mapping 关系在 mapping2.txt 中，使用逗号相隔。

3. mapping_info.txt 中存储了 mapping 关系文件与对应的 csv 中字段的关系

## CSV 文件

source_old.csv

```csv
ID,Name,Country,City
1,Name1,Japan,HangZhou
2,Name2,China,HangZhou
3,Name3,America,Other
4,Name4,TaiWan,ShangHai
5,Name5,English,TaiZou
```

source_new.csv

```csv
ID,Name,Country,City
1,Name1,JP,HZ
2,Name2,CN,Other
3,Name3,US,Other
4,Name4,TaiWan,SH
5,Name5,US,TZ
```

## mapping 文件

mapping1.txt

```
China,CN
America,USA
English,US
Japan,JP
```

mapping2.txt

```
HangZhou,HZ
ShangHai,SH
TaiZou,TZ
```

## mapping info

```
Country,mapping1.txt
City,mapping2.txt
```

# 思路

## 读取 mapping 文件

对于 mapping 文件，先定义了一个通用的方法读取构造 dictionary.

```python
def read_mapping(file):
    mapping = {}
    with open(file, 'r') as f:
        rows = f.readlines()
    for row in rows:
        key, value = row.split(',')
        mapping[key.strip()] = value.strip()
    return mapping
```

## 对比

传入需要对比的 csv 文件及对应的 mapping 和字段，输出最后的结果

```python
def compare_csv(csv1_path, csv2_path, mapping, column):
    df_csv1=pd.read_csv(csv1_path)
    df_csv2=pd.read_csv(csv2_path)

    for index,row in df_csv1.iterrows():
        old_value = row[column]
        if old_value in mapping:
            expect_value = mapping.get(old_value)
            new_value = df_csv2.loc[index,column]
            if expect_value != new_value:
                print(f'File-<{csv2_path}> Col-<{column}> Row-<{index+1}> should be <{expect_value}>, but <{new_value}> found')
        else:
            print(f'File-<{csv1_path}> Col-<{column}> Row-<{index+1}> value-<{old_value}> is not in the mapping file')
```

##main 函数

先读取 mapping 文件的信息，再循环遍历，得到最后的结果：

```
if __name__=='__main__':
    mapping_info = read_mapping('mapping_info.txt')

    for column in mapping_info.keys():
        mapping_file = mapping_info[column]
        column_mapping = read_mapping(mapping_file)
        compare_csv('source_old.csv', 'source_new.csv', column_mapping, column)
```

## 输出

```console
File-<source_new.csv> Col-<Country> Row-<3> should be <USA>, but <US> found
File-<source_old.csv> Col-<Country> Row-<4> value-<TaiWan> is not in the mapping file
File-<source_new.csv> Col-<City> Row-<2> should be <HZ>, but <Other> found
File-<source_old.csv> Col-<City> Row-<3> value-<Other> is not in the mapping file
```

完整的代码在[demo04_compare_csv](https://github.com/bearfly1990/PowerScript/tree/master/Python3/pandas/demo04_compare_csv)
