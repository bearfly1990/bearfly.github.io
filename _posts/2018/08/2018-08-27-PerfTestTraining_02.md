---
layout: post
title: Performance Test Training (2)
subtitle: Performance Test Training Day 2 at SH
date: 2018-08-27
author: BF
header-img: img/bf/flower_02.jpg
catalog: true
tags:
  - test
  - training
---

# Performance Test Training (2)

性能测试培训的第二天，主要内容是性能测试一些实例，这些要总结好难，还是要实际操作。

核心的规则还是从进程，到线程，CPU，内存，堆栈，IO，网络一步步分析。

培训上基本上是都是 linux 的命令，我下面简单罗列了一些window下的命令，而且很不全，参考意义不大。

之后有时候再一块一块深入，每一块都有好多细节。。。今天就算给自己交个差吧（捂脸）

## Performance Monitor [(Link)](https://www.cnblogs.com/cappuccino917/p/6699418.html)

性能计数器是windows提供的核心监控数据，可以根据自己需要去获取。

Processor: `% Processor Time` CPU当前利用率，百分比

Memory: `Available MBytes` 当前可用内存，兆字节（虚拟内存不需要监控，只有当物理内存不够时才会使用虚拟内存，物理内存已有监控）

LogicalDisk: `% Free Space` 逻辑分区可用空间，百分比（物理磁盘IO由于RAID级别不同，或者有的机器没有RAID，无法定义统一的监控阈值）

Network Interface: `Bytes Total/sec` 网卡流量：发送+接收，字节


TCPv4: `Connections Established` 当前连接数（Established + Close-Wait）

### CPU:

#### %Processor Time

`%Priviliaged Time` CPU在特权模式下处理线程所花的时间百分比。一般的系统服务，进城管理，内存管理等一些由操作系统自行启动的进程属于这类

`%User Time` 与`%Privileged Time`计数器正好相反，指的是在用户状态模式下（即非特权模式）的操作所花的时间百分比。如果该值较大，可以考虑是否通过算法优化等方法降低这个值。如果该服务器是数据库服务器，导致此值较大的原因很可能是数据库的排序或是函数操作消耗了过多的CPU时间，此时可以考虑对数据库系统进行优化。

`%DPC Time` 处理器在网络处理上消耗的时间，该值越低越好。在多处理器系统中，如果这个值大于50%并且`%Processor Time`非常高，加入一个网卡可能会提高性能。

### Memory:

#### Available Bytes

**Pages/sec**该计数器显示由于页面不在物理内存中而需要从磁盘读取的页面数。`Pages/sec` 的值很大不一定表明内存有问题，而可能是运行使用内存映射文件的程序所致，操作系统经常会利用磁盘交换的方式提高系统可用的内存量或是提高内存的使用效率。（注意该计数器与 Page Faults/sec 的区别，后者只表明数据不能在内存的指定工作集中立即使用，包括硬错误和软错误）

**Page Faults/sec**计数器可以确保磁盘活动不是由分页导致的。在 Windows 中，换页的原因包括：配置进程占用了过多内存 或者 文件系统活动。

如果在同一硬盘上有多个逻辑分区，需要使用 `Logical Disk`计数器而非 `Physical Disk`计数器。查看逻辑磁盘计数器有助于确定哪些文件被频繁访问。当发现磁盘有大量读/写活动时，请查看读写专用计数器以确定导致每个逻辑卷负荷增加的磁盘活动类型，例如，Logical Disk: `Disk Write Bytes/sec`。

**Page Input/sec**表示为了解决硬错误而写入硬盘的页数（参考值：>=`Page Reads/sec`）

**Page Reads/sec** 表示为了解决硬错误而从硬盘上读取的页数。（参考值： <=5）

如果怀疑有内存泄露，请监视 `Memory/Available Bytes` 和 `Memory/ Committed Bytes`，以观察内存行为，并监视你认为可能在泄露内存的进程的 `Process/ Private Bytes`、`Process/ Working Set` 和`Process/ Handle Count`。如果怀疑是内核模式进程导致了泄露，则还应该监视 `Memory/ Pool Nonpaged Bytes`、`Memory/ Pool Nonpaged Allocs` 和 `Process(process_name)/ Pool Nonpaged Bytes`

