---
layout: post
title: Python Screen Gif (1)
subtitle: Python Record Screen to Gif
date: 2018-10-22
author: BF
header-img: img/bf/sun_02.jpg
catalog: true
tags:
  - python
  - opencv
  - moviepy
---

# Background

最近用了一些 gif 生成工具，感觉挺好用的，就想着 python 是不是也可以实现，自己做一个。

浏览了一些文章，发现有一些现成的库可以用。

最终的想法还是做成一个 GUI，今天是第一步，思路利用`PIL`的`ImageGrab`抓取屏幕，然后使用`opencsv`写入视频流，再用`moviepy`截取视频的画面生成 gif。

# Lib Installed

下面是主要用的库，`pillow`就是`PIL`， `opencv-python`就是`cv2`。

在使用`moviepy`的时候，会需要下载第三方 exe [ffmpeg-win32-v3.2.4.exe](https://github.com/imageio/imageio-binaries/raw/master/ffmpeg/ffmpeg-win32-v3.2.4.exe)

```batch
pip install pillow
pip install opencv-python
pip install moviepy
```

# Code

```python
import numpy as np
import cv2
import moviepy.editor as mpy
from PIL import ImageGrab

OUTPUT_NAME = 'test'

screen_grabed = ImageGrab.grab()  # 获得当前屏幕
k = np.zeros((200, 200), np.uint8)
width, height = screen_grabed.size  # 获得当前屏幕的大小

fourcc = cv2.VideoWriter_fourcc(*'XVID')  # 编码格式

video = cv2.VideoWriter('{}.avi'.format(
    OUTPUT_NAME), fourcc, 16, (width, height))  # 输出文件命名为test.avi,帧率为16，可以自己设置

while True:
    im = ImageGrab.grab()
    imm = cv2.cvtColor(np.array(im), cv2.COLOR_RGB2BGR)  # 转为opencv的BGR格式
    video.write(imm)
    cv2.imshow('imm', k)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
video.release()
cv2.destroyAllWindows()

#import imageio
# imageio.plugins.ffmpeg.download()
# 视频文件的本地路径
content = mpy.VideoFileClip('{}.avi'.format(OUTPUT_NAME))
# 剪辑0分1秒到0分6秒的片段。注意：不使用resize则不会修改清晰度
c1 = content.subclip((0, 1), (0, 6)).resize((960, 640))
# 将片段保存为gif图到python的默认路径
c1.write_gif('{}.gif'.format(OUTPUT_NAME))
```

# Next

接下来，需要研究一下几个点：

- 怎么使用 GUI 来录制固定区域的屏幕
- 是否可以截取图片流，再直接把图片打包成 gif，省去中间生成 avi 的过程。
- ...

更多信息可以参考：

[https://blog.csdn.net/Spade\_/article/details/79516322](https://blog.csdn.net/Spade_/article/details/79516322)
[https://blog.csdn.net/zzzzjh/article/details/80903597](https://blog.csdn.net/zzzzjh/article/details/80903597)
