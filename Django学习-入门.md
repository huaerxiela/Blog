---
title: Django学习-入门
date: 2019-12-13 21:12:48
tags: Django
category: python
---
<!-- TOC -->

- [1. 虚拟环境](#1-虚拟环境)
  - [1.1. 创建](#11-创建)
  - [1.2. 退出](#12-退出)
  - [1.3. 查看和使用](#13-查看和使用)
  - [1.4. 删除](#14-删除)
  - [1.5. 包操作](#15-包操作)
  - [1.6. 安装django包](#16-安装django包)
- [2. 创建项目](#2-创建项目)
  - [2.1. 创建应用](#21-创建应用)
  - [2.2. 安装应用](#22-安装应用)
  - [2.3. 开发服务器](#23-开发服务器)
- [3. 设计模型](#3-设计模型)
  - [3.1. ORM框架](#31-orm框架)
  - [3.2. 定义模型类](#32-定义模型类)
    - [3.2.1. 设计图书类](#321-设计图书类)
    - [3.2.2. 模型类设计](#322-模型类设计)
  - [3.3. 迁移](#33-迁移)
    - [3.3.1. 设计英雄类](#331-设计英雄类)
  - [3.4. 数据操作](#34-数据操作)
    - [3.4.1. 对象关联操作](#341-对象关联操作)
- [4. 后台管理](#4-后台管理)
  - [4.1. 管理页面本地化](#41-管理页面本地化)
  - [4.2. 创建管理员](#42-创建管理员)
  - [4.3. 注册模型类](#43-注册模型类)
  - [4.4. 自定义管理页面](#44-自定义管理页面)
- [5. 视图以及URL](#5-视图以及url)
  - [5.1. 定义视图](#51-定义视图)
  - [5.2. 配置URLconf](#52-配置urlconf)
    - [5.2.1. 查找视图的过程](#521-查找视图的过程)
  - [5.3. 请求访问](#53-请求访问)
- [6. 模板](#6-模板)
  - [6.1. 创建模板](#61-创建模板)
  - [6.2. 自定义模板](#62-自定义模板)
  - [6.3. 视图调用模板](#63-视图调用模板)
  - [6.4. 视图调用模板简写](#64-视图调用模板简写)

<!-- /TOC -->
# 1. 虚拟环境
虚拟环境其实就是对真实python环境的复制，这样我们在复制的python环境中安装包就不会影响到真实的python环境，在不同的虚拟环境中开发项目就实现了项目之间的隔离。

## 1.1. 创建
首先安装虚拟环境，命令如下:
```
sudo pip3 install virtualenv
```
接下来还要安装虚拟环境扩展包，命令如下：
```
sudo pip3 install virtualenvwrapper
```
安装虚拟环境包装器的目的是使用更加简单的命令来管理虚拟环境。

修改home目录下的配置文件 `.bashrc` ，添加如下内容：
```
export WORKEON_HOME=$HOME/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh
```
这里运行可能会出现 `/usr/bin/python: No module named virtualenvwrapper` 错误，修改 `/usr/local/bin/virtualenvwrapper.sh` 如下即可：
![](Django学习-入门/2019-12-13-21-35-50.png)

使用 `source .bashrc` 命令使配置文件生效。

创建python3虚拟环境命令如下：
```
mkvirtualenv -p python env_name
```
例如：
![](Django学习-入门/2019-12-13-21-40-39.png)

可以看到已经创建了一个虚拟环境`/home/ckcat/.virtualenvs/django_study`，并且自动进入了该虚拟环境。

## 1.2. 退出
退出虚拟环境的命令如下：
```
deactivate
```
![](Django学习-入门/2019-12-13-21-45-12.png)

## 1.3. 查看和使用
查看所有虚拟环境命令如下：
```
workon 两次tab键
```
使用命令如下：
```
workon env_name
```
![](Django学习-入门/2019-12-13-21-48-46.png)

## 1.4. 删除
删除虚拟环境使用如下命令：
```
rmvirtualenv env_name
```
但是删除前，必须先退出虚拟环境，例如：
![](Django学习-入门/2019-12-13-21-51-42.png)

## 1.5. 包操作
在虚拟环境中使用pip命令操作python包，安装命令如下：
```
pip install packegename
```

查看以安装的python包命令如下：
```
pip list 
pip freeze
```

## 1.6. 安装django包
本次学习中使用的是 `django1.8.2` 的版本，安装命令如下
```
pip install django==1.8.2
```
![](Django学习-入门/2019-12-13-21-58-25.png)

至此，整个虚拟环境搭建完成，下面开始创建项目。


# 2. 创建项目
首先选定好项目的存放目录，我这里存放在 `/home/ckcat/code/PythonProjects/Django_Study` ，使用如下命令创建项目：
```
django-admin startproject project_name
```
例如：
![](Django学习-入门/2019-12-13-22-04-29.png)

目录结构：
* `manage.py` 是项目管理文件，通过他管理项目。
* `test01` 与项目同名的目录。
* `_init_.py` 是一个空文件，作用是这个目录`test01`可以被当作包使用。
* `settings.py` 是项目的整体配置文件。
* `urls.py` 是项目的URL配置文件。
* `wsgi.py` 是项目与WSGI兼容的WEB服务器入口。

接下来可以使用pycharm打开该项目。

## 2.1. 创建应用
使用一个应用开发一个业务模块，此处创建应用名称为booktest,创建应用命令如下：
```
python manage.py startapp booktest
```
![](Django学习-入门/2019-12-13-22-15-11.png)

目录结构：
* `_init_.py` 是一个空文件，作用是这个目录`booktest`可以被当作包使用。
* `tests.py` 用于开发测试用例。
* `models.py` 与数据库操作相关。
* `views.py` 与接收浏览器请求，进行处理，返回页面相关。
* `admin.py` 与网站后台管理相关。


## 2.2. 安装应用
应用创建完成后需要安装才可以使用，在`settings.py` 中 `INSTALLED_APPS` 下添加应用名称就可以完成安装。

![](Django学习-入门/2019-12-13-22-23-28.png)

## 2.3. 开发服务器
在开发阶段，为了能快速预览到开发的效果，django提供了一个纯python编写的轻量级web服务器，仅在开发阶段使用，运行命令如下：
```
python manage.py runserver ip:port
```
> ip和端口默认为 127.0.0.1:8000

例如：
![](Django学习-入门/2019-12-13-22-29-24.png)

接着在浏览器中输入网址 `127.0.0.1:8000` 可以查看当前站点的开发效果，如下图所示：
![](Django学习-入门/2019-12-13-22-31-12.png)


# 3. 设计模型

## 3.1. ORM框架
O是object，也就是类对象的意思，R是relation，也就是关系数据库中数据表的意思，M是Mapping，是映射的意思。在ORM框架中，它帮助我们把类和数据表进行了一个映射，可以让我们通过类和类对象就能操作它所对应的表格中的数据。ORM框架还有一个功能，它可以根据我们设计的类自动帮我们生成数据库中的表格，省去了我们自己建表的过程。

django中内嵌了ORM框架，不需要直接面向数据库编程，而是定义模型类，通过模型类和对象完成数据表的增删改查操作。

使用django进行数据库开发的步骤如下：
1. 在models.py中定义模型类。
2. 迁移。
3. 通过类和对象完成数据增删改查。

## 3.2. 定义模型类
模型类定义在 models.py 文件中，继承自 models.Model 类。
> 说明：不需要定义主键列，在生成时会自动添加，并且值自动增长。

### 3.2.1. 设计图书类
读书类：
* 类名 : BookInfo
* 读书名称 : btitle
* 读书发布日期 : bpub_date

### 3.2.2. 模型类设计
根据设计，在 models.py 中定义模型类如下：
```
from django.db import models

# Create your models here.
class BookInfo(models.Model):
    btitle = models.CharField(max_length=20)
    bpub_date = models.DateField()
```

## 3.3. 迁移
迁移前目录结构如下图：
![](Django学习-入门/2019-12-14-15-03-30.png)

迁移由两步完成：
1. 生成迁移文件：根据模型类生成创建表的迁移文件。
2. 执行迁移：根据第一步生成的迁移文件在数据库中创建表。

生成迁移文件命令如下：
```
python manage.py makemigrations
```
![](Django学习-入门/2019-12-14-15-07-55.png)

执行生成迁移文件命令后，会在应用 booktest 目录下的 migrations 目录中生成迁移文件，内容如下：
![](Django学习-入门/2019-12-14-15-11-16.png)

django 框架根据我们设计的模型类生成了迁移文件，在迁移文件中我们可以看到 fields 列表中每一个元素跟 BookInfo 类属性名以及属性的类型是一致的。同时我们发现一个 id 项，这一项是 django 框架帮我们自动生成的，在创建表的时候 id 就会作为对应的主键列，并且主键列自动增长。

执行迁移命令如下：
```
python manage.py migrate
```
![](Django学习-入门/2019-12-14-15-16-36.png)

执行迁移命令后，将会生成对应的 sqlite3 数据库文件，数据表默认名称为：
```
<app_name>_<model_name>
例如：
booktest_bookinfo
```

### 3.3.1. 设计英雄类
英雄类：
* 类名 : HeroInfo
* 英雄名 : hname
* 英雄性别 : hgender
* 英雄简介 : hcomment
* 英雄所属读书 : hbook
* 读书-英雄的关系为一对多

打开 booktest/models.py , 定义英雄类代码如下：
```
class HeroInfo(models.Model):
    hname = models.CharField(max_length=20)
    hgender = models.BooleanField()
    hcomment = models.CharField(max_length=100)
    hbook = models.ForeignKey("BookInfo") # BookInfo 作为 HeroInfo 的外键。
```
生成迁移文件，并执行迁移：
```
python manage.py makemigrations
python manage.py migrate
```
![](Django学习-入门/2019-12-14-15-32-21.png)

之后将会生成 booktest_HeroInfo 表，并且 hbook 字段将会变成 hbook_id 对应 booktest_BookInfo 中的主键。

## 3.4. 数据操作
完成数据表的迁移之后，就可以通过进入项目的 shell 进行简单的 API 操作。如果需要退出可以使用 `ctrl+d` 或者输入 `quit()` .

进入项目 shell :
```
pyhotn manage.py shell
```
![](Django学习-入门/2019-12-14-15-38-59.png)

首先引入 booktest/models 中的类：
```
from booktest.models import BookInfo, HeroInfo
```
查询所有图书信息
```
BookInfo.objects.all()
```
新建图书对象，并查看所有图书信息：
```
b = BookInfo()
b.btitle = "射雕英雄传"
from datetime import date
b.bpub_date=date(1990,1,23)
b.save()
BookInfo.objects.all()
```
![](Django学习-入门/2019-12-14-15-44-42.png)

查找图书信息并查看值：
```
>>> b=BookInfo.objects.get(id=1)
>>> b
<BookInfo: BookInfo object>
>>> b.id
1
>>> b.btitle
'射雕英雄传'
>>> b.bpub_date
datetime.date(1990, 1, 23)
```
![](Django学习-入门/2019-12-14-15-47-06.png)

修改图书信息：
```
>>> b.bpub_date=date(2019,12,14)
>>> b.save
<bound method Model.save of <BookInfo: BookInfo object>>
>>> b.bpub_date
datetime.date(2019, 12, 14)
```
![](Django学习-入门/2019-12-14-15-49-14.png)

删除读书信息：
```
b.delete()
```

### 3.4.1. 对象关联操作
创建一个 HeroInfo 对象：
```
>>> h=HeroInfo()
>>> h.hname="黄蓉"
>>> h.hgender=False
>>> h.hcomment="她是女主角"
>>> h.hbook=b
>>> h.save()
```
图书与英雄是一对多的关系，django中提供了关联的操作方式。
获得关联集合，返回当前 book 对象的所有 hero 。
```
>>> b.heroinfo_set.all()
[<HeroInfo: HeroInfo object>]
```
![](Django学习-入门/2019-12-14-15-57-11.png)

# 4. 后台管理
使用 django 后台管理需要安装如下步骤操作：
1. 管理页面本地化
2. 管理管理员
3. 注册模型类
4. 自定义管理页面

## 4.1. 管理页面本地化
本地化是将显示的语言、时间等使用本地的习惯。
打开 `test01/settings.py` 文件，找到语言编码、时区的设置，将内容改为如下
```
LANGUAGE_CODE = 'zh-hans' #'en-us'使用中文
TIME_ZONE = 'Asia/Shanghai' #'UTC' 设置上海时区
```

## 4.2. 创建管理员
创建管理员命令如下，按提示输入用户名，邮箱，密码。
```
python manage.py createsuperuser 
```
![](Django学习-入门/2019-12-14-16-07-08.png)

接下来启动服务器。
```
python manage.py runserver
```
在浏览器中打开如下地址
```
http://localhost:8000/admin/
```
![](Django学习-入门/2019-12-14-16-10-01.png)

输入刚才创建的用户名和密码，进入管理页面。
![](Django学习-入门/2019-12-14-16-11-29.png)

## 4.3. 注册模型类
登陆后台管理后，默认没有我们创建的应用中定义的模型类，需要在自己应用中的 admin.py 文件中注册，才可以在后台管理中看到，并进行增删查找操作。

打开 `booktest/admin.py` 文件，编写如下代码：
```
from django.contrib import admin

# Register your models here.
from booktest.models import BookInfo, HeroInfo

admin.site.register(BookInfo)
admin.site.register(HeroInfo)
```
刷新浏览器页面，可以看到模型类 BookInfo 和 HeroInfo 的管理了。
![](Django学习-入门/2019-12-14-16-18-42.png)

## 4.4. 自定义管理页面
在列表中只显示了 BookInfo object ，对象的其他属性并没有列出来，查看非常不方便。Django 提供了自定义管理页面的功能。

打开 `booktest/admin.py` 文件，自定义类，继承自 `admin.ModelAdmin` 类。
* 属性 list_display 表示要显示哪些属性
```
class BookInfoAdmin(admin.ModelAdmin):
    list_display = ['id', 'btitle', 'bpub_date']
```
* 修改该模型类 BookInfo 的注册代码如下
```
admin.site.register(BookInfo, BookInfoAdmin)
```
* 刷新BookInfo的列表也，所有的属性都显示出来了

![](Django学习-入门/2019-12-14-16-30-04.png)

最终 booktest/admin.py 文件代码如下:
```
from django.contrib import admin

# Register your models here.
from booktest.models import BookInfo, HeroInfo

class BookInfoAdmin(admin.ModelAdmin):
    list_display = ['id', 'btitle', 'bpub_date']

class HeroInfoAdmin(admin.ModelAdmin):
    list_display = ['id', 'hname', 'hgender', 'hcomment']

admin.site.register(BookInfo, BookInfoAdmin)
admin.site.register(HeroInfo, HeroInfoAdmin)
```

# 5. 视图以及URL
django 的设计框架是 MVT ，用户在 URL 中请求的是视图，视图接收请求后进行处理，并将处理结果返回给请求者。
使用视图时需要进行两步操作：
1. 定义视图
2. 配置URLconf

## 5.1. 定义视图
视图就是一个 python 函数， 被定义在 `views.py` 中。

视图必须有一个参数，一般叫 `request` ,视图必须返回 `HttpResponse` 对象, `HttpResponse` 中的参数内容会显示在浏览器页面上。

打开 `booktest/views.py` 文件，定义视图 `index` 如下:
```
from django.shortcuts import render

# Create your views here.
from django.http import HttpResponse

def index(request):
    return HttpResponse("index")
```

## 5.2. 配置URLconf
### 5.2.1. 查找视图的过程
请求者在浏览器中输入 url ，请求到网站后，获取 url 信息，然后编写好的 URLconf 逐条匹配，如果匹配成功则调用对应的视图函数，如果所有的 URLconf 都没有匹配成功，则返回404错误。

一条 URLconf 包括 url 规则、视图两部分：
* url 规则使用正则表达式定义。
* 视图就是在 `views.py` 中定义的视图函数。

需要两步完成 URLconf 配置：
1. 在应用中定义 URLconf 。
2. 包含到项目中的 URLconf 中。

在 booktest 应用下创建 `urls.py` 文件，定义代码如下：
```
from django.conf.urls import url
from booktest import views

urlpatterns = [
    url(r'^$', views.index),
]
```
包含到项目 `test01/urls.py` 文件中, 完整代码如下
```
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^', include('booktest.urls'))
]
```

## 5.3. 请求访问
视图和 URLconf 都定义好后，在流浪其中打开 `localhost:8000` 页面。
![](Django学习-入门/2019-12-14-17-07-21.png)

# 6. 模板
在 django 中，将前端的内容定义在模板中，然后再把模板交给视图调用，各种漂亮、炫酷的效果就出现了。

## 6.1. 创建模板
为 booktet 的视图 index 创建模板 `index.html` ,目录结构如下图：
![](Django学习-入门/2019-12-14-17-13-06.png)

一般将定义 `templates` 目录在项目的根目录，其中根据不同的应用创建不同的目录。

设置查找模板的路径： 打开 `test01/setting.py` 文件，设置 TEMPLATES 的 DIRS 值：
```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')], # 设置模板路径
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

## 6.2. 自定义模板
打开 `templates/booktest/index.html`，定义代码如下：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>读书列表</title>
</head>
<body>
<!-- title 为变量名-->
<h1>{{ title }}</h1>
{% for i in list %}
{{ i }}<br>
{% endfor %}

</body>
</html>
```

## 6.3. 视图调用模板
打开 `booktest/views.py` 文件，调用上面的模板文件
```
from django.shortcuts import render

# Create your views here.
from django.http import HttpResponse
from django.template import loader, RequestContext
def index(request):
    # 1. 获取模板
    template = loader.get_template('booktest/index.html')
    # 2. 定义上下文
    context = RequestContext(request, {'title':'图书列表', 'list':range(10)})
    # 3. 渲染模板
    return HttpResponse(template.render(context))
```
对应的页面如下图：

![](Django学习-入门/2019-12-14-17-27-26.png)

## 6.4. 视图调用模板简写
视图调用模板都要执行以上三部分，于是 django 提供了一个函数 render 封装了以上代码。

方法 render 包含3个参数：
* 第一个参数为 request 对象
* 第二个参数为模板文件路径
* 第三个参数为字典，表示向模板中传递的上下文数据

打开 `booktest/views.py` 文件，调用 render 的代码如下：
```
def index(request):
    context = {'title':'图书列表', 'list':range(10)}
    return render(request, 'booktest/index.html', context)
```

> 对应的[代码地址](https://github.com/CKCat/Note/tree/master/Django/test01)







