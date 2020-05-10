---
layout: post
title: Pandas Data Concat
subtitle: Data Concat Append Merge
date: 2020-05-10
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - Pandas
  - Python
---

# 背景

最近生成的报表比原来的逻辑多了些字段，想要老的报表也加上，所以用 python 写了脚本去处理，需要用来一些 mapping 的数据。pandas 有许多合并数据的方法，Concat 可以横向与纵向的合并，merge 可以实现横向的合并类似 sql 中的 join，append 则是纵向的合并。

# 样例

以下面简单的例子为例,有下面三个文件， 想要数据合并到一起。
csv-Student

| id  | name |
| --- | ---- |
| 1   | CX   |
| 2   | XM   |
| 3   | ZS   |
| 5   | BF   |

csv-Score

| SID | Class | Score |
| --- | ----- | ----- |
| 1   | 语文  | 70    |
| 1   | 数学  | 80    |
| 1   | 英文  | 68    |
| 2   | 语文  | 87    |
| 2   | 数学  | 98    |
| 2   | 英文  | 56    |
| 3   | 语文  | 12    |
| 3   | 数学  | 43    |
| 3   | 英文  | 76    |
| 4   | 语文  | 66    |
| 4   | 数学  | 77    |

csv-Score2
| SID | Class | Score |
| --- | ----- | ----- |
| 5 | 语文 | 77 |
| 5 | 数学 | 88 |
| 5 | 英文 | 66 |

# 实现

Concat 支持两个方向的合并，所以我们可以先把 Score 合并在一起，然后再 left out join Student

```python
def concat_student_score(file_student, file_score, file_score2):
    df_stu = pd.read_csv(file_student)
    df_score1 = pd.read_csv(file_score)
    df_score2 = pd.read_csv(file_score2)
    df_score = pd.concat([df_score1, df_score2], ignore_index=True)
    # df_score = df_score1.append(df_score2, ignore_index=True)
    df_merged = pd.merge(df_score, df_stu, how='outer', on=['SID'])
    print(df_merged)
```
|     | SID | Class | Score | Name |
| --- | --- | ----- | ----- | ---- |
| 0   | 1   | 语文  | 70    | CX   |
| 1   | 1   | 数学  | 80    | CX   |
| 2   | 1   | 英文  | 68    | CX   |
| 3   | 2   | 语文  | 87    | XM   |
| 4   | 2   | 数学  | 98    | XM   |
| 5   | 2   | 英文  | 56    | XM   |
| 6   | 3   | 语文  | 12    | ZS   |
| 7   | 3   | 数学  | 43    | ZS   |
| 8   | 3   | 英文  | 76    | ZS   |
| 9   | 4   | 语文  | 66    | NaN  |
| 10  | 4   | 数学  | 77    | NaN  |
| 11  | 4   | 英文  | 88    | NaN  |
| 12  | 5   | 语文  | 77    | BF   |
| 13  | 5   | 数学  | 88    | BF   |
| 14  | 5   | 英文  | 66    | BF   |

以上是最简单的应用，还有许多的参数可以控制合并的细节，可以参考下面的资料，尤其是最后的官方的与sql的对比应用，大部分sql能实现的，pandas都可以

[demo05](https://github.com/bearfly1990/PowerScript/tree/master/Python3/pandas/demo05_merge_df)

# 参考

- [python3：pandas（合并concat和merge）](https://blog.csdn.net/sunshine_lyn/article/details/81535529)
- [PANDAS 数据合并与重塑（join/merge篇）](https://www.cnblogs.com/bigshow1949/p/7016235.html)
- [Comparison with SQL](https://pandas.pydata.org/pandas-docs/stable/getting_started/comparison/comparison_with_sql.html)