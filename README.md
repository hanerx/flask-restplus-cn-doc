# Flask-RESTPlus 中文文档

本文档是Flask-RESTPlus的文档中文翻译版，英文原版地址：https://flask-restplus.readthedocs.io/en/stable/

# 相关信息

- 文档对应版本：0.13.0 stable
- 本文档是个人翻译版本，非官方翻译版，一切内容以官方英文原版为准。
- 本文档与Flask-RESTPlus开发组无任何关系，提交bug等开发问题请移步项目所在地：https://github.com/noirbizarre/flask-restplus
- 部分翻译内容可能并不准确，没把握的地方我都标记了原文
- 文档采用[Typora](http://typora.io/)编写，推荐使用Typora进行查看

# 目录

- [安装](#安装)
- [快速开始](#快速开始) 
  - [初始化](#初始化) 
- [一个最小API](#一个最小API)  
  - [面向资源（Resource）的路由](#面向资源（Resource）的路由) 
  - [端点（Endpoint）](#端点（Endpoint）) 
  - [参数解析](#参数解析) 
  - [数据格式化](#数据格式化) 
- [响应编组](#响应编组) 
  - 基础用法
  - 重命名属性
  - 默认值
  - 自定义字段（Custom Fields）& 多个数值
  - Url & 其他预定义字段（Other Concrete Fields）
  - 复杂结构
  - 列表字段
  - 通配符字段
  - 嵌套字段
  - `api.model()` 函数
  - 自定义字段
  - 跳过null字段（Skip fields which value is None）
  - 使用Json格式定义模型
- 请求解析
  - 基础参数
  - 必要参数
  - 多个数值 & 列表
  - 其他源（Other Destinations）
  - 参数位置
  - 多位置参数
  - 高级类型处理
  - 解析器继承
  - 文件上传
  - 错误处理
  - 错误消息
- [异常处理](#异常处理)
  - [Http异常处理](#Http异常处理)
  - [Flask终止助手](#Flask终止助手) 
  - [Flask-RESTPlus终止助手](#Flask-RESTPlus终止助手) 
  - [ `@api.errorhandler` 装饰器](#apierrorhandler-装饰器) 
- [字段掩码（Fields masks）](#字段掩码Fields-masks)
  - [语法](#语法)
  - [使用方法](#使用方法)
- Swagger文档
  - 使用`@api.doc()`装饰器进行文档编辑
  - 自动记录模型
  - `@api.marshal_with()`装饰器
  - `@api.expect()`装饰器
  - 使用`@api.response`装饰器进行文档编辑
  - `@api.route()`装饰器
  - 对字段进行文档编写
  - 对函数进行文档编写
  - 级联（Cascading）
  - 已弃用标记
  - 隐藏文档
  - 文档授权（Documenting authorizations）
  - 展示支持信息（Expose vendor Extensions）
  - 导出Swagger格式
  - Swagger UI
- [Postman](#Postman) 
- [项目扩展](#项目扩展)
  - [多命名空间](#多命名空间)
  - [使用蓝图](#使用蓝图) 
  - [具有可重用名称空间的多个API（Multiple APIs with reusable namespaces）](#具有可重用名称空间的多个apimultiple-apis-with-reusable-namespaces)
- [完整实例](#完整实例)
- API

# 安装

使用 `pip`安装Flask-RESTPlus：

```shell
pip install flask-restplus
```

开发版可以从 [Github](https://github.com/noirbizarre/flask-restplus) 上下载：

```shell
git clone https://github.com/noirbizarre/flask-restplus.git
cd flask-restplus
pip install -e .[dev,test]
```

Flask-RESTPlus需要python 2.7, 3.3, 3.4 或 3.5支持。当然PyPy和PyPy3同样适用。

# 快速开始

本文档需要你对 Flask 的工作机制有所了解，同时请确保你已完成了Flask与Flask-RESTPlus的安装工作。如果没有完成安装，请参阅 [安装](#安装)。

## 初始化

和其他Flask扩展一样，你可以用 applicaiton 对象初始化它：

```python
from flask import Flask
from flask_restplus import Api

app = Flask(__name__)
api = Api(app)
```

或者使用懒加载模式：

```python
from flask import Flask
from flask_restplus import Api

api = Api()

app = Flask(__name__)
api.init_app(app)
```

## 一个最小API

一个最小的Flask-RESTPlus API 应该如下所示：

```python
from flask import Flask
from flask_restplus import Resource, Api

app = Flask(__name__)
api = Api(app)

@api.route('/hello')
class HelloWorld(Resource):
    def get(self):
        return {'hello': 'world'}

if __name__ == '__main__':
    app.run(debug=True)
```

将上述代码保存为 `api.py`并使用Python解释器运行。值得注意的是我们已经打开了Flask的调试模式（[Flask debugging](http://flask.pocoo.org/docs/quickstart/#debug-mode)）这将为我们提供代码热重载和更好的报错信息。

```shell
$ python api.py
* Running on http://127.0.0.1:5000/
* Restarting with reloader
```

> ### 警告：
>
> 调试模式不能用于任何生产环境下！

现在打开一个新窗口测试一下你的API：

```shell
$ curl http://127.0.0.1:5000/hello
{"hello": "world"}
```

你同时可以将自动文档作为你的API根目录（默认是开启的）。在本例子中，根目录为：http://127.0.0.1:5000/。查看 [Swagger UI](#Swagger UI) 章节获取完整信息。

## 面向资源（Resource）的路由

Flask-RESTPlus所能提供的主要组件由资源类（Resource）实现。资源类是在 [Flask可扩展视图（Flask pluggable views）](https://flask.palletsprojects.com/en/1.1.x/views/#views) 的基础上实现的，提供了添加指定函数就能实现多种http访问方法的功能。一个实现代办事项（todo）应用程序的基础CURD（译者：就是增删改查的意思）资源类如下所示：

```python
from flask import Flask, request
from flask_restplus import Resource, Api

app = Flask(__name__)
api = Api(app)

todos = {}

@api.route('/<string:todo_id>')
class TodoSimple(Resource):
    def get(self, todo_id):
        return {todo_id: todos[todo_id]}

    def put(self, todo_id):
        todos[todo_id] = request.form['data']
        return {todo_id: todos[todo_id]}

if __name__ == '__main__':
    app.run(debug=True)
```

测试一下：

```shell
$ curl http://localhost:5000/todo1 -d "data=Remember the milk" -X PUT
{"todo1": "Remember the milk"}
$ curl http://localhost:5000/todo1
{"todo1": "Remember the milk"}
$ curl http://localhost:5000/todo2 -d "data=Change my brakepads" -X PUT
{"todo2": "Change my brakepads"}
$ curl http://localhost:5000/todo2
{"todo2": "Change my brakepads"}
```

如果你的python安装了 [python-requests](http://docs.python-requests.org/) 你也可以用python进行测试：

```python
>>> from requests import put, get
>>> put('http://localhost:5000/todo1', data={'data': 'Remember the milk'}).json()
{u'todo1': u'Remember the milk'}
>>> get('http://localhost:5000/todo1').json()
{u'todo1': u'Remember the milk'}
>>> put('http://localhost:5000/todo2', data={'data': 'Change my brakepads'}).json()
{u'todo2': u'Change my brakepads'}
>>> get('http://localhost:5000/todo2').json()
{u'todo2': u'Change my brakepads'}
```

Flask-RESTPlus支持多种函数返回值。和 Flask 类似，你可以返回任意可迭代的对象（iterable）包括原生 Flask 响应对象（raw Flask response objects）而它们会被转化成响应（response）。Flask-RESTPlus同样支持使用多个返回值自定义状态码（response code）和响应头（response headers），示例如下：

```python
class Todo1(Resource):
    def get(self):
        # Default to 200 OK
        return {'task': 'Hello world'}

class Todo2(Resource):
    def get(self):
        # Set the response code to 201
        return {'task': 'Hello world'}, 201

class Todo3(Resource):
    def get(self):
        # Set the response code to 201 and return custom headers
        return {'task': 'Hello world'}, 201, {'Etag': 'some-opaque-string'}
```

## 端点（Endpoint）

在很多情况下，一个API中，你的一个资源类可能需要对应多个url地址。你可以传入多个url地址到 **add_resource()** 函数中或者使用 **route()** 装饰器，这两个方法都由`Api`对象提供。其中这的任意一个方法都能实现多url路由到你的资源类上：

```python
api.add_resource(HelloWorld, '/hello', '/world')

# or

@api.route('/hello', '/world')
class HelloWorld(Resource):
    pass
```

同时你也可以将一部分匹配路径作为变量传入你的资源类函数中：

```python
api.add_resource(Todo, '/todo/<int:todo_id>', endpoint='todo_ep')

# or

@api.route('/todo/<int:todo_id>', endpoint='todo_ep')
class HelloWorld(Resource):
    pass
```

> ### 提示：
>
> 如果请求无法匹配应用程序中的任何端点，Flask-RESTPlus 会返回404错误信息并附带最接近请求的端点猜测。你可以在配置文件中将 `ERROR_404_HELP`改为`False`来禁用这个功能。

## 参数解析

虽然 Flask 提供了很方便的途径访问请求数据（如：请求参数（querystring）或 POST表单数据（POST form encoded data）），但是验证表单数据依然很麻烦。Flask-RESTPlus提供了内置的表单验证功能，具体由一个类似于 [argparse](https://docs.python.org/3/library/argparse.html#module-argparse)  的库完成。

```python
from flask_restplus import reqparse

parser = reqparse.RequestParser()
parser.add_argument('rate', type=int, help='Rate to charge for this resource')
args = parser.parse_args()
```

> ### 提示：
>
> 和 [argparse](https://docs.python.org/3/library/argparse.html#module-argparse) 不同的是，`parse_args()` 返回的是一个Python字典对象，而不是一个自定义的数据结构

使用 **RequestParser**类同时也可以自动给你提供清晰的错误信息。如果一个参数在验证过程中出现问题，Flask-RESTPlus 将返回一个400错误并高亮错误。

```shell
$ curl -d 'rate=foo' http://127.0.0.1:5000/todos
{'status': 400, 'message': 'foo cannot be converted to int'}
```

`input`模块提供了大量转化函数，如 **date()** 和 **url()** 。

调用 **parse_args()** 时附带参数 `stict=True`，程序将在表单中出现解析器（parser）未定义的参数时抛出异常。

```python
args = parser.parse_args(strict=True)
```

## 数据格式化

在默认情况下，你返回的所有可迭代对象将会被按照原样呈现。在针对Python原生数据结构时这种策略能够很好的完成任务，但是在处理对象时效果令人沮丧。为了解决这个问题，Flask-RESTPlus 提供了 **字段（Field）**模块和 **marshal_with()**装饰器。和 Django ORM 与 WTForm 类似，你可以使用 `字段（fields）`模块来定义你的响应数据结构。

```python
from collections import OrderedDict

from flask import Flask
from flask_restplus import fields, Api, Resource

app = Flask(__name__)
api = Api(app)

model = api.model('Model', {
    'task': fields.String,
    'uri': fields.Url('todo_ep')
})

class TodoDao(object):
    def __init__(self, todo_id, task):
        self.todo_id = todo_id
        self.task = task

        # This field will not be sent in the response
        self.status = 'active'

@api.route('/todo')
class Todo(Resource):
    @api.marshal_with(model)
    def get(self, **kwargs):
        return TodoDao(todo_id='my_todo', task='Remember the milk')
```

上面的例子我们拿了一个Python对象并准备将其序列化。 **marlshal_with()** 装饰器将会将其通过 `model`进行转化。 **fields.Url**  字段是个很特别的字段，他将选取一个端点在响应中生成带该端点的URL。使用 **marlshal_with()** 同时也会为文档添加Swagger格式的文档内容。大部分字段类型已经预定义过。查看 `字段（fields）`导航获取完整列表。

### 指令保存

在默认情况下，字段指令（fields order）并不会被保存，因为这将影响性能。如果你任然希望字段指令被保存，你可以通过添加参数 `ordered=True`在某些类或者函数上进行强制保存：

- 在**Api**上全局设置：`api = Api(ordered=True)`

- 在 **命名空间（Namespace）** 上全局设置：`ns = Namespace(ordered=True)`

- 使用 **marshal()** 局部设置：return marshal(data, fields, ordered=True)

# 响应编组

# 字段掩码（Fields masks）

Flask-RESTPlus通过一个自定义请求头支持部分对象获取（partial object fetching）也就是字段掩码（fields mask）。

默认情况下头名为 `X-Fields` ，但是它可以被 `RESTPLUS_MASK_HEADER` 参数修改。

## 语法

语法非常简单。你只要提供一个包含字段名并用逗号分隔的列表，可以选择性的用括号包裹。

```python
# These two mask are equivalents
mask = '{name,age}'
# or
mask = 'name,age'
data = requests.get('/some/url/', headers={'X-Fields': mask})
assert len(data) == 2
assert 'name' in data
assert 'age' in data
```

实现嵌套字段只需要将内容用大括号包裹即可。

```python
mask = '{name, age, pet{name}}'
```

嵌套规范适用于嵌套对象或对象列表：

```python
# Will apply the mask {name} to each pet
# in the pets list.
mask = '{name, age, pets{name}}'
```

特殊字符米字星（*）代表“所有剩余字段（all remaining fields）”。它仅允许指定嵌套过滤：

```python
# Will apply the mask {name} to each pet
# in the pets list and take all other root fields
# without filtering.
mask = '{pets{name},*}'

# Will not filter anything
mask = '*'
```

## 使用方法

默认情况下，每次使用 **api.marshal** 或 **@api.marshal_with** 时，如果存在标头，掩码将自动应用。

当你每次使用 **@api.marshal_with** 修饰器时，标头将作为Swagger参数展示。

由于Swagger并不允许展示全局标头，这将会使你的Swagger文档更加冗长。你可以修改 `RESTPLUS_MASK_SWAGGER` 为 `False` 来禁用这个功能。

您也可以指定默认的掩码，如果找不到掩码，则将应用默认掩码。

```python
class MyResource(Resource):
    @api.marshal_with(my_model, mask='name,age')
    def get(self):
        pass
```

默认掩码也可以在模型级别处理：

```python
model = api.model('Person', {
    'name': fields.String,
    'age': fields.Integer,
    'boolean': fields.Boolean,
}, mask='{name,age}')
```

它将被导出到 `x-mask` 字段：

```json
{"definitions": {
    "Test": {
        "properties": {
            "age": {"type": "integer"},
            "boolean": {"type": "boolean"},
            "name": {"type": "string"}
        },
        "x-mask": "{name,age}"
    }
}}
```

要覆盖默认掩码，您需要提供另一个掩码或将*用作掩码。

# 异常处理

## Http异常处理

Werkzeug HTTP异常（Werkzeug HTTPException）将适当的自动重写描述（description）属性。

```python
from werkzeug.exceptions import BadRequest
raise BadRequest()
```

上述代码将会返回一个400状态码和输出：

```json
{
    "message": "The browser (or proxy) sent a request that this server could not understand."
}
```

而这段代码：

```python
from werkzeug.exceptions import BadRequest
raise BadRequest('My custom message')
```

将会输出：

```json
{
    "message": "My custom message"
}
```

你可以通过修改data属性值添加额外的参数到你的异常中：

```python
from werkzeug.exceptions import BadRequest
e = BadRequest('My custom message')
e.data = {'custom': 'value'}
raise e
```

这将输出：

```json
{
    "message": "My custom message",
    "custom": "value"
}
```

## Flask终止助手

**终止助手（abort helper）** 适当的将错误封装成了一个 **Http异常（ HTTPException ）** ，因此它们表现几乎相同。

```python
from flask import abort
abort(400)
```

上述代码将会返回一个400状态码和输出：

```json
{
    "message": "The browser (or proxy) sent a request that this server could not understand."
}
```

而这段代码：

```python
from flask import abort
abort(400, 'My custom message')
```

将会输出：

```json
{
    "message": "My custom message"
}
```

## Flask-RESTPlus终止助手

**errors.abort()** 与 **Namespace.abort()** 终止助手与原生Flask **Flask.abort()** 原理相似，但将会把关键字参数（keyword arguments）打包进响应中。

```python
from flask_restplus import abort
abort(400, custom='value')
```

上述代码将会返回一个400状态码和输出：

```json
{
    "message": "The browser (or proxy) sent a request that this server could not understand.",
    "custom": "value"
}
```

而这段代码：

```python
from flask import abort
abort(400, 'My custom message', custom='value')
```

将会输出：

```json
{
    "message": "My custom message",
    "custom": "value"
}
```

## `@api.errorhandler` 装饰器

`@api.errorhandler`装饰器将为指定的异常（或继承于这个异常的任何异常）注册一个特别的处理机（handler），你也可以用同样的方法使用 Flask/Blueprint 的 `@errorhandler`装饰器。

```python
@api.errorhandler(RootException)
def handle_root_exception(error):
    '''Return a custom message and 400 status code'''
    return {'message': 'What you want'}, 400


@api.errorhandler(CustomException)
def handle_custom_exception(error):
    '''Return a custom message and 400 status code'''
    return {'message': 'What you want'}, 400


@api.errorhandler(AnotherException)
def handle_another_exception(error):
    '''Return a custom message and 500 status code'''
    return {'message': error.specific}


@api.errorhandler(FakeException)
def handle_fake_exception_with_header(error):
    '''Return a custom message and 400 status code'''
    return {'message': error.message}, 400, {'My-Header': 'Value'}


@api.errorhandler(NoResultFound)
def handle_no_result_exception(error):
    '''Return a custom not found error message and 404 status code'''
    return {'message': error.specific}, 404
```

> ### 提示：
>
> 一个带有描述的“未找到结果（NoResultFound）”异常是 OpenAPI 2.0规范所规定的。处理机种的文档字符串（docstring）（译者：就是那坨三引号）将会被输出到swagger.json中作为描述。

你也可以专门为异常编写文档：

```python
@api.errorhandler(FakeException)
@api.marshal_with(error_fields, code=400)
@api.header('My-Header',  'Some description')
def handle_fake_exception_with_header(error):
    '''This is a custom error'''
    return {'message': error.message}, 400, {'My-Header': 'Value'}


@api.route('/test/')
class TestResource(Resource):
    def get(self):
        '''
        Do something

        :raises CustomException: In case of something
        '''
        pass
```

在这个例子里，`:raises:`字符串将会被自动提取并填充至文档400错误处。

你只需要在使用时忽略参数，就可以重写默认的 `errorhandler`：

```python
@api.errorhandler
def default_error_handler(error):
    '''Default error handler'''
    return {'message': str(error)}, getattr(error, 'code', 500)
```

> ### 提示：
>
> Flask-RESTPlus 默认会返回一个带消息的错误信息。如果你在自定义一个错误响应并且不需要消息字段的话，你可以在配置文件中将 `ERROR_INCLUDE_MESSAGE`  改为 `False` 以禁用消息字段。

`errorhandler`也可以在命名空间中注册。在命名空间中注册的处理机将会覆盖在 **Api** 中注册的处理机。

```python
ns = Namespace('cats', description='Cats related operations')

@ns.errorhandler
def specific_namespace_error_handler(error):
    '''Namespace error handler'''
    return {'message': str(error)}, getattr(error, 'code', 500)
```

# Postman

为了方便你测试，你可以将你的API导出为一个 [Postman](https://www.getpostman.com/) 集合（Postman collection）。

```python
from flask import json

from myapp import api

urlvars = False  # Build query strings in URLs
swagger = True  # Export Swagger specifications
data = api.as_postman(urlvars=urlvars, swagger=swagger)
print(json.dumps(data))
```

# 项目扩展

本章涵盖构建稍微复杂一些的Flask-RESTPlus应用程序，该程序将为在实际部署一个基于Flask-RESTPlus的应用程序前提供一些最佳的实践。[快速开始](#快速开始) 章节更加适合第一次接触Flask-RESTPlus的人，所以如果你是Flask-RESTPlus的初学者请先查看快速开始章节。

## 多命名空间

这里有很多种方法组织你的Flask-RESTPlus应用程序代码，而下面我们将介绍一种在大型应用程序种保持良好的组织性和优秀的可扩展性的一种方法。

Flask-RESTPlus 提供了一种几乎与 Flask蓝图（Flask’s blueprint）相同的功能。它的主要思路是将应用程序分割为多个可重用的 命名空间（Namespace）。

下面是一个目录结构的例子：

```shell
project/
├── app.py
├── core
│   ├── __init__.py
│   ├── utils.py
│   └── ...
└── apis
    ├── __init__.py
    ├── namespace1.py
    ├── namespace2.py
    ├── ...
    └── namespaceX.py
```

`app` 模块将作为一个遵循经典的Flask模式之一（详见 [Larger Applications](https://flask.palletsprojects.com/en/1.1.x/patterns/packages/#larger-applications) and [Application Factories](https://flask.palletsprojects.com/en/1.1.x/patterns/appfactories/#app-factories)）的主程序入口（entry point）。

`core` 模块包含了事务逻辑。实际上你爱叫啥叫啥，甚至可以是很多个包。

`apis` 模块将作为你的API的主入口，你需要在这里导入并注册你的应用程序，而命名空间模块就按照你平时写Flask蓝图的时候写就行。

一个命名空间的模块要包含模型和资源类的声明。举个例子：

```python
from flask_restplus import Namespace, Resource, fields

api = Namespace('cats', description='Cats related operations')

cat = api.model('Cat', {
    'id': fields.String(required=True, description='The cat identifier'),
    'name': fields.String(required=True, description='The cat name'),
})

CATS = [
    {'id': 'felix', 'name': 'Felix'},
]

@api.route('/')
class CatList(Resource):
    @api.doc('list_cats')
    @api.marshal_list_with(cat)
    def get(self):
        '''List all cats'''
        return CATS

@api.route('/<id>')
@api.param('id', 'The cat identifier')
@api.response(404, 'Cat not found')
class Cat(Resource):
    @api.doc('get_cat')
    @api.marshal_with(cat)
    def get(self, id):
        '''Fetch a cat given its identifier'''
        for cat in CATS:
            if cat['id'] == id:
                return cat
        api.abort(404)
```

`apis.__init__` 模块应该用于整合他们：

```python
from flask_restplus import Api

from .namespace1 import api as ns1
from .namespace2 import api as ns2
# ...
from .namespaceX import api as nsX

api = Api(
    title='My Title',
    version='1.0',
    description='A description',
    # All API metadatas
)

api.add_namespace(ns1)
api.add_namespace(ns2)
# ...
api.add_namespace(nsX)
```

你可以在注册API的时候为你的命名空间添加 `网址前缀（url-prefixes）` 。你不需要在定义命名空间对象的时候指定 `网址前缀（url-prefixes）` 。

```python
from flask_restplus import Api

from .namespace1 import api as ns1
from .namespace2 import api as ns2
# ...
from .namespaceX import api as nsX

api = Api(
    title='My Title',
    version='1.0',
    description='A description',
    # All API metadatas
)

api.add_namespace(ns1, path='/prefix/of/ns1')
api.add_namespace(ns2, path='/prefix/of/ns2')
# ...
api.add_namespace(nsX, path='/prefix/of/nsX')
```

用这种模式，你只需要在 `app.py` 中这样注册你的API即可：

```python
from flask import Flask
from apis import api

app = Flask(__name__)
api.init_app(app)

app.run(debug=True)
```

## 使用蓝图

查看Flask文档中的 “[使用蓝图进行模块化编程（ Modular Applications with Blueprints）](https://flask.palletsprojects.com/en/1.1.x/blueprints/#blueprints)” 以了解什么是蓝图和为什么使用蓝图。以下是一个将 **Api** 链接到 **蓝图（Blueprint）** 上的例子：

```python
from flask import Blueprint
from flask_restplus import Api

blueprint = Blueprint('api', __name__)
api = Api(blueprint)
# ...
```

使用蓝图将使你可以将你的API挂载到任何网址前缀（url-prefixes）下，如/或你的应用程序的子域名：

```python
from flask import Flask
from apis import blueprint as api

app = Flask(__name__)
app.register_blueprint(api, url_prefix='/api/1')
app.run(debug=True)
```

> ### 提示：
>
> 由于在注册蓝图的时候会处理应用程序的路由设置，所以不需要调用 **Api.init_app()** 。 

> ### 提示：
>
> 在使用蓝图时，请记住调用 **url_for()** 时加上蓝图名：
>
> ```python
> # without blueprint
> url_for('my_api_endpoint')
> 
> # with blueprint
> url_for('api.my_api_endpoint')
> ```

## 具有可重用名称空间的多个API（Multiple APIs with reusable namespaces）

有时候你可能需要维护一个API的多个版本。如果你使用的时命名空间来搭建你的API，那么将其扩展成多个API是个很简单的事情。

根据先前的布局，我们可以将其迁移到以下目录结构：

```shell
project/
├── app.py
├── apiv1.py
├── apiv2.py
└── apis
    ├── __init__.py
    ├── namespace1.py
    ├── namespace2.py
    ├── ...
    └── namespaceX.py
```

每个 `apivX.py` 都拥有如下的结构：

```python
from flask import Blueprint
from flask_restplus import Api

api = Api(blueprint)

from .apis.namespace1 import api as ns1
from .apis.namespace2 import api as ns2
# ...
from .apis.namespaceX import api as nsX

blueprint = Blueprint('api', __name__, url_prefix='/api/1')
api = Api(blueprint
    title='My Title',
    version='1.0',
    description='A description',
    # All API metadatas
)

api.add_namespace(ns1)
api.add_namespace(ns2)
# ...
api.add_namespace(nsX)
```

而 `app` 将这样挂载它们：

```python
from flask import Flask
from api1 import blueprint as api1
from apiX import blueprint as apiX

app = Flask(__name__)
app.register_blueprint(api1)
app.register_blueprint(apiX)
app.run(debug=True)
```

这些只是建议，你可以根据你的需求随意修改。查看  [github repository examples folder](https://github.com/noirbizarre/flask-restplus/tree/master/examples)  获取更多完整实例。

# 完整实例

下面是 [TodoMVC](http://todomvc.com/)  的完整API实例：

```python
from flask import Flask
from flask_restplus import Api, Resource, fields
from werkzeug.contrib.fixers import ProxyFix

app = Flask(__name__)
app.wsgi_app = ProxyFix(app.wsgi_app)
api = Api(app, version='1.0', title='TodoMVC API',
    description='A simple TodoMVC API',
)

ns = api.namespace('todos', description='TODO operations')

todo = api.model('Todo', {
    'id': fields.Integer(readonly=True, description='The task unique identifier'),
    'task': fields.String(required=True, description='The task details')
})


class TodoDAO(object):
    def __init__(self):
        self.counter = 0
        self.todos = []

    def get(self, id):
        for todo in self.todos:
            if todo['id'] == id:
                return todo
        api.abort(404, "Todo {} doesn't exist".format(id))

    def create(self, data):
        todo = data
        todo['id'] = self.counter = self.counter + 1
        self.todos.append(todo)
        return todo

    def update(self, id, data):
        todo = self.get(id)
        todo.update(data)
        return todo

    def delete(self, id):
        todo = self.get(id)
        self.todos.remove(todo)


DAO = TodoDAO()
DAO.create({'task': 'Build an API'})
DAO.create({'task': '?????'})
DAO.create({'task': 'profit!'})


@ns.route('/')
class TodoList(Resource):
    '''Shows a list of all todos, and lets you POST to add new tasks'''
    @ns.doc('list_todos')
    @ns.marshal_list_with(todo)
    def get(self):
        '''List all tasks'''
        return DAO.todos

    @ns.doc('create_todo')
    @ns.expect(todo)
    @ns.marshal_with(todo, code=201)
    def post(self):
        '''Create a new task'''
        return DAO.create(api.payload), 201


@ns.route('/<int:id>')
@ns.response(404, 'Todo not found')
@ns.param('id', 'The task identifier')
class Todo(Resource):
    '''Show a single todo item and lets you delete them'''
    @ns.doc('get_todo')
    @ns.marshal_with(todo)
    def get(self, id):
        '''Fetch a given resource'''
        return DAO.get(id)

    @ns.doc('delete_todo')
    @ns.response(204, 'Todo deleted')
    def delete(self, id):
        '''Delete a task given its identifier'''
        DAO.delete(id)
        return '', 204

    @ns.expect(todo)
    @ns.marshal_with(todo)
    def put(self, id):
        '''Update a task given its identifier'''
        return DAO.update(id, api.payload)


if __name__ == '__main__':
    app.run(debug=True)
```

你可以在  [github repository examples folder](https://github.com/noirbizarre/flask-restplus/tree/master/examples)  获取更多完整实例。