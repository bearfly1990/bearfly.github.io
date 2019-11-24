---
layout: post
title: 字符串相似度
subtitle: 比较两个字符串之间的相似程度
date: 2019-11-24
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - python
  -
---

# 背景

在工作中有需要从 comment 中提取所需要的值，是通过前缀来判断的，由于注释是人为输入的，所以很多时候会有一些拼写错误。
目前是通过写死前缀的字符串依次遍历来达到目的。

比如我们要的是 as of date mm/dd/yyyy，但也想要接受 as of data mm/dd/yyyy 那这样的话，就可以兼容许多拼写的错误。

字符串的相似性比较应用场合很多，像拼写纠错、文本去重、上下文相似性等。

评价字符串相似度最常见的办法就是：把一个字符串通过插入、删除或替换这样的编辑操作，变成另外一个字符串，所需要的最少编辑次数，这种就是编辑距离（edit distance）度量方法，也称为**Levenshtein距离**。
海明距离是编辑距离的一种特殊情况，只计算等长情况下替换操作的编辑次数，只能应用于两个等长字符串间的距离度量。

其他常用的度量方法还有 Jaccard distance、J-W距离（Jaro–Winkler distance）、余弦相似性（cosine similarity）、欧氏距离（Euclidean distance）等。

# difflib
python 有内置的difflib来判断相似度，非常方便。
```python
import difflib
query_str = 'as of date 11/12/2019'
s1 = 'as of data 11/12/2019'
s2 = 'as fo date 11/12/2019'
s3 = 'similar date 11/12/2019'
print(difflib.SequenceMatcher(None, query_str, s1).quick_ratio())  
print(difflib.SequenceMatcher(None, query_str, s2).quick_ratio())  
print(difflib.SequenceMatcher(None, query_str, s3).quick_ratio()) 
```

```
PS C:\Users\mayn\Desktop> python .\test.py
0.9523809523809523
1.0
0.8181818181818182
```


## 数据库中的数据


参考：
- ![Python 字符串相似性的几种度量方法](https://blog.csdn.net/dcrmg/article/details/79228589)
- ![Python 字符串相似性的几种度量方法](https://blog.csdn.net/dcrmg/article/details/79228589)
