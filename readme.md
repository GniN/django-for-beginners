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

如果我們希望在url 後面加上參數，讓顯示的時間可以是現在時間加上幾小時，讓網址可以向是下面這樣。

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
    url(r'^time/plus/\d+/$', hours_ahead),
    # ...
]
```