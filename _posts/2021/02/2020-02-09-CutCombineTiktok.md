---
layout: post
title: Cut and Combine Tiktok Videos
subtitle: Cut and Combine downloaded Tiktok videos with moviepy
date: 2021-02-09
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - python
  - moviepy
  - tiktok
---

#  更新

| date       | update                                                       |
| ---------- | ------------------------------------------------------------ |
| 2021-02-17 | 1. 修复了不同视频大小的压缩算法 2. 修改代码结构并支持并发预处理文件 |

# 背景

最近看到tiktok上有许多有意思的视频，所以想下载下来。但下过来的视频会有水印，主要是视频后3秒会有抖音的视频水印，很影响观感。

对于文字水印，需要用别的方式来下载，不太方便，后面可以再想办法。

现在的需求是：我在手机上下了n个视频，copy到电脑上之后，希望能去掉片尾的视频水印，并合并成一个大的视频。

# 功能

将当前目录下，所以的mp4文件的后3秒（视频水印）去掉，然后合成一个大的视频文件。

## 主要代码

在网上找了许多资料，最后决定使用moviepy。

```python
import imageio
import win_unicode_console
win_unicode_console.enable()
import os
from moviepy.video.io.VideoFileClip  import VideoFileClip
from moviepy.video.compositing.concatenate import concatenate_videoclips
from moviepy.editor import VideoFileClip, clips_array, vfx, CompositeVideoClip
import glob

if __name__=="__main__":
    output_folder = './output'
    files = glob.glob('**/*.mp4', recursive=True)
    video_list = []
    start_sec = 0
    for file in files:
        try:
            source = file
            target = os.path.join(output_folder, source) #拼接文件名路径
            
            if not os.path.exists(os.path.dirname(target)):
                os.makedirs(os.path.dirname(target))
            video = VideoFileClip(source)
            total_seconds = video.duration
            start_time = 0
            stop_time = total_seconds - 3
            video = video.subclip(int(start_time), int(stop_time))#执行剪切操作
            
            video.to_videofile(target, fps=20, remove_temp=True)#输出文件
            
            # os.remove(source)
            
            print(video.size[0],video.size[1])
            print(video.size[0]/1300,video.size[1]/720)
            rate_x = video.size[0]/1300
            rate_y = video.size[1]/720
            rate_max = max(rate_x, rate_y)
            if rate_max > 1:
                rate_max = 1/rate_max
            else:
                rate_max = 1
            video = video.set_start(start_sec).set_pos("center").resize(rate_max)
            print('-=====>', rate_max)
            start_sec = start_sec + video.duration
            video_list.append(video)#将加载完后的视频加入列表
            
        except Exception as e:
            print('have error:',e)
        finally:
            print(file, 'done')
    final_clip = CompositeVideoClip(video_list, size=(1300, 720))
    final_clip.to_videofile(os.path.join(output_folder, 'combined.mp4'), fps=20, remove_temp=True)
    # final_clip = concatenate_videoclips(video_list)#进行视频合并
    
    # final_clip.write_videofile(os.path.join(output_folder, 'combined.mp4'), fps=20, remove_temp=True)
    
    # final_clip.to_videofile(os.path.join(output_folder, 'combined.mp4'), fps=20, remove_temp=True)#将合并后的视频输出
```

## 合并成一个视频

这边对于每个新的视频，都重新设置位置和size，主要是为了支持后面合并不同分辨率做准备。

```python
video = video.set_start(start_sec).set_pos("center").resize(video.size[0]/1300)
start_sec = start_sec + video.duration
video_list.append(video)#将加载完后的视频加入列表
```

这边使用`CompositeVideoClip`来合并视频，而上面的`start_sec`，便是每个视频在合并的视频中，开始播放的时间。如果不设置，就是一所有视频都在`0`s开始播放，大家可以想到，如果这个时间配置视频的位置，就可以达到同时放多个小视频的效果，而这里使用它，纯粹是为了支持合并多个不同的分辨率
```python
final_clip = CompositeVideoClip(video_list, size=(1300, 720))
final_clip.to_videofile(os.path.join(output_folder, 'combined.mp4'), fps=20, remove_temp=True)
```
# 2021-02-17更新

## 修复了不同视频大小的压缩算法 

之前在合并不同大小的视频时，对于合并的算法有问题，不能适应所有的情况。

