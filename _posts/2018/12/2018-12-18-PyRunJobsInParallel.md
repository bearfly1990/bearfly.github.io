---
layout: post
title: Run Jobs In Parallel
subtitle: Using yaml and python to run jobs in parallel
date: 2018-12-18
author: BF
header-img: img/bf/sun_03.jpg
catalog: true
tags:
  - python
  - yaml
  - thread
---

# 背景

这周的一个目标是把之前 Benchmark 的脚本改写，使它支持并发的跑 job。

之前的配置是使用 ini，不适合配置现在这种需求，也不想用之前的 xml 来配置，所以今天就简单的研究了下使用 yaml 来配置并运行。

# yaml 配置文件

这面是简单的一个例子，希望最后执行的时候，不同的 task 之间是串行在跑的，而同一个 task 里的 job 是并发跑的：

```yaml
description: sample to run jobs in parallel
tasks:
  - taskid: 01
    desc: task 01
    jobs:
      - jobid: 001
        name: job 001 in parallel
      - jobid: 002
        name: job 002 in parallel
      - jobid: 003
        name: job 003 in parallel
      - jobid: 004
        name: job 004 in parallel
  - taskid: 02
    desc: task 02
    jobs:
      - jobid: 005
        name: job 005
  - taskid: 03
    desc: task 03
    jobs:
      - jobid: 06
        name: job 006
```

# 使用 Queue 控制运行

下面直接上代码:

```python
import yaml
import threading
import random, time
import queue

class Job:
    def __init__(self, jobid, name):
        self.jobid = jobid
        self.name = name

class JobProcessor():
    def __init__(self, queue):
        self.queue = queue

    def run_job(self, job):
        print('start run job {}'.format(job.name))
        time.sleep(random.randint(1,10))
        print('ended run job {}'.format(job.name))
        self.queue.task_done()

    def run_jobs(self):
        while not self.queue.empty():
            job = self.queue.get()
            myThread = threading.Thread(target=self.run_job, args=(job,))
            myThread.start()

def main():
    f = open('jobs.yaml','r',encoding='utf-8')
    yaml_content = f.read()
    json_content = yaml.load(yaml_content)
    tasks_list = json_content['tasks']

    for task in tasks_list:
        jobs = task['jobs']
        jobs_num = len(jobs)
        jobs_queue = queue.Queue(jobs_num)
        for job in jobs:
            jobs_queue.put(Job(job['jobid'], job['name']))
        JobProcessor(jobs_queue).run_jobs()
        jobs_queue.join()

if __name__ == '__main__':
    main()
```

仔细看的话，主要是利用 queue 存储了 job 信息，然后利用`.task_done`和`.join`方法，实现等待任务的完成的操作。

下面是运行的结果：

```bash
C:\Users\bearfly1990\Desktop\python_test\task_parallel>python runjobs.py
start run job job 001 in parallel
start run job job 002 in parallel
start run job job 003 in parallel
start run job job 004 in parallel
ended run job job 001 in parallel
ended run job job 004 in parallel
ended run job job 003 in parallel
ended run job job 002 in parallel
start run job job 005
ended run job job 005
start run job job 006
ended run job job 006
```

# 升级配置，增加功能

之前（上面）的简单例子是不能够满足实际的需求的，只是多线程的一个简单例子。

这周花了些时间，增加了一些配置项，针对 Job 提供了 JobGroups，每个 Group 之间都是默认并行的，但是也可以通过 pregroup 来限定先后顺序。
而 group 内部，我则希望是可以配置的，既可以并行，也可以串行。

下面是新的 yaml 配置：

