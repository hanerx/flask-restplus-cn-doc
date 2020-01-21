# flask-restplus 中文文档

本文档是Flask-RESTPlus的文档中文翻译版，英文原版地址：https://flask-restplus.readthedocs.io/en/stable/

# 相关信息

- 文档对应版本：0.13.0
- 本文档是个人翻译版本，非官方翻译版，一切内容以官方英文原版为准。
- 本文档与Flask-RESTPlus开发组无任何关系，提交bug等开发问题请移步项目所在地：https://github.com/noirbizarre/flask-restplus

# 目录

- 安装
- 快速开始

  - 初始化

  - 一个最小API

  - 面向资源（Resource）的路由

  - 端点（Endpoint）

  - 参数解析

  - 数据格式

- 响应编组
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
  - 异常处理
  - 异常消息
- 异常处理
  - Http异常处理
  - Flask终止助手
  - Flask-RESTPlus终止助手
  - `@api.errorhandler`装饰器
- 字段影藏（Fields masks）
  - 语法
  - 使用方法
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
- Postman
- 项目扩展
  - 多个命名空间
  - 使用蓝图
  - 具有可重用名称空间的多个API（Multiple APIs with reusable namespaces）
- 完整例子
- API
