---
layout: post
title: Python Screen Gif
subtitle: Python Record Screen to Gif
date: 2018-10-27
author: BF
header-img: img/bf/sun_02.jpg
catalog: true
tags:
  - python
  - imageio
  - PIL
  - tkinter
---
# 2018-10-22
## Background

最近用了一些 gif 生成工具，感觉挺好用的，就想着 python 是不是也可以实现，自己做一个。

浏览了一些文章，发现有一些现成的库可以用。

最终的想法还是做成一个 GUI，今天是第一步，思路利用`PIL`的`ImageGrab`抓取屏幕，然后使用`opencsv`写入视频流，再用`moviepy`截取视频的画面生成 gif。

## Lib Installed

下面是主要用的库，`pillow`就是`PIL`， `opencv-python`就是`cv2`。

在使用`moviepy`的时候，会需要下载第三方 exe [ffmpeg-win32-v3.2.4.exe](https://github.com/imageio/imageio-binaries/raw/master/ffmpeg/ffmpeg-win32-v3.2.4.exe)

```batch
pip install pillow
pip install opencv-python
pip install moviepy
```

## Code

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

## Next

接下来，需要研究一下几个点：

- 怎么使用 GUI 来录制固定区域的屏幕
- 是否可以截取图片流，再直接把图片打包成 gif，省去中间生成 avi 的过程。
- ...

# 2018-10-24
## Background

上一篇把截图做成视频，再把视频转回 gif 似乎有点画蛇添足，今天就考虑直接把截图转成 gif

## Code

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

## Next

- 怎么使用 GUI 来录制固定区域的屏幕
- 是否可以提高性能，每次截图放到 list 中性能消耗比较大
- ...

# 2018-10-27

## Background

今天更新了下代码，提高了可用性，增加了UI设定截屏的区域，有时间再更新这篇，已经基本可用了。

## Code

```python
import tkinter
import os, time, tempfile
import imageio
from PIL import ImageGrab
from threading import Thread
import msvcrt

output_gif_name = 'test.gif'
width_grab = 600
height_grab = 400
x_grab = 0
y_grab = 0
flag_stop_record = [False]
flag_is_recording = [False]

root = tkinter.Tk()
root.overrideredirect(True)
#root.attributes("-alpha", 0.3)窗口透明度70 %
root.attributes("-alpha", 0.3)#窗口透明度60 %
root.geometry("{}x{}+{}+{}".format(width_grab, height_grab, x_grab,y_grab))#"300x200+10+10"
canvas = tkinter.Canvas(root)
canvas.configure(width = width_grab)
canvas.configure(height = height_grab)
canvas.configure(bg = "blue")
canvas.grid(column=0,row=0)
# canvas.configure(highlightthickness = 0)
# canvas.pack()
# canvas.pack(side="bottom",fill="both",expand=True)
# x, y = 0, 0
def move(event):
    global x_grab,y_grab, width_grab, height_grab

    new_x = (event.x-x)+root.winfo_x()
    new_y = (event.y-y)+root.winfo_y()
    x_grab,y_grab = new_x,new_y
    s = "{}x{}+{}+{}".format(width_grab, height_grab, new_x, new_y)
    # s = "{}x{}+" + str(new_x)+"+" + str(new_y)
    root.geometry(s)
    # canvas.create_rectangle(new_x, new_y, 300, 200, fill="blue")
    print("s = ",s)
    print(root.winfo_x(),root.winfo_y())
    print(event.x,event.y)
    print()

def button_1(event):
    global x,y
    x,y = event.x,event.y
    print("x, y = ", x, y)
    print("event.x, event.y = ",event.x,event.y)

def start_record_gif():
    global output_gif_name, x_grab, y_grab, flag_stop_record
    images_temp_list = []
    # start_time = time.time()
    isEnded = False
    while(not isEnded):
        if(flag_stop_record[0] == True):
            isEnded = True
        # try:
            # flag_stop_record.get(False)
            # isEnded = True
        # except:
            # isEnded = False
        im = ImageGrab.grab((x_grab, y_grab , x_grab+width_grab, y_grab+height_grab))
        im.save('temp.png')
        images_temp_list.append(imageio.imread('temp.png'))
        time.sleep(0.001)
        imageio.mimsave(output_gif_name, images_temp_list, duration=0.3)
        # end_time = time.time()
        # if(end_time - start_time > 5):
            # break

def record_gif(event):
    global flag_is_recording
    if(not flag_is_recording[0]):
        root.geometry("0x0+0+0")
        flag_is_recording[0] = True
        t = Thread(target=start_record_gif, args=())
        t.start()
    else:
        pass


def exit(event):
    # flag_stop_record.put(True)
    flag_stop_record[0] = True
    root.destroy()

canvas.bind("<B1-Motion>", move)
canvas.bind("<Button-1>", button_1)
canvas.bind("<Button-3>", exit)
canvas.bind("<Double-Button-1>",record_gif)

root.mainloop()
```

## Next

- 优化使用方式
- 使用temp目录
- ...

更多信息可以参考：

[python小应用之moviepy的视频剪辑制作gif图](https://blog.csdn.net/Spade_/article/details/79516322)

[利用Python来完成屏幕录制](https://blog.csdn.net/zzzzjh/article/details/80903597)
