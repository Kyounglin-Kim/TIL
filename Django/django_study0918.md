# django

2020-09-18

#### 오늘학습내용

model를 통해 데이터를 가져오는 활동

model 생성은 class를 만들고 변수지정하면 자동으로 만들어짐

Template는 화면을 담당하는 html영역

그것을 컨트롤하는 것이 view



urls 랑 views랑 템플릿 html 문서들 왔다갔다 하면서 데이터를 주고 받음



![실습18_7](C:\Users\KIMKYOUNLIN\Desktop\django_study0918.assets\실습18_7.PNG)

____



__PollsApp.urls.py__

```python
from django.contrib import admin
from django.urls import path, include
from PollsApp import views

urlpatterns = [
    path('index/', views.index, name = 'index'),
    path('<int:question_id>/', views.choice, name='choice'),
    path('vote/', views.vote, name='vote'),
    path('result/', views.result,name='result'),

]

# <int : 아무이름 설정>
```

​	

__PollsApp.views.py__

```python
from django.shortcuts import render
from django.http import HttpResponse
# Create your views here.
from .models import *

def index(request) :

    lists = Question.objects.all() # 리스트안에 딕셔너리가 들어있음
    print("-"*50)
    print(lists)
    print("-"*50)


    context = {'lists' : lists}
    return render(request, 'polls/index.html', context)


# get 방식 1
#def choice(request, ):
#    request.GET['question.id']

# get 방식 2 - 이 방식은 post는 절대 안됨
def choice(request,question_id):
    print('param question_id', str(question_id))
    lists = get_object_or_404(Question, pk=question_id)
    print("-"*100)
    print(lists) # 여기서 print는 확인해보기위한 디버깅
    print("-" * 100)
    context = {'clist' : lists}
    return render(request, 'polls/choice.html',context)

def vote(request) :
    choice = request.POST['choice']
    question_id = request.POST['question_id']
    print('param value choice - ', str(choice))
    print('param value id - ', str(question_id))

    question = get_object_or_404(Question, pk=question_id)
    checked_choice = question.choice_set.get(pk=choice)
    checked_choice.votes +=1
    checked_choice.save()

    context = {}
    request.session['question_id'] = question_id
    return redirect('result') # 매개변수를 심을수 있냐 없냐의 차이 밑에 함수와
    #return HttpResponseRedirect(reverse('polls:result',args=(question_id,)))

    #return render(request, 'polls/result.html', context)


def result(request):
    question_id = request.session['question_id']
    print('---------------------------   views.result', str(question_id))
    question = get_object_or_404(Question, pk=question_id)
    context = {'question':question}

    return render(request, 'polls/result.html',context)

```



__Sql(PYTHON에서 작성)  VS 장고에서  데이터베이스에 데이터 추출법 __

```python
# modelName.objects.all() # 모든 데이터를 가져옴

# select * from table where id = 1;   -> modelName.objects.filter(id=1)
# filter가 where조건절에 들어간다고 생각하면됨

# And 조건 비교
# select * from table where id = 1 and pwd =1 ;
# -> modelName.objects.filter(id =1 , pwd = 1)

# Or 조건 비교
# select * from table where id = 1 or pwd =1 ;
# -> modelName.objects.filter(Q(id =1) | Q(pwd = 1))

# 패턴 검색
# select * from table where subject like '%공지%'
# -> modelName/objects.filter(subject_icontains='공지')
# select * from table where subject like '공지%'
# -> modelName/objects.filter(subject_startwith='공지')
# select * from table where subject like '%공지'
# -> modelName/objects.filter(subject_endswith='공지')

# 삽입
# insert into table values('')
# model(attr = value , attr = value .....)
# -> model.save()

# 삭제 -> 객체를 넘겨받아서 삭제 
# delete * from table where id = 1
# -> modelName.objects.get(id=1).delete()

# update table set attr = value where id = 1
# obj = modelName.objects.get(id=1) 먼저 데이터를 하나 꺼내옴
# obj.attr = '변경' 개체의 속성 변경
# obj.save()  -- commit / save를 통해 업데이트
```


__PollsApp.models.py__  

* 모델을 수정하면 마이크레이션 작업을 다시 해주어야 한다.

