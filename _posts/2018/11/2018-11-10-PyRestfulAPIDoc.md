---
layout: post
title: RESTful API Doc
subtitle: RESTful API Doc javascript lib apidocjs 
date: 2018-11-10
author: BF
header-img: img/bf/river_01.jpg
catalog: true
tags:
  - apidoc
  - python
  - RESTful
---

# RESTful API Doc

针对`RESTful API`,有许多工具可以用来编写文档比如`Swagger2`，之前发现一个挺好用的库就是[apidocjs](http://apidocjs.com/)

这个库支持大多数流行程序语言，把接口相关的信息放在注释中，而这个js库解析注释中的有效信息生成html文档，需要安装`node.js`。

# Sample

下面直接根据昨天的API例子来添加注释，生成文档。

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

class Todo(Resource):
    """
    @api {get} /todo/task/<task_id> get todo by id
    @apiVersion 0.1.0
    @apiName GetTodoItem
    @apiGroup Todo
    @apiPermission all
    @apiParam {Number} task_id The todo id.
    @apiExample Example usage:
        https://localhost:5000/v1/todo/task/todo1
    @apiSuccessExample {json} Success-Response:
        HTTP/1.1 200 OK
        {'task': 'Build an API'}
    """
    """
    @api {get} /todo/<todo_id> get todo by id
    @apiVersion 0.2.0
    @apiName GetTodoItem
    @apiGroup Todo
    @apiPermission all
    @apiParam {Number} todo_id The todo id.
    @apiExample Example usage:
        https://localhost:5000/v1/todo/todo1
    @apiSuccessExample {json} Success-Response:
        HTTP/1.1 200 OK
        {'task': 'Build an API'}
    """
    def get(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        return TODOS[todo_id]
    """
    @api {delete} /todo/<todo_id> delete todo task by id
    @apiVersion 0.1.0
    @apiName DeleteTodo
    @apiGroup Todo
    @apiPermission admin
    @apiParam {Number} todo_id The todo id.
    @apiExample Example usage:
        https://localhost:5000/v1/todo/todo1
    @apiSuccessExample {json} Success-Response:
        HTTP/1.1 204 No Content
    @apiErrorExample Response (example):
        HTTP/1.1 401 Not Authenticated
        {"error": "NoAccessRight"}
    """
    def delete(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        del TODOS[todo_id]
        return '', 204
    """
    @api {put} /todo/<todo_id> put a new task
    @apiVersion 0.1.0
    @apiName PutTask
    @apiGroup Todo
    @apiPermission admin
    @apiParam {Number} todo_id The todo id.
    @apiExample Example usage:
        https://localhost:5000/v1/todo/todo1
        {"task": "New Task"}
    @apiSuccessExample {json} Success-Response:
        HTTP/1.1 201 Created 
        {"task": "New Task"}
    """
    def put(self, todo_id):
        args = parser.parse_args()
        task = {'task': args['task']}
        return task, 201

class TodoList(Resource):
    """
    @api {get} /todos get todolist
    @apiVersion 0.1.0
    @apiName GetTodoList
    @apiGroup TodoList
    @apiPermission all
    @apiExample Example usage:
        https://localhost:5000/v1/todos
    """
    def get(self):
        return TODOS
    """
    @api {post} /todos post todo list
    @apiVersion 0.1.0
    @apiName PostTodoList
    @apiGroup TodoList
    @apiPermission admin
    @apiExample Example usage:
        https://localhost:5000/v1/todo/todo1
        {"task":"New Task"}
    @apiSuccessExample {json} Success-Response:
        HTTP/1.1 201 Created
        {'task': 'New Task'}
    """
    def post(self):
        args = parser.parse_args()
        todo_id = int(max(TODOS.keys()).lstrip('todo')) + 1
        todo_id = 'todo%i' % todo_id
        TODOS[todo_id] = {'task': args['task']}
        return TODOS[todo_id], 201

api.add_resource(TodoList, '/todos')
api.add_resource(Todo, '/todos/<todo_id>')

if __name__ == "__main__":
    app.run(debug=True)
```

运行下面的命令，就可以在`pytrestapi_doc`目录下生成api文档

```powershell
PS C:\Users\mayn\Desktop\workspace> apidoc -i .\pyrestapi\ -o .\pytrestapi_doc
info: Done.
```

下面是报告的结果,可以感受下：
[https://bearfly1990.github.io/apidoc/pyrestapi-doc/](https://bearfly1990.github.io/apidoc/pyrestapi-doc/)

下面是简单的环境配置步骤：

- 安装`node.js`
- 安装 apidoc `npm install apidoc -g`
- 装备好已经编写好对应注释的代码项目
- `apidoc -i myapp/ -o apidoc/`  就能生成在apidoc文件夹中

更多信息请参考官网：

[apidocjs](http://apidocjs.com/)