如果发生了内存泄漏,`process\private bytes`计数器和`process\working set` 计数器的值往往会升高,同时`avaiable bytes`的值会降低

`private Bytes` 是指进程所分配的无法与其他进程共享的当前字节数量。该计数器主要用来判断进程在性能测试过程中有无内存泄漏。

例如：对于一个IIS之上的web应用，我们可以重点监控inetinfo进程的`Private Bytes`，如果在性能测试过程中，该进程的`Private Bytes`计数器值不断增加，或是性能测试停止后一段时间，该进程的`Private Bytes`仍然持续在高水平，则说明应用存在内存泄漏。

### Disk：

#### PhysicalDisk\Avg. Disk sec/Read

以秒计算的在此盘上读取数据的所需平均时间。

#### Physical Disk\ Disk Reads/sec

在读取操作时从磁盘上传送的字节平均数。

#### PhysicalDisk\ Avg. Disk sec/Write

以秒计算的在此盘上写入数据的所需平均时间。

#### Physical Disk\ DiskWrites/sec

在写入操作时从磁盘上传送的字节平均数。

#### Physical Disk\ Avg.Disk sec/Transfer

反映磁盘完成请求所用的时间。较高的值表明磁盘控制器由于失败而不断重试该磁盘。这些故障会增加平均磁盘传送时间。

#### %Disk Time 和 Avg.Disk Queue Length

RAID 磁盘中的 `% Disk Time` 计数器会指示大于 100% 的值。如果出现这种情况，则使用 `PhysicalDisk: Avg.Disk Queue Length`计数器来确定等待进行磁盘访问的平均系统请求数量。

如果不是RAID，则使用 `% Disk Time` 和 `Current Disk Queue Length`计数器确定是否磁盘存在瓶颈，如果这两个计数器的值一直很高，则可能是磁盘存在瓶颈

Physical Disk： `DiskTransfers/sec` 磁盘`IOPS % Disk Time` 当前物理磁盘利用率，如果是RAID，该值会大于100% `Current Disk Queue Length` 等待进行磁盘访问的当前系统请求数量 `Avg.Disk Queue Length` 等待进行磁盘访问的平均系统请求数量，用于RAID

## Task

### tasklist
```shell
PS C:\Users\mayn\Desktop> tasklist

映像名称                       PID 会话名              会话#       内存使用
========================= ======== ================ =========== ============
System Idle Process              0 Services                   0          8 K
System                           4 Services                   0        128 K
Registry                       108 Services                   0     13,504 K
smss.exe                       376 Services                   0      1,168 K
csrss.exe                      580 Services                   0      4,932 K
wininit.exe                    672 Services                   0      6,576 K
csrss.exe                      680 Console                    1      5,368 K
```

### tskill/taskkill
```shell
tskill notepad
```
## CPU

### wmic cpu

上面显示的有位宽，最大始终频率， 生产厂商，二级缓存等信息

```shell
PS C:\Users\mayn\Desktop> wmic cpu
AddressWidth  Architecture  AssetTag                Availability  Caption                                 Characteristics  ConfigManagerErrorCode  ConfigManagerUserConfig  CpuStatus  CreationClassName  CurrentClockSpeed  CurrentVoltage  DataWidth  Description                             DeviceID  ErrorCleared  ErrorDescription  ExtClock  Family  InstallDate  L2CacheSize  L2CacheSpeed  L3CacheSize  L3CacheSpeed  LastErrorCode  Level  LoadPercentage  Manufacturer  MaxClockSpeed  Name                                     NumberOfCores  NumberOfEnabledCore  NumberOfLogicalProcessors  OtherFamilyDescription  PartNumber              PNPDeviceID  PowerManagementCapabilities  PowerManagementSupported  ProcessorId       ProcessorType  Revision  Role  SecondLevelAddressTranslationExtensions  SerialNumber            SocketDesignation  Status  StatusInfo  Stepping  SystemCreationClassName  SystemName  ThreadCount  UniqueId  UpgradeMethod  Version  VirtualizationFirmwareEnabled  VMMonitorModeExtensions  VoltageCaps
64            9             To Be Filled By O.E.M.  3             Intel64 Family 6 Model 158 Stepping 10  236                                                               1          Win32_Processor    3000               10              64         Intel64 Family 6 Model 158 Stepping 10  CPU0                                      100       205                  1536                       9216         0                            6      13              GenuineIntel  3000           Intel(R) Core(TM) i5-8500 CPU @ 3.00GHz  6              6                    6                                                  To Be Filled By O.E.M.                                            FALSE                     BFEBFBFF000906EA  3                        CPU   TRUE                                     To Be Filled By O.E.M.  LGA1151            OK      3                     Win32_ComputerSystem     PC-CX       6                      1                       FALSE                          TRUE
```

