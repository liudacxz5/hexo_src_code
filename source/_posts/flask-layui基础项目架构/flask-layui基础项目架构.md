---
title: flask+layui基础项目架构
date: 2025-04-05 15:16:54
updated: 2025-04-05 15:16:54
tags: 
 - flask 
 - layui
categories: 
 - 技术
 - flask
keywords: 
 - flask
 - layui
description: 简述了docker容器的重启策略
top_img: 
comments: true
cover:
highlight_shrink: true
---
## **Flask** + **Layui** 基础项目架构
**Flask**（后端）和 **Layui**（前端UI框架）的典型项目结构设计，兼顾模块化和可维护性：

---

### **项目结构示例**

```bash
/my_flask_layui_project
├── app/                  # Flask核心代码
│   ├── __init__.py       # 应用工厂
│   ├── routes/           # 路由（使用蓝图）
│   │   ├── home.py       # 首页路由
│   │   ├── user.py       # 用户管理路由
│   │   └── api.py        # 数据接口API（供Layui表格/表单调用）
│   ├── static/           # 静态资源
│   │   ├── layui/        # Layui框架文件（从官网下载的完整包）
│   │   │   ├── css/
│   │   │   ├── js/
│   │   │   └── fonts/
│   │   ├── css/          # 自定义CSS
│   │   │   └── style.css
│   │   └── js/           # 自定义JavaScript
│   │       └── main.js
│   ├── templates/        # Jinja2模板
│   │   ├── layout.html   # 基础模板（引入Layui）
│   │   ├── home/         # 页面模块
│   │   │   └── index.html
│   │   └── user/
│   │       ├── list.html # 用户列表（使用Layui表格）
│   │       └── edit.html # 用户编辑（使用Layui表单）
│   ├── models.py         # 数据库模型
│   └── config.py         # 配置文件
│
├── tests/                # 单元测试
├── venv/                 # 虚拟环境
├── requirements.txt      # 依赖列表
└── run.py                # 启动脚本
```

---

### **关键文件说明**

#### **1. Flask相关**
- **`app/__init__.py`**：应用工厂，初始化Flask实例并注册蓝图。
  ```python
  from flask import Flask
  from .config import Config

  def create_app():
      app = Flask(__name__)
      app.config.from_object(Config)

      # 注册蓝图
      from .routes.home import home_bp
      from .routes.user import user_bp
      from .routes.api import api_bp
      app.register_blueprint(home_bp)
      app.register_blueprint(user_bp, url_prefix='/user')
      app.register_blueprint(api_bp, url_prefix='/api')

      return app
  ```

- **`routes/api.py`**：提供Layui组件需要的数据接口。
  ```python
  from flask import Blueprint, jsonify
  api_bp = Blueprint('api', __name__)

  @api_bp.route('/user/list')
  def user_list():
      # 返回Layui表格要求的格式：{ "code":0, "msg":"", "count":100, "data":[...] }
      data = [{"id":1, "username":"admin"}, ...]
      return jsonify(code=0, msg="success", count=len(data), data=data)
  ```

---

#### **2. Layui相关**
- **`static/layui/`**：存放Layui官方库（需从[官网](https://www.layuiweb.com/)下载完整包）。
- **`templates/layout.html`**：基础模板，统一引入Layui资源。
  ```html
  <!DOCTYPE html>
  <html>
<head>
    <meta charset="utf-8">
    <title>{% block title %}{% endblock %}</title>
    <!-- 引入 jQuery -->
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <!-- 引入Layui CSS -->
    <link rel="stylesheet" href="../static/layui/css/layui.css">
    <!-- 引入Layui JS -->
    <script src="../static/layui/layui.js"></script>
</head>
<body>
{% block content %}{% endblock %}

</body>
  </html>
  ```

- **`templates/user/list.html`**：使用Layui表格的页面。
  ```html
  {% extends "layout.html" %}
  {% block title %}用户列表{% endblock %}
  {% block content %}
  <div class="layui-card">
      <div class="layui-card-body">
          <table id="userTable" lay-filter="userTable"></table>
      </div>
  </div>
  <script>
      layui.table.render({
          elem: '#userTable',
          cols: [[
              {field: 'id', title: 'ID'},
              {field: 'username', title: '用户名'}
          ]]
      });
  </script>
  {% endblock %}
  ```

---

### **前后端交互示例**
1. **Layui表格数据加载**：通过Flask的`/api/user/list`接口返回符合Layui格式的JSON数据。
2. **表单提交**：使用Layui的`form.on('submit')`监听事件，通过AJAX调用Flask接口。
3. **静态资源管理**：所有Layui文件通过`static/layui/`目录统一管理，模板中用`url_for`动态生成路径。

---

### **最佳实践**
1. **模块化开发**：使用Flask蓝图拆分功能（如用户管理、API模块）。
2. **前后端分离**：API接口返回JSON数据，前端Layui负责渲染。
3. **版本控制**：将Layui库文件锁定特定版本（如`static/layui-v2.8.11/`）。
4. **生产优化**：
   - 使用Nginx托管静态文件（减少Flask负担）。
   - 通过`FLASK_ENV=production`禁用调试模式。

---

此结构清晰划分了前后端职责，适合中小型项目快速开发。如需扩展，可进一步添加`utils`（工具类）、`middlewares`（中间件）等目录。