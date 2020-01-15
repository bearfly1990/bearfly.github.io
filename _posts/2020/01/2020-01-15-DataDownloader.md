---
layout: post
title: Data Downloader
subtitle: Download data from DB
date: 2020-01-15
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - python
  - tkinter
  - database
  - download
---

# 背景

有时候需要从数据库中把数据导出来查看,可以在编辑器中直接拷出来或者导出来。

又或者像 weekly/monthly 的数据，我偶尔导一次，不想再打开 sqldeveloper 去操作 Oracle，所以就写了类似下面的工具。

# 功能

使用 tkniter 来编写了简单的界面：

![2020-01-15-DataDownloader-01.jpg](/img/post/2020/01/2019-01-15-DataDownloader-01.jpg)

## 数据库配置 DB.ini

为了省去输入数据库配置的时间，定义了一个配置文件，在刚开始加载的时候就读取进去，支持 Oracle 和 SqlServer

```ini
[Oracle]
host=host
port=25
service=instance
[SqlServer]
service=PC-CX\SQLEXPRESS
```

```python
def read_config(self):
    config = configparser.RawConfigParser()
    config.read('./DB.ini')
    self.oracle_info = OracleInfo(host=config['Oracle']['host'], port=config['Oracle']['port'],
                                  service=config['Oracle']['service'])
    self.sql_server_info = SqlServerInfo(service=config['SqlServer']['service'])
```

## UI
### Username & Password

按公司规定，用户名和密码是不能写在文件中的，所以每次都需要手动输入，于是把这一块放到 BaseUI 中。

```python
def init_user_pwd_entry(self, row=0, user_val=''):
    self.init_frame()
    self.entry_user = self.init_input_field(text="Username:", val=user_val, row=row, column=0)
    self.entry_pwd = self.init_input_field(text="Password:", row=row + 1, column=0, with_star=True)
```

### Sql TextBox

把需要执行的query语句写在这里：
```python
def init_sql_textbox(self):
    self.init_frame()
    tk.Label(self.frame, text='sql:').grid(row=0, column=0, sticky=tk.W)
    self.entry_sql = tk.Text(self.frame, height=4, width=50)
    self.entry_sql.grid(row=1, column=0, stick=tk.W)
```

### Output path
默认输出到文件Output.csv中，当然可以换成别的文件：
```python
def init_output_path(self):
    self.init_frame()
    tk.Label(self.frame, text='output path:').grid(row=0, column=0, sticky=tk.W)
    self.label_output_file = tk.Label(self.frame, text='./output.csv')
    self.label_output_file.grid(row=0, column=1, sticky=tk.W)
    tk.Button(self.frame, text='...', command=self.select_file).grid(row=0, column=2, sticky=tk.W, pady=4)
```

### DB Type
可以切换数据库的类型，目前支持SqlServer和Oracle
```python
def init_db_info_radios(self):
    self.init_frame()
    tk.Radiobutton(self.frame, text="SqlServer", command=self.change_db_type,
                    variable=self.db_category_var, value=1).grid(row=0, column=0, stick=tk.W)
    tk.Radiobutton(self.frame, text="Oracle", command=self.change_db_type,
                    variable=self.db_category_var, value=2).grid(row=0, column=1, stick=tk.W)

def init_db_info_details(self):
    self.frame_sql_server = self.create_new_frame()
    self.entry_service_sql_server = self.init_input_field(frame=self.frame_sql_server, text='Service:', row=0,
                                                          column=0)
    self.entry_service_sql_server.insert(tk.END, self.sql_server_info.service)

    self.frame_oracle = self.create_new_frame()
    self.entry_host_oracle = self.init_input_field(frame=self.frame_oracle, text='Host:', row=0, column=0)
    self.entry_host_oracle.insert(tk.END, self.oracle_info.host)

    self.entry_port_oracle = self.init_input_field(frame=self.frame_oracle, text='Port:', row=1, column=0)
    self.entry_port_oracle.insert(tk.END, self.oracle_info.port)

    self.entry_service_oracle = self.init_input_field(frame=self.frame_oracle, text='Service:', row=2, column=0)
    self.entry_service_oracle.insert(tk.END, self.oracle_info.service)

    self.frame_oracle.pack_forget()
```
![2020-01-15-DataDownloader-02.jpg](/img/post/2020/01/2019-01-15-DataDownloader-02.jpg)

### Control Buttons

最后的开始与退出按钮也作为BaseUI的一部分。
```python
def init_control_buttons(self, row=0):
    self.frame_control = self.create_new_frame()
    tk.Button(self.frame_control, text='Start', command=self.do_ok).grid(row=row, column=0, sticky=tk.W, pady=4)
    tk.Button(self.frame_control, text='Quit', command=self.frame.quit).grid(row=row, column=1, sticky=tk.W, pady=4)
```

## 关键处理过程

### 绑定执行方法

最后的执行方法如下：
```python
class DownloadAction(object):
    @classmethod
    def start_download(cls, db_info, sql_str, output_file_name):
        start_time = datetime.now()
        with db_info.connect() as conn:
            print(f'start query {sql_str}')
            sql_query = pd.read_sql_query(sql_str, conn)
            df = pd.DataFrame(sql_query)
            df.to_csv(output_file_name, index=False)
            end_time = datetime.now()
            print(f'save data to {output_file_name} finished. cost={end_time - start_time}s')
```

通过初始化UI时，绑定对应的方法：

```python
def __init__(self, action):
        self.action = action
```

把方法当成参数传递：
```python
if __name__ == '__main__':
    DownloadUI(DownloadAction().start_download)
```
在点击Start的时候，执行do_ok方法：
```python
tk.Button(self.frame_control, text='Start', command=self.do_ok).grid(row=row, column=0, sticky=tk.W, pady=4)
```
### do_ok()
添加了对用户名和密码的简单判空。
```python
def do_ok(self):
    sql_str = self.entry_sql.get("1.0", "end-1c")

    if not (self.entry_user.get().strip() and self.entry_pwd.get().strip()):
        tkinter.messagebox.showerror(title='Error', message='Please input username and password!')
        return

    if not sql_str.strip():
        tkinter.messagebox.showerror(title='Error', message='Please input sql!')
        return

    self.root.withdraw()
    db_info = self.sql_server_info
    if self.db_category_var.get() == 2:
        db_info = self.oracle_info

    db_info.user = self.entry_user.get()
    db_info.pwd = self.entry_pwd.get()

    self.action(db_info, sql_str, self.label_output_file['text'])
    self.root.deiconify()
```

# 最后

完整代码在[tkinter/demo_02](https://github.com/bearfly1990/PowerScript/tree/master/Python3/tkinter/demo_02)