## Memory

### wmic memorychip

可以显示出来 2 条内存，各 8G 速度 2666MHz

```shell
PS C:\Users\mayn\Desktop> wmic memorychip
Attributes  BankLabel  Capacity    Caption   ConfiguredClockSpeed  ConfiguredVoltage  CreationClassName     DataWidth  Description  DeviceLocator   FormFactor  HotSwappable  InstallDate  InterleaveDataDepth  InterleavePosition  Manufacturer  MaxVoltage  MemoryType  MinVoltage  Model  Name      OtherIdentifyingInfo  PartNumber       PositionInRow  PoweredOn  Removable  Replaceable  SerialNumber  SKU  SMBIOSMemoryType  Speed  Status  Tag                TotalWidth  TypeDetail  Version
1           BANK 0     8589934592  物理内存   2666                  1200               Win32_PhysicalMemory  64          物理内存     ChannelA-DIMM1  8                                      1                    1                   G-Skill       0           0           0                  物理内存                        F4-2666C19-8GNT                                                    00000000           26                2666           Physical Memory 0  64          128
1           BANK 2     8589934592  物理内存   2666                  1200               Win32_PhysicalMemory  64          物理内存     ChannelB-DIMM1  8                                      1                    2                   G-Skill       0           0           0                  物理内存                        F4-2666C19-8GNT                                                    00000000           26                2666           Physical Memory 1  64          128
```

## Disk

### wmic logicaldisk

可以看到有几个盘，每一个盘的文件系统和剩余空间

```shell
PS C:\Users\mayn\Desktop> wmic logicaldisk
Access  Availability  BlockSize  Caption  Compressed  ConfigManagerErrorCode  ConfigManagerUserConfig  CreationClassName  Description   DeviceID  DriveType  ErrorCleared  ErrorDescription  ErrorMethodology  FileSystem  FreeSpace     InstallDate  LastErrorCode  MaximumComponentLength  MediaType  Name  NumberOfBlocks  PNPDeviceID  PowerManagementCapabilities  PowerManagementSupported  ProviderName  Purpose  QuotasDisabled  QuotasIncomplete  QuotasRebuilding  Size           Status  StatusInfo  SupportsDiskQuotas  SupportsFileBasedCompression  SystemCreationClassName  SystemName  VolumeDirty  VolumeName  VolumeSerialNumber
0                                C:       FALSE                                                        Win32_LogicalDisk  本地固定磁盘   C:        3                                                            NTFS        74612744192                               255                     12         C:                                                                                                                                                                   106530430976                       FALSE               TRUE                          Win32_ComputerSystem     PC-CX                                04C2790D
0                                D:       FALSE                                                        Win32_LogicalDisk  本地固定磁盘   D:        3                                                            NTFS        342652858368                              255                     12         D:                                                                                                                                                                   372486508544                       FALSE               TRUE                          Win32_ComputerSystem     PC-CX                                00004823
0                                E:       FALSE                                                        Win32_LogicalDisk  本地固定磁盘   E:        3                                                            NTFS        364452909056                              255                     12         E:                                                                                                                                                                   1000202768384                      FALSE               TRUE                          Win32_ComputerSystem     PC-CX                                00004823
```

### wmic volume

