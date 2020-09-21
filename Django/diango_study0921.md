# diango

2020-09-21



------

새로운 App 만들기

python manage.py starApp BbsApp



diangoWEB

__diagoWEB.urls__ 에서 PATH를 입력하여 연결해 주어야 한다.

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('hello/', include('helloApp.urls')),
    path('polls/', include('PollsApp.urls')),
    path('bbs/', include('BbsApp.urls')) # 새로 추가된 어플
    # 여기서 bbs는 http://localhost:8000/bbs/index/에 들어가는 경로이다.
    
]
```

__diagoWEB.settings__

```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'BbsApp','static')

]
STATIC_ROOT = os.path.join(BASE_DIR,'static')
# static 파일에 css 자바 이미지들을 관리하겠다는 설정
```

__static 파일을 한 곳으로 모을 때 __

python manage.py collectstatic



__모델 생성후__ 

python manage.py makemigrations

python manage.py migrate





__templates__ 

- html (tag + text)
- {{print}}
- {% 동적 코드를 작성하고자 할 때 %}
- {% 프리젠테이션 레이어에 로직을 심을 수 있다 %}



```html
<!--header.html 이랑 footer.html을 home.html 페이지의 윗 화면이랑 아래 화면에 불러옴-->

{% load static %}
----------------------------------------home.html
{ % block content %}
<section></section>
{% endblock %}
{% include 'footer.html' %}

```



