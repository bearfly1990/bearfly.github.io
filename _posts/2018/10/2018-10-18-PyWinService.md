---
layout: post
title: Python Windows Service
subtitle: Register python script as windows service
date: 2018-10-18
author: BF
header-img: img/bf/sky_01.jpg
catalog: true
tags:
  - python
  - pywin32
---
# Background

最近在优化`LoadTest`的流程，在执行测试的时候，我们需要在10台虚拟机上运行`HostProxy`, 这意味着需要登陆这10台机子并且一个个运行代理程序，这简直要人命。

所以我第一个想到的就是把这个过程做成一个`windows service`，让他们一直监控一个文件命令，当下达运行指令的时候，10台机子就会自动运行，同理也能停止。

虽然最后发现通过service启的进程不能被`Host Manager`访问到（有时间再仔细研究，应该有解决方案），用`Scheduler`替代了，但其它操作还是没有问题的。

# pywin32

如果想要编写`windows service`，就要用到`pywin32`这个库，可以从网上下到，我记得好像不能直接使用pip安装。

# Code

下面便是一个简单的例子，在服务启动之后，便会每5秒在文件中写数据。

主要我们自己的业务逻辑便是写在`SvcDoRun`这个函数中。

```python
import win32serviceutil
import win32service
import win32event
import servicemanager
import socket
import os, time

class PySvc(win32serviceutil.ServiceFramework):
    # you can NET START/STOP the service by the following name
    _svc_name_ = "PythonTestService"
    # this text shows up as the service name in the Service
    # Control Manager (SCM)
    _svc_display_name_ = "Python Test Service"
    # this text shows up as the description in the SCM
    _svc_description_ = "This service writes stuff to a file"

    def __init__(self, args):
        win32serviceutil.ServiceFramework.__init__(self,args)
        # create an event to listen for stop requests on
        self.hWaitStop = win32event.CreateEvent(None, 0, 0, None)

    # core logic of the service
    def SvcDoRun(self):
        # import servicemanager
        f = open('c:\\test.txt', 'w+')
        rc = None
        # if the stop event hasn't been fired keep looping
        while rc != win32event.WAIT_OBJECT_0:
            f.write('TEST DATA\n')
            f.flush()
            # block for 5 seconds and listen for a stop event
            rc = win32event.WaitForSingleObject(self.hWaitStop, 5000)

        f.write('Service is stopped.\n')
        f.close()

    # called when we're being shut down
    def SvcStop(self):
        # tell the SCM we're shutting down
        self.ReportServiceStatus(win32service.SERVICE_STOP_PENDING)
        # fire the stop event
        win32event.SetEvent(self.hWaitStop)

if __name__ == '__main__':
    win32serviceutil.HandleCommandLine(PySvc)
```

# Usage

在编写好脚本之后，比如上面的代码假设文件名为`PythonService.py`,那么就可以用以下的方式来使用。

当然，在第一部安装之后，可以直接在`service`面板操作，不一定需要用脚本。

```python

#1.安装服务
python PythonService.py install
#2.让服务自动启动
python PythonService.py --startup auto install 
#3.启动服务
python PythonService.py start
#4.重启服务
python PythonService.py restart
#5.停止服务
python PythonService.py stop
#6.删除/卸载服务
python PythonService.py remove
```

更多信息可以参考：

[https://www.cnblogs.com/zoro-robin/p/6110188.html](https://www.cnblogs.com/zoro-robin/p/6110188.html)
