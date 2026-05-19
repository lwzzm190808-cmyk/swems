
<p align="center">
  <h1 align="center">SWEMS</h1>
  <p align="center"><strong>Site-Wide Extension Management System</strong></p>
  <p align="center">通用网站扩展管理系统 · v0.1.0 Beta</p>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-0.1.0--Beta-blue" alt="version">
  <img src="https://img.shields.io/badge/python-3.10+-green" alt="python">
  <img src="https://img.shields.io/badge/django-5.0-green" alt="django">
  <img src="https://img.shields.io/badge/license-MIT-yellow" alt="license">
</p>

## 作者

- **开发者：** THIRTEEN
- **邮箱：** 1481031171@qq.com
- **GitHub：** [https://github.com/yourname](https://github.com/lwzzm190808-cmyk/swems/tree/main)


---

## 目录

- [简介](#简介)
- [系统架构](#系统架构)
- [快速开始](#快速开始)
- [部署指南](#部署指南)
- [后台管理](#后台管理)
- [标签系统](#标签系统)
- [插件开发](#插件开发)
- [插件沙盒](#插件沙盒)
- [权限管理](#权限管理)
- [系统配置](#系统配置)
- [服务管理](#服务管理)
- [开发指南](#开发指南)
- [常见问题](#常见问题)

---

## 简介

SWEMS 是一个通用的网站扩展管理系统。它不是面向终端用户的成品系统，而是**面向系统管理员和开发者的扩展平台**。

**核心能力：**

- **标签系统** — 通过配置 SQL 查询、Model 关联或脚本，从数据库中提取数据，生成可在前端模板中使用的标签变量
- **插件系统** — 可安装、启用、停用、卸载的插件机制，支持插件间的 API 调用和数据交互
- **权限管理** — 三级用户类型（超级管理员/管理员/普通用户）+ 自定义权限组，按功能模块分配权限
- **模板管理** — 上传和管理前端模板文件
- **系统配置** — 通过后台界面管理网站、数据库、Redis、邮件等全部配置
- **操作日志** — 全站操作审计，登录日志、安全日志、运行时日志分类记录
- **插件沙盒** — 隔离运行环境，监控网络访问、数据库查询、资源使用，防止恶意插件

**技术栈：**

| 组件 | 技术 |
|------|------|
| 后端框架 | Django 5.0 + Django REST Framework |
| 数据库 | MySQL 8.0 |
| 缓存 / 消息队列 | Redis |
| 异步任务 | Celery |
| 进程管理 | Supervisor |
| 反向代理 | Nginx |
| 前端 | 原生 HTML + CSS（后台管理面板） |

---

## 系统架构

```
edu_platform/
├── config/                     # Django 项目配置
│   ├── settings.py             # 主配置文件
│   ├── urls.py                 # 根 URL 路由
│   ├── celery.py               # Celery 配置
│   └── wsgi.py / asgi.py
├── core/                       # 核心应用
│   ├── models.py               # 全部数据模型
│   ├── services/               # 业务服务层
│   │   ├── tag_engine.py       # 标签解析引擎
│   │   ├── plugin_manager.py   # 插件管理器
│   │   └── log_service.py      # 日志服务
│   ├── middleware/              # 中间件
│   │   ├── operation_logger.py # 操作日志中间件
│   │   └── permission_check.py # 权限检查中间件
│   └── management/commands/
│       └── init_site.py        # 初始化脚本
├── modules/                    # 功能模块
│   ├── tag/                    # 标签模块
│   ├── template/               # 模板模块
│   ├── config/                 # 配置模块
│   ├── permission/             # 权限模块
│   ├── plugin/                 # 插件模块
│   │   ├── loader.py           # 插件加载器（含沙盒集成）
│   │   └── installer.py        # 插件安装器
│   └── log/                    # 日志模块
├── shared/                     # 共享模块
│   ├── sandbox/                # 插件沙盒
│   │   └── monitor.py          # 沙盒监控器
│   └── notification/           # 通知模块
├── admin_panel/                # 后台管理面板
│   ├── views.py                # 后台视图
│   ├── urls.py                 # 后台路由
│   └── context_processors.py
├── frontend/                   # 前台应用
├── templates/                  # 模板文件
│   └── admin/                  # 后台模板
├── static/                     # 静态资源
├── plugins/                    # 插件安装目录
├── logs/                       # 日志目录
├── scripts/                    # 辅助脚本
├── swems.sh                    # 服务管理脚本
├── manage.py
└── requirements.txt
```

---

## 快速开始

### 环境要求

- Python 3.10+
- MySQL 8.0+
- Redis 6.0+
- Node.js 18+（可选，用于前端资源构建）

### 本地开发

```bash
# 1. 克隆项目
git clone <repo-url> edu_platform
cd edu_platform

# 2. 创建虚拟环境
python3 -m venv .venv
source .venv/bin/activate

# 3. 安装依赖
pip install -r requirements.txt

# 4. 创建数据库
mysql -u root -e "CREATE DATABASE edu_platform CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -e "CREATE USER 'edu_user'@'localhost' IDENTIFIED BY 'your_password';"
mysql -u root -e "GRANT ALL PRIVILEGES ON edu_platform.* TO 'edu_user'@'localhost';"
mysql -u root -e "FLUSH PRIVILEGES;"

# 5. 修改配置
# 编辑 config/settings.py 中的数据库连接信息

# 6. 初始化数据库
python manage.py makemigrations core
python manage.py migrate

# 7. 初始化基础数据（配置项、标签、权限组）
python manage.py init_site

# 8. 启动开发服务器
python manage.py runserver 0.0.0.0:8000
```

访问 `http://localhost:8000/admin-panel/` 进入后台管理面板。

---

## 部署指南

### 一键部署

```bash
# 环境检查
sudo ./swems.sh --check

# 打包（在开发机器上）
./swems.sh --package

# 部署（在目标服务器上）
sudo ./swems.sh --deploy

# 启动所有服务
sudo ./swems.sh --start

# 安装为 systemd 服务（开机自启）
sudo ./swems.sh --install-service
```

### 手动部署

```bash
# 1. 安装系统依赖
sudo apt update
sudo apt install -y python3 python3-pip python3-venv \
    mysql-server redis-server nginx supervisor

# 2. 创建项目目录
sudo mkdir -p /opt/swems
sudo chown $USER:$USER /opt/swems

# 3. 复制项目文件到 /opt/swems/

# 4. 创建虚拟环境并安装依赖
cd /opt/swems
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install gunicorn celery mysqlclient psutil

# 5. 配置数据库
# 编辑 config/settings.py

# 6. 初始化
python manage.py migrate
python manage.py collectstatic --noinput
python manage.py init_site

# 7. 配置 Supervisor
# 参考下方 Supervisor 配置

# 8. 配置 Nginx
# 参考下方 Nginx 配置

# 9. 重启服务
sudo supervisorctl reread
sudo supervisorctl update
sudo systemctl restart nginx
```

### Supervisor 配置

```ini
[program:swems]
command=/opt/swems/.venv/bin/gunicorn config.wsgi:application
    --bind 127.0.0.1:8000
    --workers 4
    --timeout 120
    --access-logfile /opt/swems/logs/access.log
    --error-logfile /opt/swems/logs/error.log
directory=/opt/swems
user=www-data
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/opt/swems/logs/supervisor_app.log

[program:swems-celery]
command=/opt/swems/.venv/bin/celery -A config worker -l info -c 2
directory=/opt/swems
user=www-data
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/opt/swems/logs/supervisor_celery.log
```

### Nginx 配置

```nginx
server {
    listen 80;
    server_name your-domain.com;

    client_max_body_size 50M;

    location /static/ {
        alias /opt/swems/staticfiles/;
        expires 30d;
    }

    location /media/ {
        alias /opt/swems/media/;
        expires 7d;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 60s;
        proxy_read_timeout 120s;
    }
}
```

---

## 后台管理

后台管理面板地址：`http://your-domain.com/admin-panel/`

### 功能模块

| 模块 | 路径 | 说明 |
|------|------|------|
| 仪表盘 | `/admin-panel/` | 系统概览、资源监控、快捷操作 |
| 用户管理 | `/admin-panel/users/` | 用户增删改查、类型筛选、权限组分配 |
| 权限管理 | `/admin-panel/permissions/` | 权限组管理、按模块分配权限 |
| 标签管理 | `/admin-panel/tags/` | 标签增删改查、数据源配置、在线测试 |
| 模板管理 | `/admin-panel/templates/` | 模板文件上传和管理 |
| 配置管理 | `/admin-panel/settings/` | 系统配置分组管理 |
| 插件管理 | `/admin-panel/plugins/` | 插件安装、启用、停用、卸载、API 调用 |
| 插件权限 | `/admin-panel/plugin-permissions/` | 插件安装时的权限申请审批 |
| 沙盒监控 | `/admin-panel/sandbox/` | 插件行为监控、安全级别配置 |
| 操作日志 | `/admin-panel/logs/` | 全站操作审计日志 |
| 网站设置 | `/admin-panel/site/` | 网站名称、标语、版权等 |
| 系统更新 | `/admin-panel/update/` | 系统版本检查和更新 |

### 仪表盘

仪表盘提供系统运行状态概览：

- CPU、内存、磁盘使用率
- 快捷操作入口（标签解析测试、插件管理、服务重启）
- 插件 API 测试面板
- 最近操作日志
- 服务器状态信息

### 用户类型

| 类型 | user_type | 说明 |
|------|-----------|------|
| 超级管理员 | admin | 拥有全部权限，可访问后台，最后一个不可删除 |
| 后台管理员 | admin | 根据权限组访问后台功能模块 |
| 前台用户 | frontend | 注册的普通用户，默认只能访问前台 |

---

## 标签系统

标签是 SWEMS 的核心能力之一。标签将数据库中的数据提取为可复用的变量，供前端模板引用。

### 标签类型

| 类型 | source_type | 说明 |
|------|-------------|------|
| 自定义查询 | `custom_query` | 执行 SQL 查询获取数据 |
| Model 字段 | `model_field` | 从 Django Model 中读取字段值或计数 |
| 脚本 | `script` | 执行 Python 脚本（高级用法） |
| 手动输入 | `manual` | 手动设定固定值 |

### 创建标签

**后台创建：** 进入 `/admin-panel/tags/`，点击「创建标签」。

**代码创建：**

```python
from core.models import Label

# 1. SQL 查询标签 — 网站名称
Label.objects.create(
    name='网站名称',
    tag_code='site.name',
    source_type='custom_query',
    source_config={
        'sql': "SELECT value FROM sys_config WHERE `group`='site' AND `key`='site_name' LIMIT 1"
    },
    is_system=True,
)

# 2. SQL 查询标签 — 注册用户数
Label.objects.create(
    name='注册用户数',
    tag_code='stats.user_count',
    source_type='custom_query',
    source_config={
        'sql': "SELECT COUNT(*) AS cnt FROM sys_user WHERE is_active=1"
    },
    is_system=True,
)

# 3. Model 字段标签
Label.objects.create(
    name='启用插件数',
    tag_code='stats.plugin_count',
    source_type='model_field',
    source_config={
        'model': 'core.Plugin',
        'count': True,
        'filter': {'status': 'enabled'}
    },
    is_system=True,
)
```

### 使用标签

**在模板中使用：**

```html
<h1>{{ site.name }}</h1>
<p>{{ site.slogan }}</p>
<footer>{{ site.copyright }}</footer>
<p>当前注册用户：{{ stats.user_count }} 人</p>
<p>已启用插件：{{ stats.plugin_count }} 个</p>
```

**代码中解析标签：**

```python
from core.services.tag_engine import TagEngine

engine = TagEngine()

# 解析单个标签
value = engine.resolve('site.name')       # → "SWEMS"
count = engine.resolve('stats.user_count') # → 128

# 解析模板文本（自动替换所有 {{tag_code}} 标记）
text = "欢迎访问 {{site.name}}，当前用户数：{{stats.user_count}}"
result = engine.resolve_text(text)  # → "欢迎访问 SWEMS，当前用户数：128"
```

### 默认标签

系统初始化后自动创建以下标签：

| 标签编码 | 说明 | 数据源 |
|----------|------|--------|
| `site.name` | 网站名称 | sys_config 表 |
| `site.slogan` | 网站标语 | sys_config 表 |
| `site.copyright` | 版权信息 | sys_config 表 |
| `stats.user_count` | 全站用户数 | User 表计数 |
| `stats.frontend_count` | 前台用户数 | User 表筛选 user_type=frontend |
| `stats.admin_count` | 后台管理员数 | User 表筛选 user_type=admin |
| `stats.plugin_count` | 启用插件数 | Plugin 表筛选 status=enabled |

### 标签测试

在后台标签管理页面，点击标签旁的「测试」按钮，可以即时验证标签能否正确解析。

也可以通过仪表盘的「标签解析测试」功能批量测试：

```python
# 输入
{{site.name}} - {{stats.user_count}}

# 输出
SWEMS - 42
```

---

## 插件开发

### 插件目录结构

每个插件是一个独立的 Python 包，放在项目的 `plugins/` 目录下：

```
plugins/
└── content_manager/
    ├── manifest.json          # 插件清单（必需）
    ├── __init__.py            # 插件入口（必需）
    ├── api.py                 # 插件 API（可选）
    ├── models.py              # 插件数据模型（可选）
    ├── templates/             # 插件模板（可选）
    ├── static/                # 插件静态资源（可选）
    └── migrations/            # 数据库迁移（可选）
```

### manifest.json

插件清单文件，声明插件的基本信息和声明：

```json
{
    "id": "content_manager",
    "name": "content_manager",
    "title": "内容管理",
    "version": "1.0.0",
    "description": "文章和分类管理插件",
    "author": "开发者名称",
    "dependencies": [],
    "requested_permissions": {
        "network": ["api.example.com"],
        "database_tables": ["articles", "categories"],
        "file_read": true,
        "file_write": false,
        "admin_api": false,
        "user_data": false
    },
    "sandbox": {
        "security_level": "moderate",
        "network_whitelist": ["api.example.com"]
    }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | string | 插件唯一标识，与目录名一致 |
| `name` | string | 插件内部名称 |
| `title` | string | 显示名称 |
| `version` | string | 版本号 |
| `description` | string | 插件描述 |
| `author` | string | 作者 |
| `dependencies` | array | 依赖的其他插件 ID |
| `requested_permissions` | object | 申请的权限（安装时需管理员审批） |
| `sandbox` | object | 沙盒配置 |

### __init__.py

插件入口文件，必须提供 `PluginInterface` 的实现：

```python
# plugins/content_manager/__init__.py

from modules.plugin.loader import PluginInterface
from .api import list_articles, create_article


class Plugin(PluginInterface):
    """
    插件主类，必须继承 PluginInterface
    """

    # 插件配置（可选）
    config = {}

    def on_load(self, config: dict = None):
        """
        插件加载时调用
        用于初始化资源、建立连接等
        """
        if config:
            self.config = config
        print('内容管理插件已加载')

    def on_enable(self):
        """
        插件启用时调用
        用于注册信号、启动后台任务等
        """
        print('内容管理插件已启用')

    def on_disable(self):
        """
        插件停用时调用
        用于清理资源、注销信号等
        """
        print('内容管理插件已停用')

    def on_uninstall(self):
        """
        插件卸载时调用
        用于清理数据库、删除临时文件等
        """
        print('内容管理插件已卸载')

    def get_apis(self) -> dict:
        """
        注册插件 API

        返回格式: {
            'api_name': callable_function,
            ...
        }

        其他插件或系统可通过 PluginLoader.call_plugin_api() 调用
        """
        return {
            'list_articles': list_articles,
            'create_article': create_article,
        }
```

### api.py

插件 API 实现文件：

```python
# plugins/content_manager/api.py

import time


def list_articles(category=None, page=1, page_size=20):
    """
    获取文章列表

    参数:
        category: 分类筛选（可选）
        page: 页码
        page_size: 每页数量

    返回:
        dict: {'articles': [...], 'total': int, 'page': int}
    """
    # 实际实现中应查询数据库
    return {
        'articles': [
            {'id': 1, 'title': '示例文章', 'category': 'tech'},
        ],
        'total': 1,
        'page': page,
    }


def create_article(title: str, content: str, category: str = 'default'):
    """
    创建文章

    参数:
        title: 文章标题
        content: 文章内容
        category: 分类

    返回:
        dict: {'id': int, 'title': str, 'created_at': str}
    """
    return {
        'id': int(time.time()),
        'title': title,
        'category': category,
        'created_at': time.strftime('%Y-%m-%d %H:%M:%S'),
    }
```

### 插件 API 调用

**插件之间互相调用：**

```python
from modules.plugin.loader import PluginLoader

# 调用其他插件的 API
result = PluginLoader.call_plugin_api(
    plugin_id='content_manager',
    api_name='list_articles',
    params={'category': 'tech', 'page': 1}
)
print(result['articles'])
```

**通过管理面板测试：**

进入 `/admin-panel/plugins/`，选择插件，使用 API 测试功能。

### 插件事件系统

插件可以在 `on_load`/`on_enable`/`on_disable` 中注册和注销 Django 信号：

```python
from django.db.models.signals import post_save
from core.models import User


class Plugin(PluginInterface):

    def on_enable(self):
        # 订阅信号
        post_save.connect(self._on_user_created, sender=User)

    def on_disable(self):
        # 取消订阅
        post_save.disconnect(self._on_user_created, sender=User)

    def _on_user_created(self, sender, instance, created, **kwargs):
        if created:
            print(f'新用户注册: {instance.username}')

    def get_apis(self):
        return {}
```

### 插件生命周期

```
manifest.json 被发现
    → 安装（install）：写入 Plugin 数据库记录
    → 权限审批（permission_approval）：管理员审核插件申请的权限
    → 启用（enable）：调用 on_load() + on_enable()
    → 运行中（running）：API 可被调用，事件可被接收
    → 停用（disable）：调用 on_disable()
    → 卸载（uninstall）：调用 on_uninstall()，删除数据库记录
```

---

## 插件沙盒

插件沙盒提供运行时隔离和行为监控，防止非官方插件进行恶意操作。

### 安全级别

| 级别 | 网络 | 数据库查询 | 超时 | 内存 |
|------|------|-----------|------|------|
| **strict** | 阻断非白名单 | 最多 50 次 | 10 秒 | 128 MB |
| **moderate** | 记录所有访问 | 最多 200 次 | 30 秒 | 256 MB |
| **permissive** | 仅记录日志 | 最多 1000 次 | 60 秒 | 512 MB |

### 沙盒配置

在插件的 `manifest.json` 中配置：

```json
{
    "sandbox": {
        "security_level": "moderate",
        "network_whitelist": ["api.example.com", "cdn.example.com"],
        "network_blacklist": ["evil.com"],
        "allowed_tables": ["articles", "categories"],
        "max_queries": 100,
        "max_time": 15,
        "max_memory_mb": 200
    }
}
```

也可以在后台 `/admin-panel/sandbox/` 页面为每个插件单独配置。

### 监控内容

| 监控项 | 说明 |
|--------|------|
| 网络访问 | 记录/阻断目标主机和端口 |
| 数据库查询 | 计数查询次数，限制访问的表 |
| 执行时间 | 超时自动终止 |
| 内存使用 | 超限自动终止 |
| 违规事件 | 所有违规写入 OperationLog（log_type=security） |

### 沙盒报告

**后台查看：** `/admin-panel/sandbox/`

**命令行查看：**

```bash
sudo ./swems.sh --sandbox
```

输出示例：

```
沙盒日志总数: 15
违规事件: 2

最近 10 条沙盒事件:
──────────────────────────────────────────
  2025-05-19 10:30:15  [INFO   ]  content_manager      耗时:0.12s  网络:0  SQL:3  违规:0
  2025-05-19 10:29:58  [WARNING]  evil_plugin           耗时:5.00s  网络:12  SQL:0  违规:1

已安装插件沙盒配置:
──────────────────────────────────────────
  content_manager      内容管理        级别:moderate      白名单:api.example.com
  evil_plugin          可疑插件        级别:strict        白名单:无
```

---

## 权限管理

### 默认权限组

系统初始化后自动创建三个默认权限组（不可删除）：

| 权限组 | 范围 | 说明 |
|--------|------|------|
| **超级管理员** | 后台 | 全部模块全部权限 |
| **管理员** | 后台 | 全部模块的「使用」权限，不可修改和删除 |
| **用户** | 前台 | 只能查看日志 |

### 权限粒度

**核心模块（用户/标签/模板/配置/权限/日志）：**

| 权限 | 说明 |
|------|------|
| 使用 | 查看数据、使用功能 |
| 修改 | 编辑数据、更改设置 |
| 删除 | 删除数据 |

**插件模块（动态：安装插件后自动出现）：**

| 权限 | 说明 |
|------|------|
| 执行 | 调用插件 API、使用插件功能 |

### 创建自定义权限组

1. 进入 `/admin-panel/permissions/`
2. 点击「创建权限组」
3. 填写名称和说明
4. 勾选可访问的功能模块
5. 展开每个模块，设置具体权限（使用/修改/删除）

示例：创建「内容编辑」权限组，只允许使用标签管理和模板管理：

- 勾选「标签管理」→ 使用 ✓
- 勾选「模板管理」→ 使用 ✓、修改 ✓
- 其他模块不勾选

### 权限检查

后台中间件自动检查权限。自定义代码中手动检查：

```python
from core.services.permission_service import PermissionService

svc = PermissionService(user)

# 检查用户是否有某个模块的某个权限
if svc.has_permission('tag', 'use'):
    # 允许使用标签
    pass

if svc.has_permission('template', 'modify'):
    # 允许修改模板
    pass
```

### 插件权限审批

插件安装时如果声明了 `requested_permissions`，需要管理员审批：

1. 插件安装后状态为 `disabled`
2. 进入 `/admin-panel/plugin-permissions/`
3. 查看插件申请的权限详情
4. 选择「批准并启用」或「拒绝」

---

## 系统配置

### 配置分组

| 分组 | 键名 | 说明 |
|------|------|------|
| **site** | site_name | 网站名称 |
| | site_slogan | 网站标语 |
| | copyright | 版权信息 |
| | domain | 网站域名 |
| | ip_address | 绑定 IP 地址 |
| | port | 应用端口 |
| | protocol | 协议（http/https） |
| | site_url | 完整站点地址 |
| **database** | db_host | 数据库主机 |
| | db_port | 数据库端口 |
| | db_name | 数据库名 |
| | db_user | 数据库用户 |
| | db_password | 数据库密码（加密） |
| **redis** | redis_host | Redis 主机 |
| | redis_port | Redis 端口 |
| | redis_password | Redis 密码（加密） |
| | redis_db | 数据库编号 |
| **backup** | backup_enabled | 是否启用自动备份 |
| | backup_interval | 备份间隔（秒） |
| | backup_path | 本地备份目录 |
| | backup_remote_url | 远程备份地址 |
| **email** | email_host | SMTP 服务器 |
| | email_port | SMTP 端口 |
| | email_user | SMTP 用户名 |
| | email_password | SMTP 密码（加密） |
| | email_from | 发件人地址 |

### 修改配置

**后台修改：** `/admin-panel/settings/` → 选择分组 → 修改 → 保存

**命令行修改：**

```bash
python manage.py shell -c "
from core.models import SystemConfig
SystemConfig.objects.filter(group='site', key='site_name').update(value='新网站名称')
"
```

### 初始化/重置配置

```bash
python manage.py init_site
```

此命令会创建或更新所有基础配置项、标签和默认权限组。已有的自定义数据不受影响。

---

## 服务管理

SWEMS 提供 `swems.sh` 脚本统一管理所有服务。

### 基本操作

```bash
# 启动所有服务
sudo ./swems.sh --start

# 停止所有服务
sudo ./swems.sh --stop

# 重启所有服务
sudo ./swems.sh --restart

# 查看运行状态
sudo ./swems.sh --status
```

### 日志查看

```bash
# 查看所有日志
sudo ./swems.sh --logs all

# 查看应用日志
sudo ./swems.sh --logs app

# 查看 Celery 日志
sudo ./swems.sh --logs celery

# 查看 Django 日志
sudo ./swems.sh --logs django

# 查看 Nginx 日志
sudo ./swems.sh --logs nginx
```

### 健康检查

```bash
# 单次健康检查
sudo ./swems.sh --health

# 启动健康监控守护进程（每 60 秒检查一次）
sudo ./swems.sh --health-daemon
```

健康检查项：
- Gunicorn 进程状态
- Celery Worker 状态
- Nginx 状态
- MySQL 连接
- Redis 连接
- HTTP 端点响应
- 磁盘空间

异常服务会自动尝试重启。

### Systemd 服务

```bash
# 安装为 systemd 服务
sudo ./swems.sh --install-service

# 使用 systemctl 管理
sudo systemctl start swems
sudo systemctl stop swems
sudo systemctl restart swems
sudo systemctl status swems

# 健康检查定时器
sudo systemctl list-timers swems-health

# 卸载
sudo ./swems.sh --uninstall-service
```

### 完整命令列表

| 命令 | 说明 |
|------|------|
| `--package` | 打包项目 |
| `--deploy` | 部署项目 |
| `--start` | 启动所有服务 |
| `--stop` | 停止所有服务 |
| `--restart` | 重启所有服务 |
| `--status` | 查看运行状态 |
| `--health` | 执行健康检查 |
| `--health-daemon` | 启动健康监控守护进程 |
| `--sandbox` | 查看插件沙盒报告 |
| `--logs [service]` | 查看日志 |
| `--install-service` | 安装 systemd 服务 |
| `--uninstall-service` | 卸载 systemd 服务 |
| `--reset` | 还原至初始状态 |
| `--check` | 环境检查 |

---

## 开发指南

### 数据模型

核心模型位于 `core/models.py`：

| 模型 | 表名 | 说明 |
|------|------|------|
| User | sys_user | 用户（自定义用户模型） |
| PermissionGroup | sys_permission_group | 权限组 |
| Label | sys_label | 标签定义 |
| Plugin | sys_plugin | 插件记录 |
| Template | sys_template | 模板文件 |
| SystemConfig | sys_config | 系统配置 |
| OperationLog | sys_operation_log | 操作日志 |
| LoginLog | sys_login_log | 登录日志 |

### 自定义管理命令

```bash
# 初始化基础数据
python manage.py init_site

# 创建 Django 管理命令
# 在 core/management/commands/ 下创建 .py 文件
```

### 编写管理命令

```python
# core/management/commands/my_command.py

from django.core.management.base import BaseCommand


class Command(BaseCommand):
    help = '命令说明'

    def add_arguments(self, parser):
        parser.add_argument('--name', type=str, help='参数说明')

    def handle(self, *args, **options):
        name = options.get('name')
        self.stdout.write(f'执行命令: {name}')
```

### 添加新的后台页面

1. **在 views.py 中添加视图：**

```python
class AdminMyView(AdminAccessMixin, View):
    def get(self, request):
        return render(request, 'admin/my_page.html', {
            'active_section': 'my_section',
            'data': '...',
        })
```

2. **在 urls.py 中添加路由：**

```python
path('my-page/', views.AdminMyView.as_view(), name='my_page'),
```

3. **在 base.html 侧边栏中添加入口：**

```html
<a href="/admin-panel/my-page/" class="nav-item {% if active_section == 'my_section' %}active{% endif %}">
    <span class="nav-icon">☆</span><span class="nav-label">我的页面</span>
</a>
```

4. **创建模板：**

```html
{% extends "admin/base.html" %}
{% block title %}我的页面{% endblock %}
{% block content %}
<div class="page-header">
    <h1 class="page-title">我的页面</h1>
</div>
<!-- 内容 -->
{% endblock %}
```

### 后台 API

后台管理面板提供 JSON API，用于异步数据获取：

```javascript
// 通用 API 调用函数
const result = await apiCall('/admin-api/permission/group/1/', 'GET');
// result = { ok: true, data: {...} }

const result = await apiCall('/admin-api/tag/resolve/', 'POST', { text: '{{site.name}}' });
// result = { ok: true, data: { resolved: 'SWEMS' } }
```

---

## 常见问题

### Q: 忘记管理员密码怎么办？

```bash
python manage.py shell -c "
from core.models import User
u = User.objects.filter(is_superuser=True).first()
u.set_password('new_password')
u.save()
print('密码已重置')
"
```

### Q: 如何添加新的系统配置项？

编辑 `core/management/commands/init_site.py`，在 `_create_configs` 方法中添加配置项，然后运行：

```bash
python manage.py init_site
```

### Q: 插件如何访问数据库？

插件在沙盒中运行，可直接使用 Django ORM。通过 `requested_permissions.database_tables` 声明需要访问的表，由沙盒进行限制。

```python
from django.db import connection

# 在插件 API 中执行查询
with connection.cursor() as cursor:
    cursor.execute("SELECT * FROM my_table")
    rows = cursor.fetchall()
```

### Q: 如何备份和恢复？

```bash
# 备份数据库
mysqldump -u root edu_platform > backup_$(date +%Y%m%d).sql

# 恢复数据库
mysql -u root edu_platform < backup_20250101.sql
```

自动备份可在系统配置的 `backup` 分组中设置。

### Q: 日志太多怎么清理？

```bash
# 命令行清理 30 天前的日志
python manage.py shell -c "
from core.models import OperationLog
from django.utils import timezone
import datetime
cutoff = timezone.now() - datetime.timedelta(days=30)
deleted = OperationLog.objects.filter(created_at__lt=cutoff).count()
OperationLog.objects.filter(created_at__lt=cutoff).delete()
print(f'清理日志: {deleted} 条')
"
```

### Q: 如何自定义后台样式？

后台模板位于 `templates/admin/`，静态资源位于 `static/admin/`。所有 CSS 变量定义在 base.html 的 `:root` 中，可通过修改变量自定义主题。

---

### Q：如何使用SWEMS？
```text
# === 开发机器 ===
./swems.sh --package          # 打包成 dist/swems_xxx.tar.gz
# 把 tar.gz 传到目标服务器

# === 目标服务器 ===
./swems.sh --check            # 检查环境够不够
sudo ./swems.sh --deploy      # 部署（装依赖、建库、配置 Nginx/Supervisor）
sudo ./swems.sh --start       # 启动服务
sudo ./swems.sh --install-service  # 设为开机自启

# === 日常运维 ===
sudo ./swems.sh --status      # 看状态
sudo ./swems.sh --health      # 健康检查
sudo ./swems.sh --logs all    # 看日志
sudo ./swems.sh --restart     # 重启

```

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v0.1.0 Beta | 2026-05 | 初始版本。标签系统、插件系统、权限管理、沙盒监控、后台管理面板、服务管理脚本 |

---

## 许可证

MIT License

Copyright (c) 2026 SWEMS
