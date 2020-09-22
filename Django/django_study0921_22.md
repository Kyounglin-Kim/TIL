# diango

2020-09-21 ~ 09-22

[학습내용]

모델에서 Bbs클래스라는 데이터베이스 테이블을 생성한 뒤 관리자페이지에서 이 테이블에 데이터를 추가하고 데이터를 모델에서 view로 옮겨 templates에 있는 html을 통해 출력해서 보여주는 활동을 했다.

게시판의 글을 몇명이 봤는지에 대한 viewcnt와 TITLE 제목에 url을 연결하여 게시판의 내용을 볼 수 있게하고 마지막으로 게시판을 쓴 작성자가 게시글이 마음에 들지 않을때 삭제 할 수 있게 만들어주는 버튼을 생성하는 하는 내용을 배웠다.

------

__새로운 App 만들기__

python manage.py starApp BbsApp

[diangoWEB]

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



__models.py__

```python
from django.db import models
from django.utils import timezone
# Create your models here.

class BbsUserRegister(models.Model):
    user_id = models.CharField(max_length=50)
    user_pwd = models.CharField(max_length=50)
    user_name = models.CharField(max_length=50)

    def __str__(self):
        return self.user_id+" , "+self.user_pwd+" , "+self.user_name


class Bbs(models.Model):
    # id = models.AutoField(primary key = True) -> 이것은 기본키로 지정되므로 코딩안해도됨
    title = models.CharField(max_length=100)
    writer = models.CharField(max_length=100)
    content = models.TextField()
    regdate = models.DateTimeField(default=timezone.now)
    viewcnt = models.IntegerField(default=0) # 모델을 만들었으면 테이블생성이라는 과정이 필요하다. 반영을 위해 migration 그 다음 admin

    def __str__(self):
        return self.title
```



__모델 생성후__ terminal에 실행 

python manage.py makemigrations

python manage.py migrate



__admin.py__

```python
from django.contrib import admin
from .models import *  # 모델을 불러와야함

# Register your models here.
admin.site.register(BbsUserRegister)
admin.site.register(Bbs)
```

__BbsApp.urls__

```python
from django.contrib import admin
from django.urls import path, include
from BbsApp import views

# path의 첫번째는 url경로
urlpatterns = [
    path('index/', views.loginForm,name='loginForm'), # view-> urls에 들어온 리퀘스를 로직을 처리하는곳
    path('registerForm/', views.registerForm, name='registerForm'),
    path('register/', views.register, name='register'),
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),
    path('bbs_list/', views.list, name='bbs_list'),
    path('bbs_registerForm/', views.bbsRegisterForm, name = 'bbs_registerForm'),
    path('bbs_register/', views.bbsRegister , name = 'bbs_register'),
    path('bbs_read/<int:id>', views.bbsRead, name='bbs_read'), #<int:id>는 list에 적용한 값과 int값이 동일해야한다. // GET방식
    path('bbs_remove/', views.bbsRemove, name='bbs_remove'),

]
```



__views.py__

- render(request , templates(html), context) - forword

- context에 데이터를 심으면 해당 데이터는 요청된 페이지내에서만 사용이 가능한 scope가지게 된다.

- session에 심어서 데이터 사용범위를 모든 템플릿에서 사용할 수 있는 데이터로 저장해야 한다.

