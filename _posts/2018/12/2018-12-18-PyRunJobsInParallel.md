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
```

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

    def run_job_group(self, job_group, job_goup_map):
        
        pregroup = job_group.get('pregroup', -1)
        jobs = job_group['Jobs']
        mode = job_group.get('mode', 'parallel')
        # add jobs to the queu
        jobs_num = len(jobs)
        jobs_queue = queue.Queue(jobs_num)
        job_goup_map[job_group['id']] = jobs_queue
        for job in jobs:
            jobs_queue.put(Job(job['jobid'], job['name']))
        # wait pre group run finished if have pregroup
        if(pregroup != -1):
            while(True):
                pre_queue = job_goup_map.get(pregroup, None)
                if pre_queue:
                    pre_queue.join()
                    break
                time.sleep(1)
        
        # print('### start run jobgroup:{} ###'.format(job_group['name']))
        self.queue = jobs_queue
        if(mode == 'parallel'):
            self.run_jobs_parallel()
            # JobProcessor(jobs_queue).run_jobs_parallel()
        else:
            self.run_jobs_serial()
            # JobProcessor(jobs_queue).run_jobs_serial()
        print('### end run jobgroup:{} ###'.format(job_group['name']))

    def run_job_groups(self, job_groups):
        job_goup_map = {}
        for job_group in job_groups:
            run_job_group_thread = threading.Thread(
                target=self.run_job_group, args=(job_group, job_goup_map))
            run_job_group_thread.start()

def run_import(importInfo):
    print('import {}'.format(importInfo.name))

def main():
    fin = open('JobSchedule_BatchServer.yaml', 'r', encoding='utf-8')
    json_content = yaml.load(fin)
    task_list = json_content['Tasks']

    for task in task_list:
        print(task['name'], '|', task['mode'], '|', task.get(
            'type', 'None'), '|', task.get('file', 'None'))
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

具体实现在真正写的时候需要再调整一下。

---

参考：

- [python 队列 Queue](https://www.cnblogs.com/itogo/p/5635629.html)
- [Python 操作 yaml 文件](https://www.cnblogs.com/xinjing-jingxin/p/9128293.html)