```shell
PS C:\Users\mayn\Desktop> wmic volume
Access  Automount  Availability  BlockSize  BootVolume  Capacity       Caption                                            Compressed  ConfigManagerErrorCode  ConfigManagerUserConfig  CreationClassName  Description  DeviceID                                           DirtyBitSet  DriveLetter  DriveType  ErrorCleared  ErrorDescription  ErrorMethodology  FileSystem  FreeSpace     IndexingEnabled  InstallDate  Label  LastErrorCode  MaximumFileNameLength  Name                                               NumberOfBlocks  PageFilePresent  PNPDeviceID  PowerManagementCapabilities  PowerManagementSupported  Purpose  QuotasEnabled  QuotasIncomplete  QuotasRebuilding  SerialNumber  Status  StatusInfo  SupportsDiskQuotas  SupportsFileBasedCompression  SystemCreationClassName  SystemName  SystemVolume
        TRUE                     4096       TRUE        106530430976   C:\                                                FALSE                                                                                        \\?\Volume{81b9fa95-b225-44ef-84fe-e6b4bcd49f21}\               C:           3                                                            NTFS        74612363264   TRUE                                                255                    C:\                                                                TRUE                                                                                                                                             79853837                          TRUE                TRUE                                                   PC-CX       FALSE
        TRUE                     4096       FALSE       844099584      \\?\Volume{d9656dfe-1091-41c2-b677-aed0eda4ea25}\  FALSE                                                                                        \\?\Volume{d9656dfe-1091-41c2-b677-aed0eda4ea25}\                            3                                                            NTFS        369049600     TRUE                                                255                    \\?\Volume{d9656dfe-1091-41c2-b677-aed0eda4ea25}\                  FALSE                                                                                                                                            2759807823                        TRUE                TRUE                                                   PC-CX       FALSE
        TRUE                     4096       FALSE       372486508544   D:\                                                FALSE                                                                                        \\?\Volume{da460ecc-d1db-4ffc-8fd3-9445c2154bf4}\               D:           3                                                            NTFS        342652858368  TRUE                                                255                    D:\                                                                FALSE                                                                                                                                            18467                             TRUE                TRUE                                                   PC-CX       FALSE
        TRUE                     4096       FALSE       1000202768384  E:\                                                FALSE                                                                                        \\?\Volume{673bf2eb-d8ce-4689-98fe-add079591f66}\               E:           3                                                            NTFS        364452909056  TRUE                                                255                    E:\                                                                FALSE                                                                                                                                            18467                             TRUE                TRUE                                                   PC-CX       FALSE
        TRUE                     2048       FALSE       102539264      \\?\Volume{e9b34647-3273-4702-9f11-7296af3707f2}\                                                                                               \\?\Volume{e9b34647-3273-4702-9f11-7296af3707f2}\                            3                                                            FAT         76158976                                                          255                    \\?\Volume{e9b34647-3273-4702-9f11-7296af3707f2}\                  FALSE                                                                                                                                            18467                             FALSE               FALSE                                                  PC-CX       TRUE
```

每个盘的剩余空间量，其实上一个命令也可以查看的

```shell
PS C:\Users\mayn\Desktop> fsutil volume diskfree c:
可用字节总数              : 74612154368
字节总数                  : 106530430976
可用的尚未使用的字节总数  : 74612154368
```

这个命令查看每一个卷的容量信息是很方便

## BIOS

### wmic bios
```shell
PS C:\Users\mayn\Desktop> wmic bios
BiosCharacteristics                                                              BIOSVersion                                                  BuildNumber  Caption  CodeSet  CurrentLanguage  Description  EmbeddedControllerMajorVersion  EmbeddedControllerMinorVersion  IdentificationCode  InstallableLanguages  InstallDate  LanguageEdition  ListOfLanguages                                                                                                                                                      Manufacturer              Name  OtherTargetOS  PrimaryBIOS  ReleaseDate                SerialNumber          SMBIOSBIOSVersion  SMBIOSMajorVersion  SMBIOSMinorVersion  SMBIOSPresent  SoftwareElementID  SoftwareElementState  Status  SystemBiosMajorVersion  SystemBiosMinorVersion  TargetOperatingSystem  Version
{7, 10, 11, 12, 15, 16, 17, 19, 23, 24, 25, 26, 27, 28, 29, 32, 33, 40, 42, 43}  {"ALASKA - 1072009", "0401", "American Megatrends - 5000D"}               0401              en|US|iso8859-1  0401         255                             255                                                 9                                                   {"en|US|iso8859-1", "fr|FR|iso8859-1", "zh|TW|unicode", "zh|CN|unicode", "ja|JP|unicode", "de|DE|iso8859-1", "es|ES|iso8859-1", "ru|RU|iso8859-5", "ko|KR|unicode"}  American Megatrends Inc.  0401                 TRUE         20180319000000.000000+000  System Serial Number  0401               3                   1                   TRUE           0401               3                     OK      5                       13                      0                      ALASKA - 1072009
```
