---
layout:     post
title:      Catch CPU/Memory Monitor
subtitle:   Catch monitor data from whole process
date:       2018-06-14
author:     BF
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - python
    - log
---
## 根据log中的实际取得monitor中的数据。
现在有一个完整的导数据的log，类似于如下ImportLog.txt：
```log
...略...
2018-06-14 00:07:58,156 [INFO ] -----START IMPORT-----
2018-06-14 00:07:58,515 [INFO ] xxx
2018-06-14 00:10:39,065 [INFO ] xxx
2018-06-14 00:10:39,081 [INFO ] Import Time(s):160.55
2018-06-14 00:10:39,128 [INFO ] xxx
2018-06-14 00:10:39,128 [INFO ] Finished:xxxAAxxx
2018-06-14 00:10:39,128 [INFO ] Import Time(s):160.96
2018-06-14 00:10:39,128 [INFO ] -----END IMPORT-----
2018-06-14 00:10:44,201 [INFO ] -----START IMPORT-----
2018-06-14 00:10:46,590 [INFO ] xxx\\IMPORT.exe BBB
2018-06-14 00:57:30,360 [INFO ] Finished:\xxx_BBB_xxx
2018-06-14 00:57:30,360 [INFO ] Import Time(s):2803.77
2018-06-14 00:57:31,125 [INFO ] -----END IMPORT-----
...略...
```
而整个过程有监控数据类似如下Monitor.csv：
```log
Time,CPU(%),Memory(%)
2018/06/13 23:51:17,2.7,12.17
2018/06/13 23:51:19,0.0,12.16
2018/06/13 23:51:21,0.0,12.16
2018/06/13 23:51:23,0.0,12.14
2018/06/13 23:51:25,0.4,12.15
2018/06/13 23:51:28,0.0,12.15
2018/06/13 23:51:30,0.0,12.15
...略...
```
那么现在的问题是只需要在跑BBB的时候的那段时间的监控数据，于是临时写了个简单的HardCode的过程达到目的，主要用到了datetime对时间的处理。
```python
"""
author: xiche
create at: 06/14/2018
description:
    Catch monitor data from whole proces
"""
from datetime import datetime

# datetime_object = datetime.strptime('Jun 1 2005  1:33PM', '%b %d %Y %I:%M%p')
dateformat1 = "%Y-%m-%d %H:%M:%S" #"2018-06-13 23:51:17"
dateformat2 = "%Y/%m/%d %H:%M:%S" #"2018/06/13 23:51:52"

def __main__():
    dt_start = None
    dt_end = None
    lines_new = []
    with open("ImportLog.txt", "r") as f:
        lines = f.readlines()
        for line in lines:
            if("BBB" in line and "Finished" in line):
                dt_end = datetime.strptime(line[0:19], dateformat1)
                break
            if("BBB" in line):
                dt_start = datetime.strptime(line[0:19], dateformat1)

    with open("Monitor.csv", "r") as f:
        lines = f.readlines()
        dt_current = None
        for line in lines:
            try:
                dt_current = datetime.strptime(line[0:19], dateformat2)
                if(dt_current < dt_start):
                    continue
                elif(dt_current > dt_end):
                    break #之前竟然是continue。。。。
                else:
                    lines_new.append(line)
            except:
                print("No expect formated:{}".format(line))

    with open("Monitor_BBB.csv", "w") as f:
        f.writelines(lines_new)     
    # dt_start    = datetime.strptime(dt_start_str, dateformat1)
    # dt_end      = datetime.strptime(dt_start_str, dateformat2)

    # a = datetime.strptime("2018-06-13 23:51:17", "%Y-%m-%d %H:%M:%S")
    # b = datetime.strptime("2018/06/13 23:51:52", "%Y/%m/%d %H:%M:%S")
    # print(a, b)
__main__()
```

