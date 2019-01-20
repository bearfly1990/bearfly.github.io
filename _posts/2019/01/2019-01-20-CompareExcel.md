---
layout: post
title: Compare Excel
subtitle: 简单对比excel
date: 2019-01-20
author: BF
header-img: img/bf/python_01.jpg
catalog: true
tags:
  - python
  - openpyxl
  - excel
---

# 背景

下周可能有一个小任务是由于系统升级，需要保证生成的 excel 是一致的，所以写了一个非常简单的对比脚本。

# 主要代码

下面直接上代码，主要的思路就是从上到下依次对比，从 sheet name 开始，最后对比 cell 的值，数据类型，数据格式。

```python
"""
author: xiche
create at: 01/20/2019
description:
    simple script to compare excel
Change log:
Date        Author      Version    Description
01/20/2019   xiche      1.0        Init
"""
import openpyxl

def compare_cell(cell01, cell02):
    if(cell01.value != cell02.value):
        print('{}{}: {} / {}'.format(cell01.column, cell01.row, cell01.value, cell02.value))
    elif(cell01.data_type != cell02.data_type):
        print('{}{}: {} / {}'.format(cell01.column, cell01.row, cell01.data_type, cell02.data_type))
    elif(cell01.number_format != cell02.number_format):
        print('{}{}: {} / {}'.format(cell01.column, cell01.row, cell01.number_format, cell02.number_format))

def compare_sheet(sheet01, sheet02):

    if(sheet01.max_row != sheet02.max_row):
        print('row is not matched! {} / {}'.format(sheet01.max_row, sheet02.max_row))

    if(sheet01.max_column != sheet02.max_column):
        print('column is not matched! {} / {}'.format(sheet01.max_column, sheet02.max_column))

    max_row = max(sheet01.max_row, sheet02.max_row)
    max_column = max(sheet01.max_column, sheet02.max_column)

    for index_row in range(max_row):
        for index_col in range(max_column):
            compare_cell(sheet01.cell(column=index_col+1, row=index_row+1), sheet02.cell(column=index_col+1, row=index_row+1))

def test_compare_same_excel(excel_file01, excel_file02):
    print('------compare {} & {}------'.format(excel_file01, excel_file02))
    wb01 = openpyxl.load_workbook(excel_file01)
    wb02 = openpyxl.load_workbook(excel_file02)

    sheet_names_01 = wb01.get_sheet_names()
    sheet_names_02 = wb02.get_sheet_names()

    if(len(sheet_names_01) != len(sheet_names_02)):
        print('sheets number are not matched!')
        return

    for i in range(len(sheet_names_01)):
        if(len(sheet_names_01[i]) != len(sheet_names_02[i])):
            print('sheets name not matched!\n{}:{} / {}:{}'.format(excel_file01, sheet_names_01[i], excel_file02, sheet_names_02[i]))
            return
        sheet01 = wb01.get_sheet_by_name(sheet_names_01[i])
        sheet02 = wb02.get_sheet_by_name(sheet_names_02[i])

        print('compare {}:{} & {}:{}'.format(excel_file01, sheet01.title, excel_file02, sheet02.title))
        compare_sheet(sheet01, sheet02)

if __name__ == '__main__':
    test_compare_same_excel('test01.xlsx', 'test01.copy.xlsx')
    test_compare_same_excel('test01.xlsx', 'test03.xlsx')
    test_compare_same_excel('test01.xlsx', 'test03.xlsx')
    test_compare_same_excel('test01.xlsx', 'test02.xlsx')
    test_compare_same_excel('test01.xlsx', 'test04.xlsx')
```

目前的输出结果如下：

```powershell
C:\Users\mayn\Desktop\GitSpace\PowerScript\Python3\openpyxl\compare_excels>python compare_excels.py
------compare test01.xlsx & test01.copy.xlsx------
compare test01.xlsx:Sheet1 & test01.copy.xlsx:Sheet1
------compare test01.xlsx & test03.xlsx------
sheets name not matched!
test01.xlsx:Sheet1 / test03.xlsx:NotMatch
------compare test01.xlsx & test03.xlsx------
sheets name not matched!
test01.xlsx:Sheet1 / test03.xlsx:NotMatch
------compare test01.xlsx & test02.xlsx------
compare test01.xlsx:Sheet1 & test02.xlsx:Sheet1
row is not matched! 4 / 5
B2: mm-dd-yy / [$-409]d\-mmm\-yy;@
C4: Test03 / Test04
B5: None / 10/21/2010
C5: None / Test05
------compare test01.xlsx & test04.xlsx------
compare test01.xlsx:Sheet1 & test04.xlsx:sheet1
row is not matched! 4 / 6
column is not matched! 3 / 4
B2: mm-dd-yy / [$-409]d\-mmm\-yy;@
C4: Test03 / Test04
B5: None / 10/21/2010
C5: None / Test05
B6: None / test
C6: None / test
D6: None / test
```

# 最后

不知道有没有第三方库直接提供对比的方法，自己的这个之后可以添加到自己的工具库中。

代码和测试的 excel 在：[compare_excels](https://github.com/bearfly1990/PowerScript/tree/master/Python3/openpyxl/compare_excels)