```python
from django.shortcuts import render, redirect
from django.http import HttpResponse

from .models import *

# Create your views here.

def loginForm(request) :
    return render(request , 'login.html')

def registerForm(request):
    return render(request, 'join.html')

def register(request):
    if request.method == 'POST':
        id = request.POST['id']
        pwd = request.POST['pwd']
        name = request.POST['name']

        register = BbsUserRegister(user_id=id, user_pwd=pwd, user_name=name)
        register.save()
    return redirect('loginForm') # redirect 새로운 요청 가능


def login(request):
    if request.method == 'GET':
        return redirect('login')
    elif request.method == 'POST':
        id = request.POST['id']
        pwd = request.POST['pwd']
        #user = BbsUserRegister.objects.filter(user_id=id, user_pwd=pwd) 이렇게 하면 쿼리셋이라는 타입이 넘어옴
        user = BbsUserRegister.objects.get(user_id=id, user_pwd=pwd) # 모델 객체가 넘어옴
        print('user result : ' , user)
        context={}
        if user is not None : # 만약 정상적으로 작동했다면
            request.session['user_name'] = user.user_name
            context['userSession'] = request.session['user_name']


    return render(request, 'home.html',context) # render는 템플릿을 찾는것이다.

def logout(request):
    # usersession을 날리는 작업
    request.session['user_name'] ={}
    request.session.modified = True

    return redirect('loginForm')

def list(request):
    # select * from BbsApp;
    # -> modelName.objects.all()

    boards = Bbs.objects.all()
    print('boards result - ',type(boards), boards)
    context = {'boards': boards}
    return render(request, 'list.html',context) # 템플릿으로 가겠다 맨 오른쪽에 데이터를 심는다

# 리스트가 header와 footer에 심어져 있기때문에 이렇게 정보를 심으면 적용이 가능해 진다.

def bbsRegisterForm(request):
    context = {'name' : request.session['user_name'],
               'id' : request.session['user_id']}
    return render(request, 'bbsRegisterForm.html',context)


def bbsRegister(request):
    if request.method == 'GET':
        return redirect('bbs_registerForm')
    elif request.method == 'POST':
        title = request.POST['title']
        content = request.POST['content']
        writer = request.POST['writer']

        board=Bbs(title = title, content = content, writer = writer)
        board.save()
    return redirect('bbs_list') # render는 템플릿이기 때문에 데이터를 가져올수 없음


def bbsRead(request,id): # get에서 가져오는 값도 적용해야함
    # print('param - ', id)
    # model과 작업을 필요로하게 된다.
    bbs = Bbs.objects.get(id=id)
    # viewcnt update
    bbs.viewcnt = bbs.viewcnt + 1
    bbs.save()

    context = {'bbs': bbs ,
               'name': request.session['user_name'],
               'id': request.session['user_id']}

    return render(request,'read.html',context)


def bbsRemove(request):
    id = request.POST['id']
    #delet from table where id = id;
    board = Bbs.objects.get(id=id)
    board.delete()
    return redirect('bbs_list')


```

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

#### templates의 html 파일들 예시

 __list.html__

```html
<!-- list는 body만 가지고 있다 그러므로 header랑 footer를 붙이는 작업이 필요하다 -->

{% include 'header.html' %}
{% block content %}



<!-- Main content -->
<section class="content">
	<div class="row">
		<!-- left column -->
		<div class="col-md-12">
			<!-- general form elements -->
            <!-- general form elements -->
			<div class='box'>
				<div class="box-header with-border">
					<h3 class="box-title">Board List</h3>
				</div>
				<div class='box-body'>
					
					<button id='newBtn'>New Board</button>
				
				</div>
			</div>

          <div class="box">
				<div class="box-header with-border">
					<h3 class="box-title">LIST ALL PAGE</h3>
				</div>
				<div class="box-body">

{% if boards %}
<table class="table table-bordered">
	<tr> <!--이 부분이 테이블의 header이다.-->
		<th style="width: 10px">BNO</th>
		<th>TITLE</th>
		<th>WRITER</th>
		<th>REGDATE</th>
		<th style="width: 40px">VIEWCNT</th>
	</tr>

	<tbody id="tbody">
	{% for board in boards %}
	<tr> <!-- 테이블을 구성하는 값들 ~ 데이터를 모델로부터 가져와야한다.-->
		<td>{{board.id}}</td>
		<td><a href="OOO">{{board.title}}</a></td>
		<td>{{board.writer}}</td>
		<td>{{board.regdate}}</td>
		<td><span class="badge bg-red">{{board.viewcnt}}</span></td>
	</tr>
    {% endfor %}
	</tbody>

</table>
{% else %}
					<p>데이터가 존재하지 않습니다.</p>
{% endif %}

				</div>
				<!-- /.box-body -->
				<div class="box-footer">Footer</div>
				<!-- /.box-footer-->
			</div>
		</div>
		<!--/.col (left) -->

	</div>
	<!-- /.row -->
</section>
<!-- /.content -->
</div>
<!-- /.content-wrapper -->


<script>

</script>


{% endblock %}
{% include 'footer.html' %}
```

__bbsRegisterForm.html__

