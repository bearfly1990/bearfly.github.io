---
layout: post
title: Python Thread Lock
subtitle: Python thread writing to file by lock
date: 2018-12-11
author: BF
header-img: img/bf/beach_06.jpg
catalog: true
tags:
  - python
  - thread
  - lock
---

# 背景

这两天在继续修改之前写的测试脚本。这两天想要实现并发跑一些任务，之后还要把结果写要 testresult.csv 中。

这样的话就涉及一个问题，因为是并发跑的，所以可以会出现资源争抢的问题，那么就需要对读写加锁控制。

在网上查了下，发现有第三方文件锁库 fcntl 可以使用，不过可惜的是 it's [Unix Specific](https://docs.python.org/3/library/unix.html)

所以最直接的还是用线程锁来解决这个问题

# threading

在 threading 模块中定义了 Lock 类，可以方便的处理锁定：

```python
# create lock
mutex = threading.Lock()
# acquire lock
mutex.acquire([timeout])
# release lock
mutex.release()
```

## Demo

下面是我写的简单的例子，模拟了实际的情况，每个 job 跑完的时间是不一定的，当他们想要写测试结果文件中写数据的时候要先去获得锁。

```python
import threading
import time
import random

mutex = threading.Lock()

def job_running():
    time.sleep(random.randint(1,6))

def write_result(txt_file, thread_name):
    mutex.acquire(10)
    with open(txt_file, 'a') as f:
        print("Thread {0} acquire lock".format(thread_name))
        f.write("write from thread {0} \r\n".format(thread_name))
        time.sleep(random.randint(1,3))
    mutex.release()
    print("Thread {0} exit".format(thread_name))

def run_job(txt_file):
    thread_name = threading.currentThread().getName()
    job_running()
    write_result(txt_file, thread_name)

if __name__ == '__main__':
    for i in range(5):
        myThread = threading.Thread(target=run_job, args=("testResult.txt",))
        myThread.start()
```

运行的结果，和预期的效果一样：

```bash
PS C:\Users\mayn\Desktop\workspace\python_file_lock> python .\file_lock_demo.py
Thread Thread-3 acquire lock
Thread Thread-3 exit
Thread Thread-5 acquire lock
Thread Thread-5 exit
Thread Thread-2 acquire lock
Thread Thread-2 exit
Thread Thread-1 acquire lock
Thread Thread-1 exit
Thread Thread-4 acquire lock
Thread Thread-4 exit
```

# 使用fcntl

因为在Windows下无法使用，自己也没去尝试，有兴趣可以通过最后参考中的链接去了解。

# 最后

多线程是个可以探究的很深的问题，还有许多的方面需要继续学习。

---

参考：

- [Python 多线程读写文件加锁](https://blog.csdn.net/qq_30554229/article/details/80094253)
- [Python 的文件锁使用](https://blog.csdn.net/u011734144/article/details/78739442)
