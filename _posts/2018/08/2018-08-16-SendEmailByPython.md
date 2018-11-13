---
layout:     post
title:      Send email by python
subtitle:   Class to send 163 email by python
date:       2018-08-16
author:     BF
header-img: img/bf/beach_01.jpg
catalog: true
tags:
    - python
    - email
---
# Send email by python

`python`有两个原生库(`smtplib`,`email`)可以用来发送邮件，只要你有`smtp` server就行。之前在公司已经写过简单的类来实现，回来又重新写了一个使用`163`邮箱来发送的工具类。

目前已经能够完成基本发送功能，包括`html`格式，添加附件等，只是弄了好久，还是没有办法在正文中添加图片，一直`554`错误，被网易认为垃圾信息([错误代码一览](http://help.163.com/09/1224/17/5RAJ4LMH00753VB8.html))而发送不了，之后有时间再看。

## 授权码

想利用163的邮箱发送邮件，就需要先开通授权码：
![permission](/img/post/2018-08-16-SendEmailByPython-permission.jpg)

## 主要代码

```python
#/usr/bin/python3
"""
author: xiche
create at: 08/15/2018
description:
    Utils for send email
Change log:
Date         Author      Version    Description
08/15/2018    xiche      1.0.1      Setup
"""
import smtplib
import os
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.mime.image import MIMEImage
from email.header import Header
class EmailUtils:
    __sender = "bearfly1990@163.com"
    __recipients = ["xiongchen1990@163.com", "xchen1230@163.com"]
    __subject = 'python email 尝试'
    __content = '<p>This is the test by <b>python</b>, please have a try.</p>' #<p>截图如下：</p><p><img src="cid:image1"></p>
    __mail_permission = 'xxxxxx'#授权码，非邮箱密码
    __mail_host = 'smtp.163.com'
    __mail_port = 25 #SSL 465
    __attachments_path = [r'C:\Users\mayn\Desktop\test.txt']
    __content_images_path = [{'cid':'image1', 'path': r'C:\Users\mayn\Desktop\ymj.jpg'}]

    def __init__(self):
        pass

    def send_email(self):
        msg = MIMEMultipart()

        # msg = MIMEText(self.content, 'plain', 'utf-8')
        msg['Subject'] =  Header(self.subject, 'utf-8')
        msg['From'] = self.sender
        msg['To'] = ','.join(self.recipients)

        content = MIMEText(self.__content,'html','utf-8')
        msg.attach(content)

        for content_image in self.__content_images_path:
            msgImage = MIMEImage(open(content_image['path'], 'rb').read())
            msgImage.add_header('Content-ID', '<{}>'.format(content_image['cid']))
            msg.attach(msgImage)

        for attachments_path in self.__attachments_path:
            attachment = MIMEText(open(attachments_path, 'rb').read(), 'base64', 'utf-8')
            attachment["Content-Type"] = 'application/octet-stream'
            attachment["Content-Disposition"] = 'attachment; filename="{}"'.format(os.path.basename(attachments_path))
            msg.attach(attachment)


        # smtp = smtplib.SMTP(self.__mail_host, port=self.__mail_port)
        try:
            smtp = smtplib.SMTP_SSL()  # 注意：如果遇到发送失败的情况（提示远程主机拒接连接），这里要使用SMTP_SSL方法
            smtp.connect(self.__mail_host)
            smtp.login(self.sender, self.__mail_permission)
            # print(msg.as_string())
            smtp.sendmail(self.sender, self.recipients, msg.as_string())
            print('email send successfully.')
        except smtplib.SMTPException as e:
            print(str(e))
        finally:
            smtp.quit()  # 发送完毕后退出smtp

    '''sender'''
    @property
    def sender(self):
        return self.__sender

    @sender.setter
    def sender(self, sender):
        self.__sender = sender
    '''subject'''
    @property
    def subject(self):
        return self.__subject

    @subject.setter
    def subject(self, subject):
        self.__subject = subject
    '''recipients'''
    @property
    def recipients(self):
        return self.__recipients

    @recipients.setter
    def recipients(self, recipients):
        if(isinstance(recipients, str)):
            self.__recipients = recipients.split(';')
        elif(isinstance(recipients, list)):
            self.__recipients = recipients
    '''sender'''
    @property
    def content(self):
        return self.__content

    @content.setter
    def content(self, content):
        self.__content = content

if __name__ == '__main__':
    emailUtils = EmailUtils()
    emailUtils.send_email()
    # print(os.path.basename(r'C:\Users\mayn\Desktop\ymj.jpg'))
```

更多信息请参考：[https://github.com/bearfly1990/PowerScript/blob/master/Python3/mylib/cmutils_email.py](https://github.com/bearfly1990/PowerScript/blob/master/Python3/mylib/cmutils_email.py)
