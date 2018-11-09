---
layout: post
title: Python RESTful API
subtitle: Python RESTful API by flask-restful
date: 2018-11-09
author: BF
header-img: img/bf/mountain_01.jpg
catalog: true
tags:
  - python
  - RESTful
  - flask-restful
---

# RESTful API

EST全称是Representational State Transfer，中文意思是表述（编者注：通常译为表征）性状态转移。 它首次出现在2000年Roy Fielding的博士论文中，Roy Fielding是HTTP规范的主要编写者之一。 他在论文中提到："我这篇文章的写作目的，就是想在符合架构原理的前提下，理解和评估以网络为基础的应用软件的架构设计，得到一个功能强、性能好、适宜通信的架构。REST指的是一组架构约束条件和原则。" 如果一个架构符合REST的约束条件和原则，我们就称它为RESTful架构。

REST本身并没有创造新的技术、组件或服务，而隐藏在RESTful背后的理念就是使用Web的现有特征和能力， 更好地使用现有Web标准中的一些准则和约束。虽然REST本身受Web技术的影响很深， 但是理论上REST架构风格并不是绑定在HTTP上，只不过目前HTTP是唯一与REST相关的实例。 所以我们这里描述的REST也是通过HTTP实现的REST。

# Python flask-restful

`flask`是python的一个轻量级网站开发框架，直接利用他就能开发简单的`web api`。

但是使用`flask-restful`的话，就更加的简单和适合。

# flask

下面是直接使用flask的例子:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/helloworld')
def hello_world():
    return "Hello World!"

if __name__ == "__main__":
    app.run(debug=True)
```

上面是一个非常简单的使用flask开发api的例子，在运行后直接就能访问使用

```powershell
PS C:\Users\mayn\Desktop> python ./hello.py
 * Serving Flask app "hello" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 215-586-846
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

```
![sample01](https://raw.githubusercontent.com/bearfly1990/bearfly1990.github.io/master/_posts/2018/11/imgs/2018-11-09-PyRestfulAPI-demo01.jpg)

# flask-restful

如果使用`flask-restful`的话，那就会结构更清晰一些：

```python
from flask import Flask
from flask_restful import reqparse, abort, Api, Resource

app = Flask(__name__)

api = Api(app)

TODOS = {
    'todo1': {'task': 'Build an API'},
    'todo2': {'task': 'Test it'},
    'todo3': {'task': 'Write Blog'},
}

def abort_if_todo_doesnt_exist(todo_id):
        if todo_id not in TODOS:
            abort(404, message="Todo {} doesn't exist.".format(todo_id))

parser = reqparse.RequestParser()
parser.add_argument('task')

# 3个方法，对应get/delete/put
class Todo(Resource):
    def get(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        return TODOS[todo_id]

    def delete(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        del TODOS[todo_id]
        return '', 204

    def put(self, todo_id):
        args = parser.parse_args()
        task = {'task': args['task']}
        return task, 201

# 2个方法，对应get/post
class TodoList(Resource):
    def get(self):
        return TODOS

    def post(self):
        args = parser.parse_args()
        todo_id = int(max(TODOS.keys()).lstrip('todo')) + 1
        todo_id = 'todo%i' % todo_id
        TODOS[todo_id] = {'task': args['task']}
        return TODOS[todo_id], 201

# 绑定api对应的类
api.add_resource(TodoList, '/todos')
api.add_resource(Todo, '/todos/<todo_id>')

if __name__ == "__main__":
    app.run(debug=True)

```

# 总结

总体来说，使用`flask-restful`框架真的方便很多, 一些处理已经封闭好。比如`json`就是默认的输入输出，状态状态码也直接放在return中和返回的内容一起就好。

更多信息请参考：

[todo.py](https://github.com/bearfly1990/PowerScript/blob/master/Python3/flask-restful/todo.py)

[RESTful架构详解](http://www.runoob.com/w3cnote/restful-architecture.html)

[python实现RESTful服务（基于flask](https://www.jianshu.com/p/6ac1cab17929)