---
layout: post
title: Text Color Of Console
subtitle: Define text color in console
date: 2018-04-21
author: BF
header-img: img/bf/camera_01.jpg
catalog: true
tags:
    - console
---
## Console输出彩色字体
在平常工作中会使用到console输出日志或者字符，如果有不同的颜色的话就非常直观。
比如Error用红色显示，Warn用黄色显示，Info用默认的白色，Success或者Pass的话就使用绿色。

终端的字符颜色是用转义序列控制的，是文本模式下的系统显示功能，和具体的语言无关,只要终端支持就好。转义序列是以ESC开头,用\033来表示（ESC的ASCII码八进制:033, 十进制:27, 十六进制:1B）。

### 书写格式：
```
\033[%showMode%;%fontcolor%;%bgcolor%m %content%\033[0m
```
开头部分`\033[parm1;parm2;parm3 m`：
注意：开头部分的三个参数：显示方式，前景色，背景色是可选参数，他们对应的值都是不一样的，所以可以只写一个，顺序也没有要求，但是建议按照默认的格式规范书写。

结尾可以省略，但是为了书写规范，建议`\033[*;*;*m`开头，`\033[0m`结尾。

数值表示的参数含义：

| 显示方式       | 前景色      | 背景色    |

| ------------- | ---------- | ---------- |
| 0（默认值）    | 30（黑色）  | 40（黑色）  |
| 1（高亮）      | 31（红色）  | 41（红色）  |
| 4（下划线）    | 32（绿色）  | 42（绿色）  |
| 5（闪烁）      | 33（黄色）  | 43（黄色）  |
| 7（反显）      | 34（蓝色）  | 44（蓝色）  |
| 22（非粗体）   | 35（洋 红） | 45（洋 红） |
| 24（非下划线） | 36（青色）  | 46（青色）  |
| 25（非闪烁）   | 37（白色）  | 47（白色）  |
| 27（非反显）   |    -       |     -      |

常见开头格式：

| 开头格式      | 效果                                             |
| ------------- | ------------------------------------------------ |
| \033[0m       | 默认字体正常显示，不高亮                         |
| \033[32;0m    | 红色字体正常显示                                 |
| \033[1;32;40m | 显示方式: 高亮    字体前景色：绿色  背景色：黑色 |
| \033[1;31;40m | 显示方式: 高亮    字体前景色：红色  背景色：黑色 |
| \033[1;35;40m | 显示方式: 高亮    字体前景色：洋红  背景色：黑色 |
| \033[1;33;40m | 显示方式: 高亮    字体前景色：黄色  背景色：黑色 |

Python实例：
```python
print("\033[1;35;40m 高亮洋红背景黑 \033[0m")
```
Java实例:
```java
public static final String ANSI_RESET = "\u001B[0m";
public static final String ANSI_BLACK = "\u001B[30m";
public static final String ANSI_RED = "\u001B[31m";
public static final String ANSI_GREEN = "\u001B[32m";
public static final String ANSI_YELLOW = "\u001B[33m";
public static final String ANSI_BLUE = "\u001B[34m";
public static final String ANSI_PURPLE = "\u001B[35m";
public static final String ANSI_CYAN = "\u001B[36m";
public static final String ANSI_WHITE = "\u001B[37m";

public static final String ANSI_BLACK_BACKGROUND = "\u001B[40m";
public static final String ANSI_RED_BACKGROUND = "\u001B[41m";
public static final String ANSI_GREEN_BACKGROUND = "\u001B[42m";
public static final String ANSI_YELLOW_BACKGROUND = "\u001B[43m";
public static final String ANSI_BLUE_BACKGROUND = "\u001B[44m";
public static final String ANSI_PURPLE_BACKGROUND = "\u001B[45m";
public static final String ANSI_CYAN_BACKGROUND = "\u001B[46m";
public static final String ANSI_WHITE_BACKGROUND = "\u001B[47m";

System.out.println(ANSI_GREEN_BACKGROUND + "This text has a green background but default text!" + ANSI_RESET);
System.out.println(ANSI_RED + "This text has red text but a default background!" + ANSI_RESET);
System.out.println(ANSI_GREEN_BACKGROUND + ANSI_RED + "This text has a green background and red text!" + ANSI_RESET);
```
