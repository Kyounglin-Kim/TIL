# django

2020-09-17

위 실습은 `파이참` 프로그램을 통해 학습한 내용입니다.



## MVT 패턴이란

> 장고 프레임워크에서는 View를 Template, Controller는 View라고 표현되며,  MVC를 MVT패턴이라고 한다. 모델은 데이터 베이스에 저장되는 데이터를 의미하는 것이고, 템플릿은 사용자에게 보여지는 UI부분을, 뷰는 실질적으로 프로그램 로직이 동작하여 데이터를 가져오고 적절하게 처리한 결과를 템플릿에 전달하는 역할을 수행한다.



* 장고라는 프레임워크는 상대방의 요청을 `URL conf`에서 관리
* URL 분석 결과를 통해 해당 URL에 대한 처리를 담당할 뷰를 결정한다
* `View`는 자신의 로직을 실행하면서 만일 데이터 베이스 처리가 필요하면 모델을 통해 처리하고 그 결과를 반환 받는다.  사용자의 요청을 받아 비즈니스 로직이 들어있음
* 뷰는 자신의 로직 처리가 끝나면 템플릿을 사용하여 클라이언트에 전송한 `HTML 파일`을 생성한다.
* 뷰는 최종결과로 HTML 파일을 `클라이언트`에 보내 응답한다.



![장고](./img/장고_0917.PNG)



View : URL conf -> View

Model :  MODEL <----ORM----> DB

T : View ----- Template 



## MVT 코딩 순서



무엇을 먼저 코딩해야하는 지는 정해진 것이 없다. UI 화면을 생각하면서 로직을 끌어나가는 것이 쉬울 때에는 보통 템플릿을 먼저 코딩한다.

1. 프로젝트 뼈대 만들기 : 프로젝트 및 앱 개발에 필요한 디렉토리와 파일 생성
2. 모델 코딩하기 : 테이블 관련 사항을 개발(models.py, admin.py 파일)
3. URL_conf 코딩하기 : URL 및 뷰 매핑 관계를 정의(urls.py 파일)
4. 템플릿 코딩하기 : 화면 UI 개발(templates/디렉토리 하위의 *.html 파일들)
5. 뷰 코딩하기 : 어플리케이션 로직 개발 (views.py 파일)



[코드설명]

```python
# 프로젝트 생성
diango-admin startproject webserver
dir/w
cd webserver
dir/w

# App 생성
python manage.py startapp greetingApp

# Server 실행하는 명령어
python manage.py runserver # runserver뒤에 7000번 이런식으로 입력안하면 default 8000번

# 사용자 입장에서
http://localhost:8000/hello/index

# 모델 마이크레이션
# 사용자 모델 -> DB(테이블) 만드는 명령어
python manage.py makemigrations # 모델을 통해 테이블 생성
python manage.py migrate # 테이블의 속성들을 실제로 반영

python manage.py createsuperuser # 확인하기 위한 admin의 계정을 생성..?
```

templates

- html (tag + text)
- {{print}}
- {% 동적 코드를 작성하고자 할 때 %}
- {% 프리젠테이션 레이어에 로직을 심을 수 있다 %}



## 실습 

먼저 `django` 패키지를 설치해야한다.

##### 어플생성 - python manage.py startapp greetingApp

![파일생성](./img/파일생성_0917.PNG)





프로젝트, python 파일, html 파일 생성하기

`python참 코딩` 

1) settings.py

```python
"""
Django settings for djangoWEB project.

Generated by 'django-admin startproject' using Django 3.0.3.

For more information on this file, see
https://docs.djangoproject.com/en/3.0/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/3.0/ref/settings/
"""

import os

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/3.0/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'yg7^20=bb9(#ou)i2o08q9=1lv)lk#t2#t+#)vlw4d3!x-^#a2'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = ['127.0.0.1','localhost'] #로컬 지정해주기 


# Application definition

# 만드는App 추가해주기 -> helloApp

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'helloApp',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'djangoWEB.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
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

WSGI_APPLICATION = 'djangoWEB.wsgi.application'


# Database
# https://docs.djangoproject.com/en/3.0/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}


# Password validation
# https://docs.djangoproject.com/en/3.0/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/3.0/topics/i18n/

LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'Asia/Seoul' # 서울시간으로 맞추기

USE_I18N = True

USE_L10N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.0/howto/static-files/

STATIC_URL = '/static/'
```



