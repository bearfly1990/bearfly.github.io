---
layout: post
title: Python Screen Gif (02)
subtitle: Python Record Screen to Gif
date: 2018-10-24
author: BF
header-img: img/bf/sun_02.jpg
catalog: true
tags:
  - python
  - imageio
  - PIL
---

# Background

上一篇把截图做成视频，再把视频转回 gif 似乎有点画蛇添足，今天就考虑直接把截图转成 gif

# Code

```python
import os, time
import imageio
from PIL import ImageGrab

images = []
start_time = time.time()
while True:
    im = ImageGrab.grab()
    im.save('temp.png')
    #save image to the list.
    images.append(imageio.imread('temp.png'))
    time.sleep(0.001)
    end_time = time.time()
    if(end_time - start_time > 5):
        break
imageio.mimsave('test.gif', images, duration=0.3)
c1.write_gif('{}.gif'.format(OUTPUT_NAME))
```

# Next

- 怎么使用 GUI 来录制固定区域的屏幕
- 是否可以提高性能，每次截图放到 list 中性能消耗比较大
- ...
  更多信息可以参考：

[Python Screen Gif (01)](https://bearfly1990.github.io/2018/10/22/PyScreenGif/)

[https://www.cnblogs.com/dcb3688/p/4608048.html](https://www.cnblogs.com/dcb3688/p/4608048.html)
