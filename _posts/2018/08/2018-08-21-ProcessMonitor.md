---
layout: post
title: System Monitor by Powershell
subtitle: System Process Monitor by Powershell Script
date: 2018-08-21
author: BF
header-img: img/bf/lake_01.jpg
catalog: true
tags:
  - powershell
  - monitor
---

# Process Monitor

先上代码:

```powershell
$CurrentDir = Split-Path $MyInvocation.MyCommand.Path
$file = "$CurrentDir/Monitor.csv"
$ps_name = "java"
$show_sys_cpu_rate = $false
$show_sys_mem_rate = $false

$memory_sys_total = (Get-WmiObject -Class Win32_PhysicalMemory |measure capacity -sum).Sum  #(gwmi win32_computersystem).TotalPhysicalMemory
$cpu_cores = (Get-WmiObject Win32_ComputerSystem).NumberOfLogicalProcessors
# $memory_sys_total = (Get-Counter "\Memory\System Driver Total Bytes").CounterSamples | Sort-Object Path
# $memory_sys_total = $memory_sys_total[0].CookedValue
"time,CPU(%),Memory(%),CPU_$ps_name(%), Memory_$ps_name(%)" >> $file
while ($True) {
    $cpu_rate_pss = (Get-Counter "\process($ps_name*)\% Processor Time").CounterSamples | Sort-Object Path
    $cpu_rate_sys = (Get-Counter "\processor(_total)\% processor time").CounterSamples | Sort-Object Path
    $cpu_rate_sys = $cpu_rate_sys[0].CookedValue
    if($show_sys_cpu_rate -eq $false){
        $cpu_rate_sys = 0
    }

    $memory_pss   = (Get-Counter "\Process($ps_name*)\Working Set - Private").CounterSamples | Sort-Object Path

    if($show_sys_mem_rate){
        $memory_sys_available   = (Get-Counter "\Memory\Available Bytes").CounterSamples | Sort-Object Path
        $memory_sys_available   = $memory_sys_available[0].CookedValue
        $memory_sys             = $memory_sys_total - $memory_sys_available
    }else{
        $memory_sys = 0
    }

    $cpu_rate_pss_total = 0
    $memory_pss_total = 0
    $cpu_rate_pss.Count
    try {
        For ($i = 0; $i -lt $cpu_rate_pss.Count; $i++) {
            $cpu_rate_pss_total  = $cpu_rate_pss_total + $cpu_rate_pss[$i].CookedValue
            $memory_pss_total    = $memory_pss_total + $memory_pss[$i].CookedValue
        }
    }catch{
        Write-Output ""
    }

    $cpu_rate_sys           = "{0:F2}" -f $cpu_rate_sys
    $cpu_rate_pss_total     = "{0:F2}" -f $cpu_rate_pss_total / $cpu_cores

    $memory_rate_sys        = "{0:F2}" -f ( $memory_sys / $memory_sys_total * 100)
    $memory_rate_pss_total  = "{0:F2}" -f ( $memory_pss_total / $memory_sys_total * 100)

    $time = Get-Date -format "MM/dd/yyyy HH:mm:ss"

    Write-Output "Time=$time;CPU=$cpu_rate_sys;Mem=$memory_rate_sys;CPU($ps_name)=$cpu_rate_pss_total;Mem($ps_name)=$memory_rate_pss_total"
    "$time,$cpu_rate_sys,$memory_rate_sys,$cpu_rate_pss_total,$memory_rate_pss_total" >> $file

    sleep 1
}
```

之前也有写过监测系统CPU和内存的`Powershell`脚本和`Python`脚本，不过当时都只有监测系统的CPU和内存的占比变化，没有针对进程。

最近在做测试的时候，已经在脚本中加入了`Python`对进程的操作，不过因为和具体项目结合在一起，还没有时间把代码抽象出来。今天本来想着在之前`Powershell`脚本的基础上改一下，不过没想到踩了一些坑，主要还是不熟悉。尤其如果对`C#`比较了解的话，写起来会方便很多。`Powershell`可以直接调用`.Net`的许多对象和方法。

## Main Knowledge

