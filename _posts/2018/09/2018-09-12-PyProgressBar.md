---
layout: post
title: Progress Bar in Python
subtitle: Several ways to implement progress bar in console
date: 2018-09-12
author: BF
header-img: img/bf/city_02.jpg
catalog: true
tags:
  - python
  - progress bar
---

# Progress Bar in Python

很多时候，我们在测试的过程中，希望能够直观的看到测试的进度，而不是去看 log 或者查询数据库得到信息。

这个时候，如果有数据完成的进度的话就非常的方便。

今天就简单总结下这两天查询的资料，并且已经在实际中用到。

## How

如果要在 Console 中实现进度条，那就是要所有所有字符全部在同一行，而且可以修改。

如果直接使用 print 语句，python 会结尾加上换行符(`\n`)，这就导致在 Console 下一旦被 print 之后就无法再修改了。所以我们不能直接使用 `print`

通过 `sys.stdout.write()` 我们就可以在 Console 输出字符，并且不会加上任何结尾。

通过 `sys.stdout.flush()` 可以把输出暂时打印在 Console 中。

而通过转义字符`\r`，就可以回到首行，然后如果继续输出的话，就可以连续在同一行改变字符，实现进度的效果。

下面就简单介绍一些进度条的实现，网上一些作者的实例。

## Demo 01

下面是一个最简单的实现：

```python
import sys, time

for i in range(50):
    sys.stdout.write('{0}\r'.format( '*' * (i + 1)))
    sys.stdout.flush()
    time.sleep(0.2)
```

![Demo01](https://raw.githubusercontent.com/bearfly1990/bearfly1990.github.io/master/_posts/2018/09/imgs/2018-09-12-PyProgressBar-demo01.gif)

## Demo 02

这个例子是上一个例子的加强版，有数值进度和向后推进的箭头。

```python
import sys,time
j = '#'
for i in range(1,61):
    j += '#'
    sys.stdout.write(str(int((i/60)*100))+'% '+j+'->'+ "\r")
    sys.stdout.flush()
    time.sleep(0.2)
```

![Demo02](https://raw.githubusercontent.com/bearfly1990/bearfly1990.github.io/master/_posts/2018/09/imgs/2018-09-12-PyProgressBar-demo02.gif)

## Demo 03

这个例子更简单一些，只有进度的数值。

```python
import sys
from time import sleep
def viewBar(i):
    output = sys.stdout
    for count in range(0, i + 1):
        second = 0.1
        sleep(second)
        output.write('\rcomplete percent ----->:%.0f%%' % count)
    output.flush()
viewBar(100)
```

![Demo03](https://raw.githubusercontent.com/bearfly1990/bearfly1990.github.io/master/_posts/2018/09/imgs/2018-09-12-PyProgressBar-demo03.gif)

## Demo 04

这个例子用了一个比较常用的第三方库`tqdm`,有兴趣大家可以去搜一下官方文档，还是比较灵活的。

```python
from time import sleep
from tqdm import tqdm
for i in tqdm(range(1, 500), ascii=True):
    sleep(0.01)
```

![Demo04](https://raw.githubusercontent.com/bearfly1990/bearfly1990.github.io/master/_posts/2018/09/imgs/2018-09-12-PyProgressBar-demo04.gif)

## Demo 05

这个例子其实和`Demo 02`差不多的

```python
import sys
import time
def view_bar(num,total):
    rate = num / total
    rate_num = int(rate * 100)
    #r = '\r %d%%' %(rate_num)
    r = '\r%s>%d%%' % ('=' * rate_num, rate_num,)
    sys.stdout.write(r)
    sys.stdout.flush
if __name__ == '__main__':
    for i in range(0, 101):
        time.sleep(0.1)
        view_bar(i, 100)
```

![Demo05](https://raw.githubusercontent.com/bearfly1990/bearfly1990.github.io/master/_posts/2018/09/imgs/2018-09-12-PyProgressBar-demo05.gif)

## Demo 06

这个例子是我觉得相对比较好的，自己改良了一下放在自己项目中用了，下面是原始的 sample，我加了线程的处理，这样就可以异步写进度，而不会影响原来的进程。

```python
import sys, time
import random
import _thread
class ShowProcess():
    """
    显示处理进度的类
    调用该类相关函数即可实现处理进度的显示
    """
    i = 0 # 当前的处理进度
    max_steps = 0 # 总共需要处理的次数
    max_arrow = 50 #进度条的长度
    infoDone = 'done'

    # 初始化函数，需要知道总共的处理次数
    def __init__(self, max_steps, infoDone = 'Done'):
        self.max_steps = max_steps
        self.i = 0
        self.infoDone = infoDone

    # 显示函数，根据当前的处理进度i显示进度
    # 效果为[>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]100.00%
    def show_process(self, i=None):
        if i is not None:
            self.i = i
        else:
            self.i += 1
        num_arrow = int(self.i * self.max_arrow / self.max_steps) #计算显示多少个'>'
        num_line = self.max_arrow - num_arrow #计算显示多少个'-'
        percent = self.i * 100.0 / self.max_steps #计算完成进度，格式为xx.xx%
        process_bar = '[{}{}] {:02}%\r'.format('#' * num_arrow, '-' * num_line,percent) #带输出的字符串，'\r'表示不换行回到最左边#带输出的字符串，'\r'表示不换行回到最左边
        sys.stdout.write(process_bar) #这两句打印字符到终端
        sys.stdout.flush()
        if self.i >= self.max_steps:
            self.close()

    def close(self):
        print('')
        print(self.infoDone)
        self.i = 0

VALUE = 0

def start_import():
    global VALUE
    while(True):
        VALUE += 1
        time.sleep(0.2 * random.random())
        if VALUE == 100:
            time.sleep(2)
            break

def monitor_process( threadName, delay):
    max_steps = 100
    process_bar = ShowProcess(max_steps, 'OK') # 1.在循环前定义类的实体， max_steps是总的步数， infoDone是在完成时需要显示的字符串
    # for i in range(max_steps):
    while(True):
        process_bar.show_process(VALUE)      # 2.显示当前进度
        time.sleep(1)
        if(VALUE == 100):
            process_bar.show_process(100)
            break

_thread.start_new_thread( monitor_process, ("Thread-1", 2, ) )
start_import()
```

![Demo06](https://raw.githubusercontent.com/bearfly1990/bearfly1990.github.io/master/_posts/2018/09/imgs/2018-09-12-PyProgressBar-demo06.gif)

更多信息可以参考：

[https://www.cnblogs.com/jsben/p/5792952.html](https://www.cnblogs.com/jsben/p/5792952.html)

[https://pypi.org/project/tqdm/](https://pypi.org/project/tqdm/)