更新后如下，根据长和宽，适合最合适的缩放大小。
```python
rate_x = video.size[0]/1300
rate_y = video.size[1]/720
rate_max = max(rate_x, rate_y)
if rate_max > 1:
    rate_max = 1/rate_max
else:
    rate_max = 1
VIDEO_LIST[i] = video.set_start(start_sec).set_pos("center").resize(rate_max)
```
## 修改代码结构并支持并发预处理文件

使用线程池来实现并发操作，充分使用电脑性能来做数据预处理。
```python
executor = ThreadPoolExecutor(max_workers=MAX_WORKERS)
files = glob.glob('**/*.mp4', recursive=True)
all_task = [executor.submit(convert_video, (file)) for file in files]
wait(all_task, return_when=ALL_COMPLETED)
```
## 完整新代码如下：
```python
import imageio
import win_unicode_console
win_unicode_console.enable()
import os
from moviepy.video.io.VideoFileClip  import VideoFileClip
from moviepy.video.compositing.concatenate import concatenate_videoclips
from moviepy.editor import VideoFileClip, clips_array, vfx, CompositeVideoClip
import glob
from concurrent.futures import ThreadPoolExecutor, wait, ALL_COMPLETED, FIRST_COMPLETED
from datetime import datetime

OUTPUT_FOLDER = './output'
VIDEO_LIST = []
MAX_WORKERS = 6

def convert_video(file):
    try:
        target = os.path.join(OUTPUT_FOLDER, file) # 拼接文件名路径
        
        try:
            if not os.path.exists(os.path.dirname(target)):
                os.makedirs(os.path.dirname(target))
        except Exception as e:
            print('have error when create subfolder:',e)
        video = VideoFileClip(file)
        total_seconds = video.duration
        start_time = 0
        stop_time = total_seconds - 3
        video = video.subclip(int(start_time), int(stop_time))# 执行剪切操作
        
        video.to_videofile(target, fps=20, remove_temp=True)# 输出文件
        
        # os.remove(source)
        
        VIDEO_LIST.append(video)# 将加载完后的视频加入列表
        
    except Exception as e:
        print('have error:',e)
    finally:
        print(file, 'done')

def combine_videos():
    start_sec = 0
    for i, video in enumerate(VIDEO_LIST):
        # print(video.size[0],video.size[1])
        
        # print(video.size[0]/1300,video.size[1]/720)
        
        rate_x = video.size[0]/1300
        rate_y = video.size[1]/720
        rate_max = max(rate_x, rate_y)
        if rate_max > 1:
            rate_max = 1/rate_max
        else:
            rate_max = 1
        VIDEO_LIST[i] = video.set_start(start_sec).set_pos("center").resize(rate_max)
        start_sec = start_sec + video.duration
        
    final_clip = CompositeVideoClip(VIDEO_LIST, size=(1300, 720))   
    final_clip.to_videofile(os.path.join(OUTPUT_FOLDER, 'combined.mp4'), fps=20, remove_temp=True)


if __name__=="__main__":
    start_time = datetime.now()
    executor = ThreadPoolExecutor(max_workers=MAX_WORKERS)
    files = glob.glob('**/*.mp4', recursive=True)
    all_task = [executor.submit(convert_video, (file)) for file in files]
    wait(all_task, return_when=ALL_COMPLETED)
    combine_videos()
    ended_time = datetime.now()
    print(f'time cost: {ended_time - start_time}')
```
# 最后

完整代码在[moviepy](https://github.com/bearfly1990/PowerScript/tree/master/Python3/moviepy/)

可以关注我的微信视频号，看简单的演示。

![my wechat video QR](/img/bf_wechat_video.jpg)

参考：

* [使用Python+moviepy连接不同尺寸的视频文件](https://cloud.tencent.com/developer/article/1582917)
* [MoviePy不同尺寸视频vedio_clip或者图片image_clip拼接出现花屏](https://blog.csdn.net/ucsheep/article/details/84630800)
* [MoviePy问题解决汇总](https://blog.csdn.net/ucsheep/article/details/84387092)
* [MoviePy - 中文文档2-快速上手-MoviePy-视频合成](https://blog.csdn.net/ucsheep/article/details/81329598)
* [python线程池 ThreadPoolExecutor 使用详解](https://blog.csdn.net/xiaoyu_wu/article/details/102820384)