```python
from django.db import models

# Create your models here.

class Question(models.Model) :#모델을 상속해 줘야함 # 1) 기본키 - id 컬럼 integer 타입
    question_text = models.CharField(max_length=200) # 길이 최대 200바이트인 열을 만든다는 뜻 2)
    regdate = models.DateTimeField('date published') # 여기까지 3개의 컬럼이 생긴다,

    def __str__(self): # 객체에 문자열을 리턴받겠다
        return self.question_text # 같은 문자로 묶어야 한다.

# 선택형 테이블을 만들어준다.
class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE) #CASCADE 함수 아니고 속성
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return str(self.question)+" , "+self.choice_text+" , "+str(self.votes)

#Question과 Choice라는 테이블을 만들었다고 생각하면 됨
```

__PollsApp.admin.py__ 

```python
from django.contrib import admin
from .models import *

# Register your models here.
admin.site.register(Question)
admin.site.register(Choice) # 2개의 모델을 추가시켜줘야한다.
```

이 다음 작업은 모델을 `마이크레이션`  해야한다.

> python manage.py makemigrations
>
> python manage,py 

![실습17_1](C:\Users\KIMKYOUNLIN\Desktop\django_study0918.assets\실습17_1.PNG)



![실습18_2](C:\Users\KIMKYOUNLIN\Desktop\django_study0918.assets\실습18_2-1600394594339.PNG)

* `templates`라는 프로젝트 생성 후 어플관련 프로젝트 polls 생성하기

* 템플릿을 통해 화면에 보여지게 된다.  view를 통해서는 작업하기어려움

* 그 후 html파일을 생성

__index.html__

```html
<body>
    {% if lists %}
    <ul>
        {% for question in lists %}
        <li><a href="../{{question.id}}">{{ question.question_text }}</a></li>
        {% endfor %} # 여기서 <ㅁ>는 링크를 걸어주는 역할을 한다.
    </ul> # 루프를 돌아야 한다. -> 화면상에 동적으로 데이터를 심어주는 작업
    {% else %}
        <p>데이터가 존재하지 않습니다~~~</p>

    {% endif %}
</body>
</html>
```

__choice.html__

* form을 해줘야 데이터를 전송해 준다.

* {% csrf_token %}:post방식의 보안양식

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>{{clist.question_text}}</h1>
    <hr/>
    <form method="post" action="{% url 'vote' %}">
    <input type="hidden" name="question_id" value="{{clist.id}}"> <!-- 이런식으로 hidden으로 형태는 안보이게 하되 값은 넣어서 보내준다. -->
        {% csrf_token%} <!-- django 에서 post를 쓸때는 csrf 토큰을 사용한다. 보안을 위해서 이다. -->
        {% for  choice in clist.choice_set.all%}<!--//  xxx_set.all 은  템플릿쪽에서 사용하는 코드로 모든 객체들을 가져온다.  -->
    <input type="radio"
           name="choice"
           value="{{choice.id}}"> <!-- name 이라는 것이 변수 이름이다.-->
    <label>
        {{ choice.choice_text}}
    </label><br/>
    {% endfor %}
        <p/>
        <input type="submit" value="VOTE">
    </form>
</body>
</html>
```

__result.html__

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>{{ question.question_text }}</h1>
    <hr/>

    <ul>
        {% for choice in question.choice_set.all %}
        <li>{{choice.choice_text}} - {{choice.votes}}</li>
        {% endfor %}
    </ul>
    <p/>
    <a href="{% url 'index' %}">첫 페이지로 이동</a>


</body>
</html>
```

## 결과

1. 질문 리스트 및 투표와 링크연결

![실습18_5]()![실습18_4](C:\Users\KIMKYOUNLIN\Desktop\django_study0918.assets\실습18_4-1600417443805.PNG)

2. 질문에 대한 투표 생성

![실습18_5](C:\Users\KIMKYOUNLIN\Desktop\django_study0918.assets\실습18_5-1600417463015.PNG)

3. 투표 할 때마다 투표수가 증가되게 표현

![실습18_6](C:\Users\KIMKYOUNLIN\Desktop\django_study0918.assets\실습18_6.PNG)



### 실습후 소감

> 오늘은 어제보다는 실습을 따라가는데 있어서 문제는 없었지만 아직 템플릿을 통해 웹을 실행하는 그 과정들이 복잡하고 헷갈리는 부분이 많다. 그러나 아직 장고를 배운지 이틀이 지났고 하루하루 배우다보면 코딩에 대한 이해를 할 수 있을거라 생각한다.