和之前[ProcessInfo](https://bearfly1990.github.io/2018/08/07/ProcessInfo/)中类似，`Powershell`也有许多方法去取得进程信息。可以利用系统本身的命令得到，而今天使用的，类似系统自身的`Performance Monitor`，通过`Get-Counter`方法来得到对应的计数器信息。

### Get-Counter

在`Powershell`运行窗体中，我们输入以下信息，可以得到系统的总的CPU使用率

```shell
PS C:\Users\mayn> Get-Counter "\processor(_total)\% processor time"

Timestamp                 CounterSamples
---------                 --------------
2018/8/21 22:41:37        \\pc-cx\processor(_total)\% processor time :
                          2.49161836524493
```

那如果我们想得到某个进程呢？
```shell
PS C:\Users\mayn> Get-Counter "\process(Code)\% processor time"

Timestamp                 CounterSamples
---------                 --------------
2018/8/21 22:54:45        \\pc-cx\process(code)\% processor time :
                          0
```

那多个同名进程呢？就如这VSCode的进程`code.exe`有好多个,就可以用通配符：
```shell
PS C:\Users\mayn> Get-Counter "\process(Code*)\% processor time"

Timestamp                 CounterSamples
---------                 --------------
2018/8/21 22:55:57        \\pc-cx\process(code#7)\% processor time :
                          0

                          \\pc-cx\process(code#6)\% processor time :
                          0

                          \\pc-cx\process(code#5)\% processor time :
                          0

                          \\pc-cx\process(code#4)\% processor time :
                          0

                          \\pc-cx\process(code#3)\% processor time :
                          0

                          \\pc-cx\process(code#2)\% processor time :
                          0

                          \\pc-cx\process(code#1)\% processor time :
                          0

                          \\pc-cx\process(code)\% processor time :
                          0

```

那么回过头来再看代码里，就思路很清晰了，就是利用`Get-Counter`来取得我们想要的信息，再处理，就可以得到我们想要的结果。

那怎么看我可以得到哪些信息呢？一个就是上网查，另一个是可以如下去得到相关的信息,你可以把`memory`替换成`process`等其它值来看有什么对应的信息可以获取。
```shell
PS C:\Users\mayn> (Get-Counter -ListSet  memory).paths
\Memory\Page Faults/sec
\Memory\Available Bytes
\Memory\Committed Bytes
\Memory\Commit Limit
\Memory\Write Copies/sec
\Memory\Transition Faults/sec
\Memory\Cache Faults/sec
\Memory\Demand Zero Faults/sec
\Memory\Pages/sec
\Memory\Pages Input/sec
\Memory\Page Reads/sec
\Memory\Pages Output/sec
\Memory\Pool Paged Bytes
\Memory\Pool Nonpaged Bytes
\Memory\Page Writes/sec
\Memory\Pool Paged Allocs
\Memory\Pool Nonpaged Allocs
\Memory\Free System Page Table Entries
\Memory\Cache Bytes
\Memory\Cache Bytes Peak
\Memory\Pool Paged Resident Bytes
\Memory\System Code Total Bytes
\Memory\System Code Resident Bytes
\Memory\System Driver Total Bytes
\Memory\System Driver Resident Bytes
\Memory\System Cache Resident Bytes
\Memory\% Committed Bytes In Use
\Memory\Available KBytes
\Memory\Available MBytes
\Memory\Transition Pages RePurposed/sec
\Memory\Free & Zero Page List Bytes
\Memory\Modified Page List Bytes
\Memory\Standby Cache Reserve Bytes
\Memory\Standby Cache Normal Priority Bytes
\Memory\Standby Cache Core Bytes
\Memory\Long-Term Average Standby Cache Lifetime (s)
```
请参考[Get-Counter](https://technet.microsoft.com/zh-cn/library/dd367892.aspx)

### CPU Cores and % Processor Time

细心的话，大家可能会发现，我在计算最后进程的cpu使用率的时候，除以了CPU的核心数。

这是因为在我们使用`"\process($ps_name*)\% Processor Time"`取得进程CPU使用率的时候，得到的是所有同名进程在不同`Processors/Cores`上使用率的总合。也就是说，比如我的机子是6核的，那么如果该进程用满了CPU，那么通过这个`CounterSet`得到的值将会是**600%**。

请参考：[Understanding Processor (% Processor Time) and Process (%Processor Time)](https://social.technet.microsoft.com/wiki/contents/articles/12984.understanding-processor-processor-time-and-process-processor-time.aspx)