---
layout: post
title: Analyze 1.usa.gove from bit.ly
subtitle: Python for Data Analysis Learn 01
date: 2019-01-05
author: BF
header-img: img/bf/snow_01.jpg
catalog: true
tags:
  - python
  - data analysis
  - pandas
  - numpy
  - matplotlib
---

# 背景

最近在看数据分析相关的知识点，同事那借了本 Python for Data Analysis 在看，接下来会记录一下学习心得和书上的例子，温故知新。

数据分析的门道还是挺多的，Python 的一些库(pandas, numpy, matplotlib)也很好用，不用自己去用标准库辛苦的写了，之前分析 log 如果了解这些知识点的话，效率会很高。

今天介绍书中的例子。

# 数据源

今天分析的数据是 URL 缩短服务 bit.ly 与美国政府网站 usa.gov 合作，从生成.gov 或.mil 短链接的用户那里收集来的匿名数据（该服务已停止）。

我们可以看到下面的文档例子，里面每一行都是 json 数据。

[example.txt](https://github.com/bearfly1990/pydata-book/blob/2nd-edition/datasets/bitly_usagov/example.txt)

下面是其中一行数据的例子：

```js
{
	'a': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.78 Safari/535.11',
	'c': 'US',
	'nk': 1,
	'tz': 'America/New_York',
	'gr': 'MA',
	'g': 'A6qOVH',
	'h': 'wfLQtf',
	'l': 'orofrog',
	'al': 'en-US,en;q=0.8',
	'hh': '1.usa.gov',
	'r': 'http://www.facebook.com/l/7AQEFzjSi/1.usa.gov/wfLQtf',
	'u': 'http://www.ncbi.nlm.nih.gov/pubmed/22415991',
	't': 1331923247,
	'hc': 1331822918,
	'cy': 'Danvers',
	'll': [42.576698, -70.954903]
}
```

像 `tz` 就是表示 time zone, `a` 则是用户浏览器的信息 agent，主要分析的数据便是他们，我们一步一步来看。

下面是对数据的初步处理，将文件读取进来，加入 Dict

```python
import json
import pandas as pd; import numpy as np
import matplotlib.pyplot as plt
from pandas import DataFrame, Series

example_path = 'example.txt'

records = [json.loads(line) for line in open(example_path)]
```

# 对时区(tz)进行计数

这里我们就用到了 pandas 中的最重要的数据结构 DataFrame, 它用于将数据表示为一个表格。

下面的代码就是将之前读取的数据创建成 DataFrame，然后通过`.value_counts()`方法非常容易的对时区进行了计数, 并且也进行了排序，所以我们取前面 10 个便是 top 10。

```python
frame = DataFrame(records)
tz_counts = frame['tz'].value_counts()
print(tz_counts[:10])
```

output:

```bash
America/New_York       1251
                        521
America/Chicago         400
America/Los_Angeles     382
America/Denver          191
Europe/London            74
Asia/Tokyo               37
Pacific/Honolulu         36
Europe/Madrid            35
America/Sao_Paulo        33
Name: tz, dtype: int64
```

那可能有人会好奇，这个生成的 DataFrame 对象是什么样的呢？下面是`print(frame)`的结果，可以看出它将 key value 值表格化（行转列了）。

```bash
       _heartbeat_                                                  a                        ...                                           tz                                                  u
0              NaN  Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKi...                        ...                             America/New_York        http://www.ncbi.nlm.nih.gov/pubmed/22415991
1              NaN                             GoogleMaps/RochesterNY                        ...                               America/Denver        http://www.monroecounty.gov/etc/911/rss.php
2              NaN  Mozilla/4.0 (compatible; MSIE 8.0; Windows NT ...                        ...                             America/New_York  http://boxer.senate.gov/en/press/releases/0316...
3              NaN  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8)...                        ...                            America/Sao_Paulo            http://apod.nasa.gov/apod/ap120312.html
4              NaN  Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKi...                        ...                             America/New_York  http://www.shrewsbury-ma.gov/egov/gallery/1341...
5              NaN  Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKi...                        ...                             America/New_York  http://www.shrewsbury-ma.gov/egov/gallery/1341...
6              NaN  Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.1...                        ...                                Europe/Warsaw  http://www.nasa.gov/mission_pages/nustar/main/...
7              NaN  Mozilla/5.0 (Windows NT 6.1; rv:2.0.1) Gecko/2...                        ...                                               http://www.nasa.gov/mission_pages/nustar/main/...
8              NaN  Opera/9.80 (X11; Linux zbov; U; en) Presto/2.1...                        ...                                               http://www.nasa.gov/mission_pages/nustar/main/...
9              NaN  Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKi...                        ...                                                         http://apod.nasa.gov/apod/ap120312.html
10             NaN  Mozilla/5.0 (Windows NT 6.1; WOW64; rv:10.0.2)...                        ...                          America/Los_Angeles  https://www.nysdot.gov/rexdesign/design/commun...
11             NaN  Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.4...                        ...                             America/New_York  http://oversight.house.gov/wp-content/uploads/...
12             NaN  Mozilla/5.0 (Windows NT 6.1; WOW64; rv:10.0.2)...                        ...                             America/New_York  https://www.nysdot.gov/rexdesign/design/commun...
13    1.331923e+09                                                NaN                        ...                                          NaN                                                NaN
14             NaN  Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US...                        ...                             America/New_York               http://toxtown.nlm.nih.gov/index.php
15             NaN  Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1...                        ...                               Asia/Hong_Kong  http://www.ssd.noaa.gov/PS/TROP/TCFP/data/curr...
16             NaN  Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1...                        ...                               Asia/Hong_Kong  http://www.usno.navy.mil/NOOC/nmfc-ph/RSS/jtwc...
17             NaN  Mozilla/5.0 (Macintosh; Intel Mac OS X 10.5; r...                        ...                             America/New_York  http://www.usda.gov/wps/portal/usda/usdahome?c...
18             NaN                             GoogleMaps/RochesterNY                        ...                               America/Denver        http://www.monroecounty.gov/etc/911/rss.php
19             NaN  Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKi...                        ...                                  Europe/Rome  http://www.nasa.gov/mission_pages/nustar/main/...
20             NaN  Mozilla/5.0 (compatible; MSIE 9.0; Windows NT ...                        ...                                 Africa/Ceuta  http://voyager.jpl.nasa.gov/imagesvideo/uranus...
21             NaN  Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.6...                        ...                             America/New_York  http://www.nasa.gov/mission_pages/nustar/main/...
22             NaN  Mozilla/4.0 (compatible; MSIE 8.0; Windows NT ...                        ...                             America/New_York  http://portal.hud.gov/hudportal/documents/hudd...
23             NaN  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3)...                        ...                             America/New_York  http://www.tricare.mil/mybenefit/ProfileFilter...
24             NaN  Mozilla/5.0 (Windows; U; Windows NT 5.1; es-ES...                        ...                                Europe/Madrid  http://www.nasa.gov/mission_pages/nustar/main/...
25             NaN  Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1...                        ...                            Asia/Kuala_Lumpur  http://www.nasa.gov/mission_pages/nustar/main/...
26             NaN  Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1...                        ...                                 Asia/Nicosia  http://www.nasa.gov/mission_pages/nustar/main/...
27             NaN  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8)...                        ...                            America/Sao_Paulo            http://apod.nasa.gov/apod/ap120312.html
28             NaN  Mozilla/5.0 (iPad; CPU OS 5_0_1 like Mac OS X)...                        ...                                               https://www.nysdot.gov/rexdesign/design/commun...
29             NaN  Mozilla/5.0 (iPad; U; CPU OS 3_2 like Mac OS X...                        ...                                               http://www.ed.gov/news/media-advisories/us-dep...
...            ...                                                ...                        ...                                          ...                                                ...
3530           NaN  Mozilla/5.0 (Windows NT 6.0) AppleWebKit/535.1...                        ...                          America/Los_Angeles  http://www.nasa.gov/multimedia/imagegallery/im...
3531           NaN  Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6...                        ...                                               http://www.nasa.gov/mission_pages/nustar/main/...
3532           NaN  Mozilla/5.0 (Windows NT 6.1; WOW64; rv:10.0.2)...                        ...                             America/New_York  http://portal.hud.gov/hudportal/HUD?src=/press...
3533           NaN  Mozilla/5.0 (iPad; CPU OS 5_1 like Mac OS X) A...                        ...                             America/New_York                         http://apod.nasa.gov/apod/
3534           NaN  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8)...                        ...                              America/Chicago  https://www.nysdot.gov/rexdesign/design/commun...
3535           NaN  Mozilla/5.0 (Windows NT 5.1; rv:10.0.2) Gecko/...                        ...                              America/Chicago  http://ntl.bts.gov/lib/44000/44300/44374/FHWA-...
3536           NaN  Mozilla/5.0 (BlackBerry; U; BlackBerry 9800; e...                        ...                                               http://www.nasa.gov/mission_pages/hurricanes/a...
3537           NaN  Mozilla/5.0 (Windows NT 6.1; WOW64; rv:10.0.2)...                        ...                          America/Tegucigalpa            http://apod.nasa.gov/apod/ap120312.html
3538           NaN  Mozilla/5.0 (iPhone; CPU iPhone OS 5_1 like Ma...                        ...                          America/Los_Angeles  http://healthypeople.gov/2020/connect/webinars...
3539           NaN    Mozilla/5.0 (compatible; Fedora Core 3) FC3 KDE                        ...                          America/Los_Angeles  http://www.federalreserve.gov/newsevents/press...
3540           NaN  Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKi...                        ...                               America/Denver  http://www.nasa.gov/mission_pages/nustar/main/...
3541           NaN  Mozilla/5.0 (X11; U; OpenVMS AlphaServer_ES40;...                        ...                          America/Los_Angeles  http://www.federalreserve.gov/newsevents/press...
3542           NaN  Mozilla/5.0 (compatible; MSIE 9.0; Windows NT ...                        ...                          America/Los_Angeles  http://www.sba.gov/community/blogs/community-b...
3543  1.331927e+09                                                NaN                        ...                                          NaN                                                NaN
3544           NaN  Mozilla/5.0 (Windows NT 6.1; WOW64; rv:5.0.1) ...                        ...                              America/Chicago  https://www.nysdot.gov/rexdesign/design/commun...
3545           NaN  Mozilla/5.0 (Windows NT 6.1; WOW64; rv:10.0.2)...                        ...                              America/Chicago  https://www.nysdot.gov/rexdesign/design/commun...
3546           NaN  Mozilla/5.0 (iPhone; CPU iPhone OS 5_1 like Ma...                        ...                          America/Los_Angeles  http://healthypeople.gov/2020/connect/webinars...
3547           NaN  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8)...                        ...                             America/New_York  http://www.epa.gov/otaq/regs/fuels/additive/e1...
3548           NaN  Mozilla/5.0 (iPhone; CPU iPhone OS 5_1 like Ma...                        ...                              America/Chicago    http://www.fda.gov/Safety/Recalls/ucm296326.htm
3549           NaN  Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKi...                        ...                             Europe/Stockholm  http://www.nasa.gov/mission_pages/WISE/main/in...
3550           NaN  Mozilla/4.0 (compatible; MSIE 8.0; Windows NT ...                        ...                             America/New_York  http://www.nlm.nih.gov/medlineplus/news/fullst...
3551           NaN  Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKi...                        ...                                               http://www.nasa.gov/mission_pages/nustar/main/...
3552           NaN  Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US...                        ...                              America/Chicago  http://travel.state.gov/passport/passport_5535...
3553           NaN  Mozilla/4.0 (compatible; MSIE 7.0; Windows NT ...                        ...                             America/New_York  http://www.shrewsbury-ma.gov/egov/gallery/1341...
3554           NaN  Mozilla/4.0 (compatible; MSIE 7.0; Windows NT ...                        ...                             America/New_York  http://www.shrewsbury-ma.gov/egov/gallery/1341...
3555           NaN  Mozilla/4.0 (compatible; MSIE 9.0; Windows NT ...                        ...                             America/New_York  http://www.fda.gov/AdvisoryCommittees/Committe...
3556           NaN  Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.1...                        ...                              America/Chicago  http://www.okc.gov/PublicNotificationSystem/Fo...
3557           NaN                             GoogleMaps/RochesterNY                        ...                               America/Denver        http://www.monroecounty.gov/etc/911/rss.php
3558           NaN                                     GoogleProducer                        ...                          America/Los_Angeles                http://www.ahrq.gov/qual/qitoolkit/
3559           NaN  Mozilla/4.0 (compatible; MSIE 8.0; Windows NT ...                        ...                             America/New_York  http://herndon-va.gov/Content/public_safety/Pu...

[3560 rows x 18 columns]
```

# 对时区数据进行微处理并生成图片

有一些数据的`tz`值是`''`的，我们当成 Unknown，也有一些是缺省的，没有这个 key，我们就设成 Missing

```python
clean_tz = frame['tz'].fillna('Missing')
# print(clean_tz)
clean_tz[clean_tz == ''] = 'Unknown'
tz_counts = clean_tz.value_counts()
print(tz_counts[:10])
```

output:

```bash
America/New_York       1251
Unknown                 521
America/Chicago         400
America/Los_Angeles     382
America/Denver          191
Missing                 120
Europe/London            74
Asia/Tokyo               37
Pacific/Honolulu         36
Europe/Madrid            35
Name: tz, dtype: int64
```

之后我们可以画出对应的柱状图，如下所示：

```python
tz_counts[:10].plot(kind='barh', rot=0)
plt.show()
```

![2019-01-05-PyAnalysisLearn01.01.jpg](img/post/2019/01/2019-01-05-PyAnalysisLearn01.01.jpg)

# 解析 Agent 信息

上面我们可以看到 Agent 的初始信息很多，字符串很长，但是我们可以根据特征使用字符串函数与正则表达式先解析出来。

下面就是将空值去掉并进行计数：

```python
results = Series([x.split()[0] for x in frame.a.dropna()])
agent_counts = results.value_counts()[:8]
print(agent_counts)
```

output:

```bash
Mozilla/5.0                 2594
Mozilla/4.0                  601
GoogleMaps/RochesterNY       121
Opera/9.80                    34
TEST_INTERNET_AGENT           24
GoogleProducer                21
Mozilla/6.0                    5
BlackBerry8520/5.0.0.681       4
dtype: int64
```

在 Agent 信息中我们可以看到一些有 Windows 的字样，表示是在 windows 系统下访问的，而没有的话，我们就认真是非 Windows 的。

```python
frame.loc[frame.tz=='', 'tz'] = 'Unknown'
cframe = frame[frame.a.notnull()]
operation_system = np.where(cframe['a'].str.contains('Windows'), 'Windows', 'Not Windows')
print(operation_system[:5])
```

我们可以看到前面 5 条数据 agent 中的情况
output:

```bash
['Windows' 'Not Windows' 'Windows' 'Not Windows' 'Windows']
```

# 根据时区和系统进行分组解析

上面我们可以得到时区和所使用系统的分布，下面我们就可以根据这些来进行分组，得知在一个时区里系统的使用情况。

```python
by_tz_os = cframe.groupby(['tz', operation_system])
agg_counts = by_tz_os.size().unstack().fillna(0)
print(agg_counts[:10])
```

output:下面是取前面的 10 个值（还没有排序）

```bash
                                Not Windows  Windows
tz
                                      245.0    276.0
Africa/Cairo                            0.0      3.0
Africa/Casablanca                       0.0      1.0
Africa/Ceuta                            0.0      2.0
Africa/Johannesburg                     0.0      1.0
Africa/Lusaka                           0.0      1.0
America/Anchorage                       4.0      1.0
America/Argentina/Buenos_Aires          1.0      0.0
America/Argentina/Cordoba               0.0      1.0
America/Argentina/Mendoza               0.0      1.0
```

接下来主要目的是选取最常出现的时区（即 Not Windows 和 Windows 的数量加起来的数量最多的时区，其实直接 count 时区是一样的），所以我们要先构造一个间接索引数组。
然后通过 take 按照这个顺序截取了最后 10 行,因为是升序，所以是 top10

```python
# order by asc
indexer = agg_counts.sum(1).argsort()
count_subset = agg_counts.take(indexer)[-10:]
print(count_subset)

# draw the real number
count_subset.plot(kind='barh', stacked=True)

# draw with %
normed_subset = count_subset = count_subset.div(count_subset.sum(1), axis=0)
normed_subset.plot(kind='barh', stacked=True)
plt.show()
```

output:

```bash
tz
America/Sao_Paulo           13.0     20.0
Europe/Madrid               16.0     19.0
Pacific/Honolulu             0.0     36.0
Asia/Tokyo                   2.0     35.0
Europe/London               43.0     31.0
America/Denver             132.0     59.0
America/Los_Angeles        130.0    252.0
America/Chicago            115.0    285.0
Unknown                    245.0    276.0
America/New_York           339.0    912.0
```

这里画了两张图，一张为实际数值:
![2019-01-05-PyAnalysisLearn01.02.jpg](img/post/2019/01/2019-01-05-PyAnalysisLearn01.02.jpg)

另一张则为百分比：
![2019-01-05-PyAnalysisLearn01.03.jpg](img/post/2019/01/2019-01-05-PyAnalysisLearn01.03.jpg)

# 最后

在学习的过程中，不出意外的我遇到了 SettingwithCopyWarning 的坑，这个是由于**链式赋值（链式索引和赋值的组合）**的引起的，还是有点意思的，下面的参考中就有一些资料帮助理解。
虽然只是warning，但还是不应该出现并且避免，因为说明有风险，赋值操作可能与想的不一样。

参考：

- [pandas 的 SettingWithCopyWarning 警告出现的原因和如何避免](https://blog.csdn.net/haolexiao/article/details/81180571)
- [Pandas 中 SettingwithCopyWarning 的原理和解决方案](https://www.jianshu.com/p/72274ccb647a)
- [Python 笔试集（1）：关于 Python 链式赋值的坑](https://blog.csdn.net/jmilk/article/details/78733409)