```html
<!-- list는 body만 가지고 있다 그러므로 header랑 footer를 붙이는 작업이 필요하다 -->
<!-- 화면이 깨지는 것을 막기 위해 블럭을 해주어야 한다. include block ~ endblock include-->
<!-- css 방식 -->


{% include 'header.html' %}
{% block content %}




<!-- Main content -->
<section class="content">
	<div class="row">
		<!-- left column -->
		<div class="col-md-12">
			<!-- general form elements -->
			<div class="box box-primary">
				<div class="box-header">
					<h3 class="box-title">REGISTER BOARD</h3>
				</div>
				<!-- /.box-header -->

<form role="form" method="post" action="{% url 'bbs_register' %}">
	<!--사용자가 버튼을 누르면 bbsRegister path로 연결-->
	{% csrf_token %} <!--POST이므로 보안 필요 -->
   
<!--데이터를 입력하면 db에 저장되는 작업을 해야한다. -->
	<div class="box-body">
		<div class="form-group">
			<label for="exampleInputEmail1">Title</label> 
			<input type="text"
				name='title' class="form-control" placeholder="Enter Title">
		</div>
		<div class="form-group">
			<label for="exampleInputPassword1">Content</label>
			<textarea class="form-control" name="content" rows="3"
				placeholder="Enter ..."></textarea>
		</div>
		<div class="form-group">
			<label for="exampleInputEmail1">Writer</label> 
			<input type="text"
				name="writer" class="form-control" readonly value="{{id}}"> <!--id session 값을 가져오겠다-->

		</div>
	</div>
	<!-- /.box-body -->

	<div class="box-footer">
		<button type="submit" class="btn btn-primary">Submit</button>
	</div>
</form>


			</div>
			<!-- /.box -->
		</div>
		<!--/.col (left) -->

	</div>
	<!-- /.row -->
</section>
<!-- /.content -->
</div>
<!-- /.content-wrapper -->



{% endblock %}
{% include 'footer.html' %}

```



__read.html__

```html
{% include 'header.html' %}
{% block content %}


<!-- Main content -->
<section class="content">
	<div class="row">
		<!-- left column -->
		<div class="col-md-12">
			<!-- general form elements -->
			<div class="box box-primary">
				<div class="box-header">
					<h3 class="box-title">READ BOARD</h3>
				</div>
				<!-- /.box-header -->

<form role="form" method="post" id="removeFrm">
	{% csrf_token %}
	
	<input type='hidden' name='id' value="{{bbs.id}}">

</form>

<div class="box-body">
	<div class="form-group">
		<label for="exampleInputEmail1">Title</label> <input type="text"
			name='title' class="form-control" value="{{bbs.title}}"
			readonly="readonly">
	</div>
	<div class="form-group">
		<label for="exampleInputPassword1">Content</label>
		<textarea class="form-control" name="content" rows="3"
			readonly="readonly">{{bbs.content}}</textarea>
	</div>
	<div class="form-group">
		<label for="exampleInputEmail1">Writer</label> <input type="text"
			name="writer" class="form-control" value="{{bbs.writer}}"
			readonly="readonly">
	</div>
</div>
<!-- /.box-body -->

<div class="box-footer">
	{% if id == bbs.writer %}
	<button type = "submit" class="btn btn-warning">Modify</button>
	<button type = "submit" class="btn btn-danger">Remove</button>
	{% endif %}
	<button id = "listBtn" type="submit" class="btn btn-primary">LIST ALL</button>
</div>



<!--#은 불러오는 거  즉 버튼을 불러온다는 뜻 -->
				<!--#removeFrm은 submit을 생성시키기 위한 버튼..?  -->
<script>

$(document).ready(function(){
	$('#listBtn').click(function() {
		location.href = '../bbs_list';
	})
	$('.btn-danger').click(function(){
		$('#removeFrm').attr('action', '../bbs_remove/');
		$('#removeFrm').submit();

	})
});


</script>


			</div>
			<!-- /.box -->
		</div>
		<!--/.col (left) -->

	</div>
	<!-- /.row -->
</section>
<!-- /.content -->
</div>
<!-- /.content-wrapper -->

{% endblock %}
{% include 'footer.html' %}
```



__실습 결과 __

header와 footer.html을 제공받아 웹 사이트를 만들었다. 오늘 한 내용은 Dashboard 게시판에 관리자계정에서 입력한 데이터를 나타내는 작업을 했다.

게시글을 작성하고 작성한 글을 url을 통해 연결하여 확인하는 작업

마지막으로 게시글 작성자가 게시글을 삭제하는 작업을 했다.

![실습1](./img/실습1_0922.PNG)

저기서 viewcnt는 게시글을 조회할때마다 하나씩 증가한다.

![실습2](./img/실습2_0922.PNG)

게시글을 올린 사람이 글을 확인할때 remove 버튼을 통해 삭제가 가능하다.



### 실습 후 소감

>2020-09-22
>
>오늘 배운 내용이 완벽하게 이해된것은 아니지만 약간의 흐름이 3일째가 되는 오늘 조금씩 이해가 되는 기분이 든다.  Diango책을 통해 스스로 공부하여 html에 대해 이해하는 시간이 필요한것 같다.
>
>2020-09-23
>
>오늘은 어제 실습에 더해 추가적으로 웹을 구성하는 것을 배웠다. 확실히 하루하루 지날수록 혼자서 실습을 조금이지만 할 수 있게 되었다. 
