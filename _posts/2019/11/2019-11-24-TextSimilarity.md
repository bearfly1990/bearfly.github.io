---
layout: post
title: Text Similarity
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

评价字符串相似度最常见的办法就是：把一个字符串通过插入、删除或替换这样的编辑操作，变成另外一个字符串，所需要的最少编辑次数，这种就是编辑距离（edit distance）度量方法，也称为**Levenshtein 距离**。
海明距离是编辑距离的一种特殊情况，只计算等长情况下替换操作的编辑次数，只能应用于两个等长字符串间的距离度量。

其他常用的度量方法还有 Jaccard distance、J-W 距离（Jaro–Winkler distance）、余弦相似性（cosine similarity）、欧氏距离（Euclidean distance）等。

# difflib

python 有内置的 difflib 来判断相似度，非常方便。

```python
import difflib
query_str = 'as of date 11/12/2019'
s1 = 'as of data 11/12/2019'
s2 = 'as fo date 11/12/2019'
s3 = 'similar date 11/12/2019'
print(difflib.SequenceMatcher(None, query_str, s1).quick_ratio()) # 0.9523809523809523

print(difflib.SequenceMatcher(None, query_str, s2).quick_ratio()) # 1.0
print(difflib.SequenceMatcher(None, query_str, s3).quick_ratio()) # 0.8181818181818182

"""
第一个参数是想要忽略的字符，可以不算在其中。
seq = difflib.SequenceMatcher(lambda x:x=" ", a, b)
"""
```

# fuzzywuzzy

fuzzywuzzy 是一个第三方库，提供了更多的方法进行不同的字符匹配需求。

可以看到 fuzzywuzzy 的默认匹配区分度更直观点。

```python
from fuzzywuzzy import fuzz
query_str = 'as of date 11/12/2019'
s1 = 'as of data 11/12/2019'
s2 = 'as fo date 11/12/2019'
s3 = 'similar date 11/12/2019'

print (fuzz.ratio(query_str, s1)) #95

print (fuzz.ratio(query_str, s2)) #95

print (fuzz.ratio(query_str, s3)) #77
```

除些外，还有 partial_ratio(), token_set_ratio(),partial_token_sort_ratio()等方法。


# Levenshtein

```python
import Levenshtein
query_str = 'as of date 11/12/2019'
s1 = 'as of data 11/12/2019'
s2 = 'as fo date 11/12/2019'
s3 = 'similar date 11/12/2019'

# hamming距离，str1和str2长度必须一致，描述两个等长字串之间对应位置上不同字符的个数

print(Levenshtein.hamming(query_str, s1)) # 1

print(Levenshtein.hamming(query_str, s2)) # 2

#print(Levenshtein.hamming(query_str, s3)) #ValueError: hamming expected two unicodes of the same length


# 编辑距离，描述由一个字串转化成另一个字串最少的操作次数，在其中的操作包括 插入、删除、替换

print(Levenshtein.distance(query_str, s1)) # 1

print(Levenshtein.distance(query_str, s2)) # 2

print(Levenshtein.distance(query_str, s3)) # 7


# 计算莱文斯坦比

print(Levenshtein.ratio(query_str, s1)) # 0.9523809523809523

print(Levenshtein.ratio(query_str, s2)) # 0.9523809523809523

print(Levenshtein.ratio(query_str, s3)) # 0.7727272727272727


# 计算jaro距离

print(Levenshtein.jaro(query_str, s1)) # 0.9682539682539683

print(Levenshtein.jaro(query_str, s2)) # 0.9841269841269842

print(Levenshtein.jaro(query_str, s3)) # 0.8151023694501954


# Jaro–Winkler距离

print(Levenshtein.jaro_winkler(query_str, s1)) # 0.9968253968253968

print(Levenshtein.jaro_winkler(query_str, s2)) # 0.9888888888888889

print(Levenshtein.jaro_winkler(query_str, s3)) # 0.8151023694501954

```


# 最后

回过头来说，使用字符串匹配度来看字符串是不是我们想要的，还是有误差的，理论上还是用正则, 或者说固定的字符更精准。

参考：

- [Python 字符串相似性的几种度量方法](https://blog.csdn.net/dcrmg/article/details/79228589)
- [python比较字符串相似度](https://blog.csdn.net/qq_41020281/article/details/82194992)
- [python: fuzzywuzzy学习笔记](https://blog.csdn.net/qq_43174128/article/details/82595317)