```yaml
description: sample to run jobs
Tasks:
  - taskid: 01
    name: task 01
    mode: import
    type: T
    file: 01.txt
  - taskid: 02
    name: task 02
    mode: import
    type: S
    file: 02.txt
  - taskid: 03
    name: task 03
    mode: jobgroup
    JobGroups:
      - id: 1
        name: G1
        mode: parallel
        Jobs:
          - jobid: 001
            name: job G1-01
          - jobid: 002
            name: job G1-02
          - jobid: 003
            name: job G1-03
          - jobid: 004
            name: job G1-04
      - id: 2
        name: G2
        mode: parallel
        Jobs:
          - jobid: 005
            name: job G2-05
          - jobid: 006
            name: job G2-06
          - jobid: 007
            name: job G2-07
          - jobid: 008
            name: job G2-08
      - id: 3
        name: G3
        mode: serial
        pregroup: 1
        Jobs:
          - jobid: 009
            name: job G3-09
          - jobid: 010
            name: job G3-10
          - jobid: 011
            name: job G3-11
          - jobid: 012
            name: job G3-12
  - taskid: 04
    name: hello world
    mode: say
  - taskid: 05
    name: test passed
    mode: wait
```

下面是具体的实现，已经在一些关键点加了备注：

```python
import yaml
import threading
import random
import time
import queue

class ImportInfo:
    def __init__(self, name, type, file):
        self.name = name
        self.type = type
        self.file = file
class Job:
    def __init__(self, jobid, name):
        self.jobid = jobid
        self.name = name

class JobProcessor():
    def __init__(self, queue=None):
        self.queue = queue

    def run_job(self, job):
        print('start run job {}'.format(job.name))
        time.sleep(random.randint(1, 7))
        print('ended run job {}'.format(job.name))
        self.queue.task_done()

    def run_jobs_parallel(self):
        while not self.queue.empty():
            job = self.queue.get()
            my_thread = threading.Thread(target=self.run_job, args=(job,))
            my_thread.start()
        self.queue.join()

    def run_jobs_serial(self):
        while not self.queue.empty():
            job = self.queue.get()
            self.run_job(job)
        self.queue.join()

    def run_job_group(self, job_group, job_group_map):

        pregroup = job_group.get('pregroup', -1)
        jobs = job_group['Jobs']
        # default mode is parallel
        mode = job_group.get('mode', 'parallel')
        # add jobs to the queue
        jobs_num = len(jobs)
        jobs_queue = queue.Queue(jobs_num)
        job_group_map[job_group['id']] = jobs_queue
        for job in jobs:
            jobs_queue.put(Job(job['jobid'], job['name']))
        # wait for pregroup run finished if have pregroup
        if(pregroup != -1):
            while(True):
                pre_queue = job_group_map.get(pregroup, None)
                if pre_queue:
                    pre_queue.join()
                    break
                time.sleep(1)

        print('### start run jobgroup:{} ###'.format(job_group['name']))
        if(mode == 'parallel'):
            JobProcessor(jobs_queue).run_jobs_parallel()
        else:
            JobProcessor(jobs_queue).run_jobs_serial()
        print('### end run jobgroup:{} ###'.format(job_group['name']))

    def run_job_groups(self, job_groups):
        # map each group queue, so that we could check the group is finished or not.
        job_group_map = {}
        thread_list = []

        for job_group in job_groups:
            run_job_group_thread = threading.Thread(
                target=self.run_job_group, args=(job_group, job_group_map))
            thread_list.append(run_job_group_thread)
            run_job_group_thread.start()
        # wait for all groups are finished
        for thread in thread_list:
            thread.join()

def run_import(importInfo):
    print('import {}'.format(importInfo.name))

def main():
    fin = open('jobs.yaml', 'r', encoding='utf-8')
    json_content = yaml.load(fin)
    task_list = json_content['Tasks']

    for task in task_list:
        print(task['name'], '|', task['mode'], '|', task.get('type', 'None'), '|', task.get('file', 'None'))
        task_mode = task['mode'].lower()
        if(task_mode == 'jobgroup'):
            JobProcessor().run_job_groups(task['JobGroups'])
        elif(task_mode == 'import'):
            run_import(ImportInfo(task['name'], task['type'], task['file']))
        elif(task_mode == 'say'):
            print('say {}'.format(task['name']))
        elif(task_mode == 'wait'):
            print('wait {}'.format(task['name']))

if __name__ == '__main__':
    main()
```

下面是一次跑的例子：

