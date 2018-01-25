**从头开始django使用**

`pip install django`  #安装django包

`python -m django --version`  #查看django版本

`django-admin startproject mysite`  #创建一个项目

项目目录结构

```
mysite/
    manage.py     命令行工具
    mysite/           项目包
        __init__.py   把目录变成包
        settings.py  配置文件
        urls.py        路由文件
        wsgi.py      wsgi兼容的web服务器入口
```

`python manage.py runserver [8080]`  #启动服务器, 可指定端口, 默认8000

`python mange.py startapp demo` #创建demo app, 将会在mysite根目录下创建demo目录

demo app目录结构

```
demo/
    __init__.py
    admin.py       应用管理
    apps.py         应用配置文件
    migrations/   数据库迁移
        __init__.py
    models.py  模型文件
    tests.py   测试文件
    views.py  视图文件
```

编辑`demo/views.py`, 创建视图

```
from django.http import HttpResponse

def index(request):
    return HttpResponse('this is index view in demo app')
```

创建文件`demo/urls.py`, 编写一个URLConf

```
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

将`demo/urls.py`注册到根URLConf, 也就是`mysite/urls.py`

```
from django.urls import include, path
from django.contrib import admin

urlpatterns = [
    path('demo/', include('demo.urls')),
    path('admin/', admin.site.urls),
]
```

关于`djaon.urls.path`方法, 其原型`path(route, view, **kwargs, name='')`

`route`参数是一个包含URL模式的字符串, 必选;

`view`参数用于指定调用的视图函数, 必选;

`kwargs`参数可以在字典中传递任意关键字参数给目标视图, 可选;

`name`参数可以全局命名URL, 可选;


**数据库设置**

`mysite/settings.py`里默认用的sqlite3数据库, 如果要使用mysql/postgresql/oracle, 如下

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',  # 可选postgresql/mysql/sqlite3/oracle
        'NAME': 'dbname',  # 如果是sqlite3, 则要指定路径如 os.path.join(BASE_DIR, 'db.sqlite3')
        'USER': 'dbuser',    # sqlite3无需配置
        'PASSWORD': 'dbpasswd',  # sqlite3无需配置
        'HOST': '10.1.10.1',   # sqlite3无需配置
        'PORT': '3306',  # sqlite3无需配置
    }
}
```

`demo/models`中创建模型

```
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

编辑`mysite/settings`, 在`INSTALLED_APPS`配置列表项中添加`demo/apps/DemoConfig,`

`python mange.py makemigrations demo`  #为demo app里的更改创建迁移

`python manage.py sqlmigrate demo 0001` #查看迁移任务0001的相应sql语句

`python manage.py check`  #检查项目健康情况

`python manage.py migrate`  #将迁移应用到数据库, 会将INSTALLED_APPS里的app相关数据库数据迁移

`python manage.py shell` #交互式环境

```
from polls.models import Question, Choice
Question.objects.all()
from django.utils import timezone
q = Question(question_text="What's new?", pub_date=timezone.now())
q.save()
q.id
q.question_text
q.pub_date
q.question_text = 'which is the huggest?'
q.save()

Question.objects.filter(id=1)
Question.objects.filter(question_text__startswith='What')
Question.objects.get(id=1)
q = Question.objects.get(pk=1)  #primary key = 1
q.choice_set.all()  #所有与q关联的choice

q.choice_set.create(choice_text='The land', votes=0)
q.choice_set.create(choice_text='The sky', votes=0)
c = q.choice_set.create(choice_text='The sea', votes=0)
c.question
q.choice_set.all()
q.choice_set.count()
current_year = timezone.now().year
c = Choice.objects.filter(question__pub_date__year=current_year)
c.delete()
```

**关于 django admin**

`python manage.py crteatesuperuser`   #创建管理员帐号, 回车输入帐号邮箱和密码

编辑`demo/admin.py`, 将demo app注册到admin app

```
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```

`python mange.py test demo`  #运行测试用例

[django 2.0.* docs](http://python.usyiyi.cn/translate/django2/index.html)