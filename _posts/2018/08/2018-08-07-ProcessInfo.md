---
layout:     post
title:      Process Info
subtitle:   Information e.g.Memory usage of single process
date:       2018-08-07
author:     BF
header-img: img/bf/light_01.jpg
catalog: true
tags:
    - python
    - tasklist
---
# Process Info

之前的测试中，使用`psutil`可以得到系统的`CPU/Memory`的使用百分比，但是没试过读取个别进程的信息。现在想要实现这样的功能，方便更好的分析`java.exe`的内存使用变化。

有一种思路是使用系统本身提供的查看进程的`shell cmd`和`subprocess.check_output`组合得到信息。

## tasklist

在`windows`系统中，提供了`tasklist`来查看`process`的信息，大致的结果如下：

```shell
C:\Users\mayn>tasklist

映像名称                       PID 会话名              会话#       内存使用
========================= ======== ================ =========== ============
System Idle Process              0 Services                   0          8 K
System                           4 Services                   0        108 K
Registry                       108 Services                   0     22,924 K
smss.exe                       376 Services                   0      1,128 K
csrss.exe                      580 Services                   0      5,124 K
wininit.exe                    672 Services                   0      6,676 K
```

查看`tasklist`的使用帮助，我们发现可以利用本身提供的参数取得想要的数据。

```shell
C:\Users\mayn>tasklist /?

TASKLIST [/S system [/U username [/P [password]]]]
         [/M [module] | /SVC | /V] [/FI filter] [/FO format] [/NH]

描述:
    该工具显示在本地或远程机器上当前运行的进程列表。


参数列表:
   /S     system           指定连接到的远程系统。

   /U     [domain\]user    指定应该在哪个用户上下文执行这个命令。

   /P     [password]       为提供的用户上下文指定密码。如果省略，则
                           提示输入。

   /M     [module]         列出当前使用所给 exe/dll 名称的所有任务。
                           如果没有指定模块名称，显示所有加载的模块。

   /SVC                    显示每个进程中主持的服务。

   /APPS 显示 Microsoft Store 应用及其关联的进程。

   /V                      显示详细任务信息。

   /FI    filter           显示一系列符合筛选器
                           指定条件的任务。

   /FO    format           指定输出格式。
                           有效值: "TABLE"、"LIST"、"CSV"。

   /NH                     指定列标题不应该
                           在输出中显示。
                           只对 "TABLE" 和 "CSV" 格式有效。

   /?                      显示此帮助消息。

筛选器:
    筛选器名称     有效运算符           有效值
    -----------     ---------------           --------------------------
    STATUS          eq, ne                    RUNNING | SUSPENDED
                                              NOT RESPONDING | UNKNOWN
    IMAGENAME       eq, ne                    映像名称
    PID             eq, ne, gt, lt, ge, le    PID 值
    SESSION         eq, ne, gt, lt, ge, le    会话编号
    SESSIONNAME     eq, ne                    会话名称
    CPUTIME         eq, ne, gt, lt, ge, le    CPU 时间，格式为
                                              hh:mm:ss。
                                              hh - 小时，
                                              mm - 分钟，ss - 秒
    MEMUSAGE        eq, ne, gt, lt, ge, le    内存使用(以 KB 为单位)
    USERNAME        eq, ne                    用户名，格式为
                                              [域\]用户
    SERVICES        eq, ne                    服务名称
    WINDOWTITLE     eq, ne                    窗口标题
    模块         eq, ne                    DLL 名称

注意: 当查询远程计算机时，不支持 "WINDOWTITLE" 和 "STATUS"
      筛选器。

Examples:
    TASKLIST
    TASKLIST /M
    TASKLIST /V /FO CSV
    TASKLIST /SVC /FO LIST
    TASKLIST /APPS /FI "STATUS eq RUNNING"
    TASKLIST /M wbem*
    TASKLIST /S system /FO LIST
    TASKLIST /S system /U 域\用户名 /FO CSV /NH
    TASKLIST /S system /U username /P password /FO TABLE /NH
    TASKLIST /FI "USERNAME ne NT AUTHORITY\SYSTEM" /FI "STATUS eq running"
```

以`vscode`的进程`code.exe`为例，可以筛选出对应的进程：

```shell
C:\Users\mayn>tasklist /FI "imagename eq code.exe"

映像名称                       PID 会话名              会话#       内存使用
========================= ======== ================ =========== ============
Code.exe                      1492 Console                    1    146,248 K
Code.exe                      9176 Console                    1     69,080 K
Code.exe                      9120 Console                    1     11,412 K
Code.exe                      1700 Console                    1     81,984 K
Code.exe                      5784 Console                    1    221,540 K
Code.exe                      9464 Console                    1    106,296 K
Code.exe                      8392 Console                    1     98,076 K
```

筛选器是可以组合的，比如：
`"tasklist  /FI "imagename eq java.exe" /FI \"username eq xiche\""`
这样就可以得到自己想要的进程信息，并取得一些值。
## subprocess.check_out

`subprocess`是python中调用进程与进程交互的一个库，可以用它来调用运行程序。

这边我们使用`check_out`来得到`tasklist`的输出结果，然后利用正则表达式取得对应的值，组装成字典列表来供之后使用。

```python
import re
import subprocess

def get_processes_running():
    tasks = subprocess.check_output(
        "tasklist  /FI \"imagename eq code.exe\" ").decode("gbk").split("\r\n")
    p = []
    for task in tasks:
        m = re.match("(.+?) +(\d+) (.+?) +(\d+) +(\d+.* K).*", task)
        if m is not None:
            p.append({"image": m.group(1),
                      "pid": m.group(2),
                      "session_name": m.group(3),
                      "session_num": m.group(4),
                      "mem_usage": m.group(5)
                      })
    return p

processes = get_processes_running()

for process in processes:
    print(process["pid"], process["mem_usage"])
```

运行的结果如下:

```shell
PS C:\Users\mayn\Desktop> python test.py
1492 157,088 K
9176 75,932 K
9120 11,416 K
1700 119,968 K
2868 237,328 K
2616 121,216 K
5580 33,724 K
10688 103,604 K
```

## psutil

可能大家发现了，这样使用tasklist取得的值是固定了，而我们想得到更多的值(e.g cpu)需要使用更多的命令来得到信息。

但其实还有一种方便的方法，直接使用`psutil`，用他来获取系统信息真的很强很方便。

```python
def processinfo(processName):
    pids = psutil.pids()
    pids_return = []
    for pid in pids:
        p = psutil.Process(pid)
        if processName.lower() in p.name().lower() :
            pids_return.append(pid)
    return pids_return

for pid in processinfo("code.exe"):
    ps = psutil.Process(pid)  
    print(ps.cpu_percent(interval=1.0), ps.memory_percent())
```

更多信息请参考：[https://github.com/bearfly1990/PowerScript/wiki/Python3](https://github.com/bearfly1990/PowerScript/wiki/Python3)