2) djangoWEB\urls.py

```python
from django.contrib import admin
from django.urls import path, include # include 함수 불러오기

urlpatterns = [
    path('admin/', admin.site.urls),
    path('hello/', include('helloApp.urls'))
]
```

3) helloApp\urls.py

```python
from django.contrib import admin
from django.urls import path, include
from helloApp import views

urlpatterns = [
    path('index/', views.index),
    path('hi/', views.hi),
    path('login/', views.login, name='login')
]
```

4) views.py 

```python
from django.shortcuts import render, HttpResponse
from .models import *

# Create your views here.

def index(request):
    #return HttpResponse('<div align=center> 섭이와 함께하는 Django WEB Framework</div>')
    return render(request, 'hello/index.html')

def hi(request) :
    context = {'ment' : '여기까지 잘 되시나요? 쉬는시간 가질까요? ^*^'}
    return render(request , 'hello/ok.html', context)

# post 방식과 get 방식이 존재 post는 주소창에 내가 입력한 id/pwd가 나오지 않음
def login(request):
    if request.method == 'POST': 
        id = request.POST['id']
        pwd = request.POST['pwd']


        user = TestUser.objects.get(user_id = id)
        context = {}
        if user is not None :
            #print(user.user_id, user.user_pwd, user.user_name)
            context['user'] = user
        return render(request, 'hello/success.html', context)


        #context = {'id' : id , 'pwd' : pwd}
        #return render(request, 'hello/success.html', context)
```

5) models.py

```python
from django.db import models

# Create your models here.

class TestUser(models.Model) :
    user_id = models.CharField(max_length=50)
    user_pwd = models.CharField(max_length=50)
    user_name = models.CharField(max_length=50)
```

6) admin.py

```python
from django.contrib import admin
from .models import *
# Register your models here.

admin.site.register(TestUser)
```

7) ok.html

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<center><font color = "pink"> {{ment}}</font></center>
</body>
</html>
```

8) index.html

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <center>NAVER</center>
    <div align="right">
        <form method="post" action="{% url 'login' %}">
            {% csrf_token %}
        <label>아이디</label> <input type="text" name="id" />
        <label>패스워드</label> <input type="password" name="pwd" />
        <input type = 'submit' value="LOGIN">
        </form>
    </div>
</body>
</html>
```

9) success.html -> 로그인을 했을 때 나타나는 웹 페이지

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
{{user.user_id}} 이고 {{user.user_pwd}} 이며 {{user.user_name}} 입니다~~~
</body>
</html>
```



## 실행 결과

__ok.html 실행 결과__

![실습1](./img/실습1_0917.PNG)



__index.html 실행 결과__

![실습2](./img/실습2_0917.PNG)



__success.html 실행 결과__

index 웹에서 로그인을 한 결과 

> python manage.py makemigrations # 모델을 통해 테이블 생성
>
> python manage.py migrate  # 실행 후 -> 아이디 및 패스워드 생성 
>
> python manage.py createsuperuser



__생성한 아이디 비번을 통해 들어갔을때 나오는 화면__

![실습3](./img/실습3_0917.PNG)



__success에 들어가기 위한 아이디 비밀번호 생성 __

![실습4](./img/실습4_0917.PNG)



__index에서 로그인 성공후 나오는 웹 화면 __

![실습5](./img/실습5_0917.PNG)



## 실습 후 소감

장고 첫 수업이라 정신이 하나도 없고 환경셋팅 자체도 힘들어서 정신이 없는 하루였지만, 조금씩 하다보면 장고에 대해 이해할 수 있을 거라 믿고 앞으로 열심히 수업듣고 이렇게 복습하고 1일 1commit 이상을 하도록 노력해야겠습니다.

