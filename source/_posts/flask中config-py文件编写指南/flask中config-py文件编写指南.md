---
title: flask中config.py文件编写指南
date: 2025-04-05 15:50:35
updated: 2025-04-05 15:50:35
tags: 
 - flask 
 - config
categories: 
 - 技术
 - flask
keywords:  
 - flask 
 - config
description: flask中config.py文件编写指南
top_img: 
comments: true
cover:
highlight_shrink: true
---
在 Flask 项目中，`config.py` 文件用于集中管理应用的配置参数（如数据库连接、密钥、调试模式等）。以下是一个典型的配置文件的编写方式和最佳实践：

---

### **1. 基础 `config.py` 示例**
```python
# config.py
import os
from dotenv import load_dotenv  # 可选：用于加载.env文件

# 加载.env文件（如果存在）
load_dotenv()

class Config:
    # 基础配置（通用配置）
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'your-default-secret-key'
    SQLALCHEMY_TRACK_MODIFICATIONS = False  # 禁用SQLAlchemy事件系统（推荐关闭）
    DEBUG = False  # 默认关闭调试模式

class DevelopmentConfig(Config):
    # 开发环境配置
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DATABASE_URL') or \
        'sqlite:///' + os.path.join(os.path.dirname(__file__), 'data-dev.sqlite')

class TestingConfig(Config):
    # 测试环境配置
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TEST_DATABASE_URL') or 'sqlite:///:memory:'

class ProductionConfig(Config):
    # 生产环境配置
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(os.path.dirname(__file__), 'data.sqlite')

# 配置类映射（便于动态选择）
config = {
    'development': DevelopmentConfig,
    'testing': TestingConfig,
    'production': ProductionConfig,
    'default': DevelopmentConfig
}
```

---

### **2. 关键配置项说明**
#### **a. 必须配置项**
- **`SECRET_KEY`**：用于加密会话、CSRF令牌等敏感操作。
  ```python
  SECRET_KEY = os.environ.get('SECRET_KEY') or 'hardcoded-fallback-key'  # 优先从环境变量读取
  ```
  - **生产环境中务必使用环境变量**，避免硬编码密钥！

#### **b. 数据库配置**
- **`SQLALCHEMY_DATABASE_URI`**：数据库连接字符串。
  ```python
  # 示例：
  # PostgreSQL: postgresql://user:password@localhost/mydb
  # MySQL: mysql+pymysql://user:password@localhost/mydb
  # SQLite: sqlite:///absolute/path/to/db.sqlite
  SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, 'app.db')
  ```

#### **c. 调试与安全**
- **`DEBUG`**：开发时设为 `True`，生产环境必须设为 `False`。
- **`TESTING`**：单元测试时启用，会禁用某些安全检查。

---

### **3. 环境变量与敏感信息**
#### **a. 使用 `.env` 文件（推荐）**
1. 安装 `python-dotenv`：
   ```bash
   pip install python-dotenv
   ```
2. 在项目根目录创建 `.env` 文件（**不要提交到版本控制**）：
   ```bash
   SECRET_KEY=your-secret-key-here
   DATABASE_URL=postgresql://user:password@localhost/prod_db
   DEV_DATABASE_URL=sqlite:///dev.db
   ```
3. 在 `config.py` 中加载：
   ```python
   from dotenv import load_dotenv
   load_dotenv()  # 自动加载.env文件
   ```

#### **b. 生产环境直接设置环境变量**
```bash
# Linux/macOS
export SECRET_KEY=your-secret-key
export DATABASE_URL=postgresql://user:password@host/db

# Windows
set SECRET_KEY=your-secret-key
set DATABASE_URL=postgresql://user:password@host/db
```

---

### **4. 在Flask应用中使用配置**
在 `app/__init__.py` 的工厂函数中动态加载配置：
```python
from flask import Flask
from config import config  # 导入config字典

def create_app(config_name='default'):
    app = Flask(__name__)
    
    # 从配置类加载设置
    app.config.from_object(config[config_name])
    
    # 可选：从instance文件夹加载私有配置
    # app.config.from_pyfile('config.py', silent=True)
    
    return app
```

启动时指定环境（如生产环境）：
```python
# run.py
from app import create_app

app = create_app('production')

if __name__ == '__main__':
    app.run()
```

---

### **5. 最佳实践**
1. **分离不同环境的配置**：开发、测试、生产环境使用不同的数据库和参数。
2. **敏感信息不上传**：通过 `.gitignore` 排除 `.env` 和 `instance/` 文件夹。
3. **使用实例文件夹（Instance Folder）**：
   - 创建 `instance` 目录（与 `app` 同级）
   - 在 `instance/config.py` 中存放生产环境私有配置：
     ```python
     # instance/config.py
     SECRET_KEY = 'production-secret-key'
     DATABASE_URL = 'postgresql://user:password@host/db'
     ```
   - 加载配置：
     ```python
     app.config.from_pyfile('config.py', silent=True)  # silent=True表示文件不存在时不报错
     ```

---

### **6. 完整配置示例**
```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

basedir = os.path.abspath(os.path.dirname(__file__))

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-fallback-key'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    MAIL_SERVER = os.environ.get('MAIL_SERVER', 'smtp.googlemail.com')
    MAIL_PORT = int(os.environ.get('MAIL_PORT', '587'))
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS', 'true').lower() in ['true', 'on', '1']
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')

class DevelopmentConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DB_URL') or \
        'sqlite:///' + os.path.join(basedir, 'dev.db')

class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'prod.db')

config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'default': DevelopmentConfig
}
```

---

通过这种方式，你可以轻松管理不同环境的配置，同时确保敏感信息的安全性。