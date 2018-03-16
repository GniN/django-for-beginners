# Dajango 入門教學
這次教學將簡單介紹Django 以及建立一個簡單的 Django專案

## 行前準備
- 已安裝python
- 可以使用pip

![add python to path](https://i.imgur.com/8OpWOv9m.jpg)


## 安裝Django

安裝django 非常簡單，只需使用以下指令
```
pip install django==1.11.11
```

1.11.11是django目前LTS的最新版。

```
Python 3.6.4 (v3.6.4:d48eceb, Dec 19 2017, 06:04:45) [MSC v.1900 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import django
>>> django.get_version()
'1.11.11'
>>>
```

如果可以成功import django module代表已經安裝成功，透過 _get_version()_ 可以查看安裝的版本。

## 建立Django 專案

Django的專案可以透過一行指令就產生
```
django-admin startproject mysite 
```
跑完指令後會出現以下的目錄
```
mysite/
  manage.py 
  mysite/
    __init__.py 
    settings.py 
    urls.py 
    wsgi.py 
```
其中
- manage.py 是一個用來跟django 專案互動的 command-line utility。可以在[官網](https://docs.djangoproject.com/en/1.11/ref/django-admin/)看到更多的指令。
- settings.py 專案的設定資訊


## setting up database

透過這個指令
```
python manage.py migrate
```
資料庫就會建立到 settings.py 所設定的位置。

## 運行dev server

透過這個指令可以運行django的dev server
```
python manage.py runserver 
```
很酷的一點是，每當你的程式有變更，不需要重新執行，dev server會自動的應用變更。

## 第一個 view 
在 \mysite\mysite\ 建立一個 view.py

```
from django.http import HttpResponse

def hello(request):
    return HttpResponse("Hello world") 
```

## 第一個 URLconf
在 url.py

```python
from django.conf.urls import url
from django.contrib import admin

from mysite.views import hello  #<--- add

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^hello/$', hello),  #<--- add
]
```

## 動態內容
我們來為我們頁面加一點動態內容，顯示現在的時間

```python
from django.http import HttpResponse
import datetime

def hello(request):
    return HttpResponse("Hello world")

def current_datetime(request):
    now = datetime.datetime.now()
    html = "<html><body>It is now %s.</body></html>" % now        
    return HttpResponse(html)
```


```python
from django.conf.urls import url
from django.contrib import admin

from mysite.views import hello, current_datetime

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^hello/$', hello),
    url(r'^time/$', current_datetime),
]
```

## 動態url

如果我們希望在url 後面加上參數，讓顯示的時間可以是現在時間加上幾小時。
網址看起來可能是這樣。

```python
urlpatterns = [
    url(r'^time/$', current_datetime),
    url(r'^time/plus/1/$', one_hour_ahead),
    url(r'^time/plus/2/$', two_hours_ahead),
    url(r'^time/plus/3/$', three_hours_ahead),
]
```

但是把所有情況都定義出來是很蠢的一件事情，我們來看看django如何處理動態url

```python
urlpatterns = [
    # ...
    url(r'^time/plus/(\d+)/$', hours_ahead),
    # ...
]
```

```python
def hours_ahead(request, offset):
    try:
        offset = int(offset)
    except ValueError:
        raise Http404()
    dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
    html = "<html><body>In %s hour(s), it will be  %s.</body></html>" % (offset, dt)
    return HttpResponse(html)
```

## json results
```python
def json_result(request):
    r = {
        'name': 'Andy',
        'age': 28
    }
    return JsonResponse(r)
```
```python
urlpatterns = [
    # ...
    url(r'^json/$', json_result),
    # ...
]
```
## Model 和資料庫

### 資料庫設定

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

### 第一個app

在有 _manage.py_ 的資料夾下:
```
python manage.py startapp books 
```
會產生一個books的app

### 建立model

```python
from django.db import models

class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

class Author(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=40)
    email = models.EmailField()

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField()
```

```python
INSTALLED_APPS = [
'books.apps.BooksConfig',  #<---- 加入到installed apps
'django.contrib.admin',
'django.contrib.auth',
'django.contrib.contenttypes',
'django.contrib.sessions',
'django.contrib.messages',
'django.contrib.staticfiles',
]
```

產生migration紀錄
```
python manage.py makemigrations books 
```

查看這次變更的sql
```
python manage.py sqlmigrate books 0001
```

對資料庫進行變更
```
python manage.py migrate 
```

## 來玩玩看資料庫吧

可以用以下指令進入 django shell
```
python manage.py shell
```

```python
>>> from books.models import Publisher
>>> p1 = Publisher(name='Apress', address='2855 Telegraph Avenue', city='Berkeley', state_province='CA', country='U.S.A.', website='http://www.apress.com/')
>>> p1.save()
>>> p2 = Publisher(name="O'Reilly", address='10 Fawcett St.', city='Cambridge', state_province='MA', country='U.S.A.',website='http://www.oreilly.com/')
>>> p2.save()
>>> publisher_list = Publisher.objects.all()
>>> publisher_list
<QuerySet [<Publisher: Publisher object>, <Publisher: Publisher object>]>py
```

但我們會發現這樣的result並不容易讀
```
<QuerySet [<Publisher: Publisher object>, <Publisher: Publisher object>]>
```

我們可以對models稍作修改

```python
from django.db import models

class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

    def __str__(self):
        return self.name

class Author(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=40)
    email = models.EmailField()

    def __str__(self):
        return u'%s %s' % (self.first_name, self.last_name)

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField()

    def __str__(self):
        return self.title
```

讓我們再試一次看看吧

```python
>>> from books.models import Publisher
>>> publisher_list = Publisher.objects.all()
>>> publisher_list
<QuerySet [<Publisher: Apress>, <Publisher: O'Reilly>]>
```

update record

```python
>>> p = publisher_list[0]
>>> p.name
'Apress'
>>> p.name = "apress2"
>>> p.save()
>>> publisher_list = Publisher.objects.all()
>>> publisher_list
<QuerySet [<Publisher: apress2>]>
```

filtering data

```python
Publisher.objects.filter(name="O'Reilly")
Publisher.objects.filter(name="O'Reilly", address='10 Fawcett St.2')
Publisher.objects.filter(name__contains="press")
```

get single object
```python
Publisher.objects.get(name="O'Reilly")
```

ordering
```python
Publisher.objects.order_by("name")
Publisher.objects.order_by("-name")
```

Chaining Lookups
```
Publisher.objects.filter(country="U.S.A.").order_by("-name")
```

limit
```python
Publisher.objects.order_by('name')[0]
```

會被轉成

```SQL
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
ORDER BY name
LIMIT 1;
```

```python
Publisher.objects.order_by('name')[0:2]
```

```SQL
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
ORDER BY name
OFFSET 0 LIMIT 2;
```

## the Django Admin Site

建立super user
```
python manage.py createsuperuser 
```

建立完後我們到admin site看看吧
```
http://127.0.0.1:8000/admin/
```

將books 的tables 加入admin site，首先我們在books資料夾中的admin.py
```python
from django.contrib import admin
from .models import Publisher, Author, Book

admin.site.register(Publisher)
admin.site.register(Author)
admin.site.register(Book)
```

還記得我們的 __str__ 嗎?
```python
class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

    def __str__(self):
        return self.name  #<--- 把這邊修改一下再來看看吧
```