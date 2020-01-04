---
title: Django 中使用 celery 完成异步操作
date: 2017-06-20 20:37:20
tags: [Python, Django]
---

Deploy a web application by django with celery.

<!-- more -->

## 目的

写一个 django 的服务，执行网络 IO 操作时使用异步框架 Celery。

## 搭建 Django 环境

+ 创建虚拟环境

``` bash
➜  mkvirtualenv django-with-celery
Using base prefix '/usr/local/Cellar/python3/3.6.1/Frameworks/Python.framework/Versions/3.6'
New python executable in /Users/lvhuiyang/.virtualenvs/test/bin/python3.6
Also creating executable in /Users/lvhuiyang/.virtualenvs/test/bin/python
Installing setuptools, pip, wheel...done.
virtualenvwrapper.user_scripts creating /Users/lvhuiyang/.virtualenvs/test/bin/predeactivate
virtualenvwrapper.user_scripts creating /Users/lvhuiyang/.virtualenvs/test/bin/postdeactivate
virtualenvwrapper.user_scripts creating /Users/lvhuiyang/.virtualenvs/test/bin/preactivate
virtualenvwrapper.user_scripts creating /Users/lvhuiyang/.virtualenvs/test/bin/postactivate
virtualenvwrapper.user_scripts creating /Users/lvhuiyang/.virtualenvs/test/bin/get_env_details
(django-with-celery) ➜
```

+ 使用 pip 安装 Django

``` bash
(django-with-celery) ➜ pip install Django
```

+ 使用 django-admin 创建项目，当前目录文件如下

``` bash
(django-with-celery) ➜  my_project tree
.
├── manage.py
├── my_app
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
└── my_project
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

+ 注意

使用celery 4.0 之后的版本请使用原生 celery， djcelery 当前版本应该是不支持celery 4.0之后版本的。

## 配置

+ 安装一个 BROKER 环境供 celery 运行，当前使用的是 redis(rabbitmq 同理)，为了方便使用 docker部署了一个 redis 环境。

``` bash
(django-with-celery) ➜  my_project docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                  PORTS               NAMES
83d34d65e6c1        redis               "docker-entrypoint..."   2 days ago          Exited (0) 2 days ago                       my-redis3
b3ed35793dcc        postgres            "docker-entrypoint..."   2 days ago          Exited (0) 2 days ago                       my-postgres
(django-with-celery) ➜  my_project docker start 83d34d65e6c1
```

+ 查看当前容器（容器6379的默认端口映射到主机6380端口了）

``` bash
➜  ~ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
83d34d65e6c1        redis               "docker-entrypoint..."   2 days ago          Up 8 minutes        0.0.0.0:6380->6379/tcp   my-redis3
➜  ~
```

+ 在 my_project/my_project/settings.py 文件中添加 BROKER 的 url

``` python
import os

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

SECRET_KEY = 'c2)2tc5b%es5*d1ar1%&me_c8ngxez$ey%k-@)1%-$()q*0m+k'

DEBUG = True

ALLOWED_HOSTS = []

CELERY_BROKER_URL = 'redis://localhost:6380/0'    # 新加的BROKER的url

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

+ 创建 my_project/my_project/celery.py 文件，添加celery基本的配置信息

``` python
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'my_project.settings')
app = Celery('demo')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

```

## 运行

+ 在 application 中添加celery任务： 新建 my_project/my_app/tasks.py 文件，写一个简单的函数

``` python
import time
from my_project.celery import app


@app.task
def add(x, y):
    time.sleep(3)
    print("等待了3秒然后打印一下")
    return x + y

```

+ 写两个简单的路由分别 同步/异步 处理

``` python
from django.shortcuts import HttpResponse
from my_app.tasks import add


def index(request):
    add(100, 200) # 常规调用了 add()， 同步处理
    return HttpResponse("HelloWorld")


def hello(request):
    add.delay(100, 200)  # 表明 delay ，使用celery异步处理
    return HttpResponse("HelloWorld")

```

并加在项目 url 规则表里

``` python
from django.conf.urls import url
from django.contrib import admin
from my_app.views import index, hello

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^$', index),      # 新加的同步处理路由
    url(r'^hello/', hello), # 新加的异步处理路由
]
```

+ `celery -A my_project worker -l info`  启动celery服务，具体命令以及含义请查阅官方材料

+ `./manage.py runserver` 运行 django， 访问 `127.0.0.1：8000/` 显示结果（常规的同步处理）

``` bash
等待了3秒然后打印一下
[20/Jul/2017 15:14:22] "GET / HTTP/1.1" 200 10
```

访问 `127.0.0.1：8000/hello` 会立即返回 HelloWorld, Web 终端中显示

``` bash
[20/Jul/2017 15:14:22] "GET / HTTP/1.1" 200 10
```

celery 终端的显示:

``` bash
[2017-07-20 14:48:46,240: INFO/MainProcess] celery@lvhuiyangdeMBP.lan ready.
[2017-07-20 14:49:00,407: INFO/MainProcess] Received task: my_app.tasks.add[b5648ac5-60bb-4961-aef0-1d4f4dff8e4c]
[2017-07-20 14:49:02,304: INFO/MainProcess] Received task: my_app.tasks.add[87eb0ef0-30ce-4103-b450-b96da2bbfa30]
[2017-07-20 14:49:03,412: WARNING/ForkPoolWorker-2] 等待了3秒然后打印一下
```

## 写在最后

+ 注意：在路由函数中把异步运算的结果返回的话是非常愚蠢的
+ 仅仅是把 celery 搭建起来了， 还需要一些进一步的事情没有搞。
+ 请查阅 [官方文档](http://docs.celeryproject.org/en/latest/)