```bash
c:\Users\mayn\Desktop\workspace\python_yaml>python runjobs.py
task 01 | import | T | 01.txt
import task 01
task 02 | import | S | 02.txt
import task 02
task 03 | jobgroup | None | None
### start run jobgroup:G1 ###
### start run jobgroup:G2 ###
start run job job G1-01
start run job job G1-02
start run job job G2-05
start run job job G2-06
start run job job G1-03
start run job job G1-04
start run job job G2-07
start run job job G2-08
ended run job job G2-05
ended run job job G1-02
ended run job job G1-01
ended run job job G2-06
ended run job job G1-03
ended run job job G2-08
ended run job job G1-04
ended run job job G2-07
### start run jobgroup:G3 ###
### end run jobgroup:G1 ###
start run job job G3-09
### end run jobgroup:G2 ###
ended run job job G3-09
start run job job G3-10
ended run job job G3-10
start run job job G3-11
ended run job job G3-11
start run job job G3-12
ended run job job G3-12
### end run jobgroup:G3 ###
hello world | say | None | None
say hello world
test passed | wait | None | None
wait test passed
```

# 使用 map 和 multiprocessing

在晓辉建议下，使用 map 函数结合线程池简化了代码，不用自己控制并发了，api 更简单。

```python
import yaml
import threading
import random
import time
import queue
from multiprocessing.dummy import Pool as ThreadPool

class ImportInfo:
    def __init__(self, name, type, file):
        self.name = name
        self.type = type
        self.file = file
class Job:
    def __init__(self, jobid, name):
        self.jobid = jobid
        self.name = name

class JobProcessor():
    def __init__(self, queue=None):
        self.queue = queue

    def run_job(self, job):
        print('start run job {}'.format(job.name))
        time.sleep(random.randint(1, 7))
        print('ended run job {}'.format(job.name))

    def run_job_group(self, param):
        job_group = param['job_group']
        job_group_map = param['job_group_map']
        pregroup = job_group.get('pregroup', -1)
        # add jobs to a new list
        jobs = [Job(job['jobid'], job['name']) for job in job_group['Jobs']]
        # default mode is parallel
        mode = job_group.get('mode', 'parallel')
        jobs_thread_pool = ThreadPool(len(jobs))
        job_group_map[job_group['id']] = jobs_thread_pool
        # wait for pregroup run finished if have pregroup
        if(pregroup != -1):
            while(True):
                pre_group_pool = job_group_map.get(pregroup, None)
                if pre_group_pool:
                    pre_group_pool.close()
                    pre_group_pool.join()
                    break
                time.sleep(1)
        print('### start run jobgroup:{} ###'.format(job_group['name']))
        if(mode == 'parallel'):
            jobs_thread_pool.map(self.run_job, jobs)
        else:
            list(map(self.run_job, jobs))

        print('### end run jobgroup:{} ###'.format(job_group['name']))

    def run_job_groups(self, job_groups):
        job_group_map = {}
        groups_thread_pool = ThreadPool(len(job_groups))
        job_group_param = [{'job_group':job_group, 'job_group_map':job_group_map} for job_group in job_groups]
        groups_thread_pool.map(self.run_job_group, job_group_param)
        groups_thread_pool.close()
        groups_thread_pool.join()

def run_import(importInfo):
    print('import {}'.format(importInfo.name))

def main():
    fin = open('jobs.yaml', 'r', encoding='utf-8')
    json_content = yaml.load(fin)
    task_list = json_content['Tasks']

    for task in task_list:
        print(task['name'], '|', task['mode'], '|', task.get('type', 'None'), '|', task.get('file', 'None'))
        task_mode = task['mode'].lower()
        if(task_mode == 'jobgroup'):
            JobProcessor().run_job_groups(task['JobGroups'])
        elif(task_mode == 'import'):
            run_import(ImportInfo(task['name'], task['type'], task['file']))
        elif(task_mode == 'say'):
            print('say {}'.format(task['name']))
        elif(task_mode == 'wait'):
            print('wait {}'.format(task['name']))

if __name__ == '__main__':
    main()
```

# 最后

具体实现在真正写的时候需要再调整一下，而且感觉代码还可以优化，感觉目前并不是最好的实现方式，或者说写法有点怪。。。

---

参考：

- [python 队列 Queue](https://www.cnblogs.com/itogo/p/5635629.html)
- [Python 操作 yaml 文件](https://www.cnblogs.com/xinjing-jingxin/p/9128293.html)
