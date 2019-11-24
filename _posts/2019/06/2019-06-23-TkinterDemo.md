---
layout: post
title: Tinkter Demo
subtitle: python tinkter demo
date: 2019-06-25
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - python
  - tkinter
---

# 背景

最近在做数据的交互与导入导出，刚开始还是按照原先的方式，把所有的参数写在配置文件中。

在运行前需要把配置改掉，这样很容易遗漏。

所以想到了写点简单的界面来配置一些选项，相对来说所见即所得。

Python 里可以写界面的选择很多，也有很强大和专业的库(e.g. PyQt)，但是我们这种程度的使用，用原生的 tinkter 就足够了。

# tinkter 基本信息

现在基本上都是基于 python 3.0 在开发，所以 tkinter 的组件命名也发生的一些变化。

```python
# Python 2.x使用这行

#from Tkinter import *

# Python 3.x使用这行

from tkinter import *
```

## ttk

程序都是直接使用 tkinter 模块下的 GUI 组件的，这些组件看上去特别“复古”，也就是丑，仿佛是从 20 年前的程序上抠出来的组件。

为了弥补这点不足，Tkinter 后来引入了一个 ttk 组件作为补充（主要就是简单包装、美化一下），并使用功能更强大的 Combobox 取代了原来的 Listbox，且新增了 LabeledScale（带标签的 Scale）、Notebook（多文档窗口）、Progressbar（进度条）、Treeview（树）等组件。

ttk 作为一个模块被放在 tkinter 包下，使用 ttk 组件与使用普通的 Tkinter 组件并没有多大的区别，只要导入 ttk 模块即可

## 布局

UI 组件需要我们摆放位置，tinkter 提供了三种布局方式：

- Pack
- Grid
- Place

相对来说 Pack 和 Grid 相对多一些，Pack 更加灵活，而 Grid 适方方正正的类表格布局，而在同一个 Frame 中，布局方式只能有一种。

# Demo

在我下面列的参考文档中有很详细的组件与细节介绍，下面我直接放一个自己写的简单Demo。
![2019-06-23-TkinterDemo.png](/img/post/2019/06/2019-06-23-TkinterDemo.png)
```python
import webbrowser
from PIL import ImageTk
from tkinter import ttk
from tkinter import messagebox
from tkinter import *

def gamerun():
    print('run game...')

def init_menu(root, imgGame, imgHelp):
    menubar =Menu(root)
    root.config(menu = menubar)

    #实例化菜单1，创建下拉菜单，调用add_separate创建分割线
    menu1 =Menu(menubar,tearoff = 0)
    menubar.add_cascade(label = "Edit",menu = menu1)
    menu1.add_command(label = "Do Nothing")
    menu1.add_separator()
    menu1.add_command(label = "Quit",command = root.quit)

    menu2 =Menu(menubar,tearoff = 0)
    menubar.add_cascade(label = "More",menu = menu2)
    menu2.add_command(label = "New Job",image = imgGame ,compound= "left",command = lambda:gamerun())

    
    menu2.add_command(label = "Tkinter",image = imgHelp,compound = "left",command =
    lambda:webbrowser.open("http://effbot.org/tkinterbook/tkinter-index.htm"))


def init_userinfo(root):
    frame_user_info = ttk.LabelFrame(root, text='UserInfo:')
    frame_user_info.pack(side=TOP, fill=X)

    # myLabel= Label(frame_user_info, text='UserInfo:', font="Helvetica 10 bold")
    # myLabel["relief"]=tk.SOLID#设置label的样式
    # myLabel["width"]=10
    # myLabel["height"]=5
    # myLabel.pack(side=LEFT)
    # myLabel.grid(row=0, sticky=W)
    Label(frame_user_info, text="Username").grid(row=1, sticky=W)
    Label(frame_user_info, text="Password").grid(row=2, sticky=W)
    username = Entry(frame_user_info).grid(row=1, column=1, sticky=E)
    password = Entry(frame_user_info, show='*').grid(row=2, column=1, sticky=E)
    # Button(frame_user_info, text="Login").grid(row=2, column=1, sticky=E)

def init_radio(root):
    frame_radio = ttk.Labelframe(root, text='Radio Test',padding=20)
    frame_radio.pack(fill=BOTH, expand=YES, padx=10, pady=10)
    books = ['C++', 'Python', 'Linux', 'Java']
    i = 0
    books_radio = []
    for book in books:
        intVar = IntVar()
        books_radio.append(intVar)
        Radiobutton(frame_radio, text=book,value=i,variable=intVar,command=changed).pack(side=LEFT)
        i = i + 1 


def changed():
    print('value changed')

def init_checkbutton(root):
    frame_checkbutton = ttk.Labelframe(root, text='Checkbutton Test',padding=20)
    frame_checkbutton.pack(fill=BOTH, expand=YES, padx=10, pady=10)
    books = ['C++', 'Python', 'Linux', 'Java']
    i = 0
    books_checkbox = []
    for book in books:
        strVar = StringVar()
        books_checkbox.append(strVar)
        cb = ttk.Checkbutton(frame_checkbutton,
            text = book,
            variable = strVar, 
            onvalue = i,
            offvalue = 'None',
            command = changed) 
        cb.pack(anchor=W)
        i += 1

def init_combobox(root):
    frame_combobox = ttk.Labelframe(root, text='Combobox Test',padding=20)
    frame_combobox.pack(fill=BOTH, expand=YES, padx=10, pady=10)
    strVar = StringVar()
    # 创建Combobox组件
    cb = ttk.Combobox(frame_combobox,
        textvariable=strVar, # 绑定到self.strVar变量
        postcommand=changed) # 当用户单击下拉箭头时触发self.choose方法
    cb.pack(side=TOP)
    # 为Combobox配置多个选项
    cb['values'] = ['Python', 'Ruby', 'Kotlin', 'Swift']

def show_it():
    messagebox.showinfo(title='Alert', message="Please try again!")

def init_showinfo(root):
    frame_showinfo = ttk.LabelFrame(root, text='ShowInfoTest:')
    frame_showinfo.pack(side=TOP, fill=X)
    Button(frame_showinfo, text="Click Me", command=show_it).pack(side=LEFT, fill=Y)
    

def init_ui():
    root = Tk()
    # root.geometry('580x680+200+100')
    root.resizable(width = False, height = False) 
    root.title("Test")
    root.iconbitmap('login.ico')
    imgGame = PhotoImage(file='game.png')
    imgHelp = ImageTk.PhotoImage(file="help.png")
    init_menu(root, imgGame, imgHelp)
    init_userinfo(root)
    init_showinfo(root)
    init_radio(root)
    init_checkbutton(root)
    init_combobox(root)
    root.mainloop()

if __name__ == '__main__':
    init_ui()
```

参考:

- [Python Tkinter 教程（GUI 图形界面开发教程）](http://c.biancheng.net/python/tkinter/)
- [Python2.7 Tkinter 创建简单登录注册界面](https://www.jianshu.com/p/f0a3c47c9c6c)
