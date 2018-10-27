---
layout: post
title: Python Screen Gif (03)
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

# Background

今天更新了下代码，提高了可用性，增加了UI设定截屏的区域，有时间再更新这篇，已经基本可用了。

# Code

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

# Next

- 优化使用方式
- 使用temp目录
- ...

更多信息可以参考：

[Python Screen Gif (01)](https://bearfly1990.github.io/2018/10/22/PyScreenGif/)

[Python Screen Gif (02)](https://bearfly1990.github.io/2018/10/24/PyScreenGif02/)
