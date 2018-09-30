---
layout:     post
title:      ACM with Python
subtitle:   Code with python to solve with ACM problems
date:       2018-05-09
author:     BF
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - python
    - acm
---
## 使用python编写acm题目
ACM一般来说都是用C或者C++来编写，因为国内的大学入门教程就是以C和C++为主，而且编译后的执行效率也比较高。但是很多的Online Judge都支持别的语言，例如Java，Python，Perl，Pascal，PHP，FPC，C#等等。

省内我们之前常用的几个学校的OJ系统([浙大](http://acm.zju.edu.cn/onlinejudge/)，[杭电](http://acm.hdu.edu.cn/)，[工大](http://acm.zjut.edu.cn/onlinejudge/))，只有浙大的才支持Python，而且只有2.7.3,不支持Python3+。不过大同小异，下面就用简单的例子来演示下用python来写acm。

### [1001 A+B Problem](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=1001)
哈哈，就以这个最简单的a+b问题为例，下面是一个标准的输入输出样式，读取样例，输出结果。
```python
while True:  
    try:  
        a, b = map(int, raw_input().strip().split())  
        print a + b
    except EOFError:  
        break
```
本来还想再写几个的，但暂时没发现特别简单的，哈哈哈哈哈！

Python输入样例参考：[Python - Input](https://github.com/bearfly1990/PowerScript/blob/master/Python3/acm/PythonInput.py)
