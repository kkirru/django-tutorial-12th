# Django tutorial 12th

## @ Part 1. 프로젝트 생
### 1.1 Django version 확인
    pip list
    or
    python -m django --version

### 1.2 Project 생성
    django-admin startproject mysite

### 1.3 개발 서버 
    python manage.py runserver <port number>

### 1.4 앱 생성하기
    python manage.py startapp <app name>
   
- view.py 작성
- urls.py 생성 -> 최상위 URLconf에서 polls.urls 모듈을 바라보게 설정

<hr/>


## @ Part 2. 장고 앱 작성하기
### 2.1 데이터베이스 설치
    python manage.py migrate
- migrate : settings.py 파일을 보고 필요한 db 테이블을 생성해준다.

### 2.2 모델 만들기
- models.py에 클래스 형태로 모델 작성 
```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
```
### 2.3 모델 활성화
- 앱을 프로젝트에 포함시키기 위해 settings.py.INSTALLED_APPS에 PollsConfig 추가


     polls.apps.PollsConfig
- 앱 마이그레이션


    python manage.py makemigrations polls
    
### 2.4 API 가지고 놀기
- python shell 실행   
    
    
    python manage.py shell
- 객체 표현을 위해 __str__ 메서드 모델 클래스에 추가

### 2.4 장고 관리자
- wow 대박... 왜 프레임워크를 쓰는지 알겠습니다.
- superuser 생성

    
    python manage.py createsuperuser
- 앱을 관리 사이트에 추가
    - admin.py에 모델 register
```python
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

## @ Part3

## 3.1 뷰 추가하기
- detail, results, vote 등
- urls.py에 url pattern 입력해주어야 함
```python
urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

## 3.2 뷰가 실제로 무언가 하게 만들기
- 뷰는 HttpResponse 객체를 반환하거나 예외를 발생시킨다.
- 템플릿 사용하기
    - ```polls/templates/polls/index.html``` 파일 생성 
```python
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```
- template shortcut : render
```python
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

- throw 404 error
    - ``` raise Http404("Question does not exist")``` 

- shortcut for 404
    - ```    question = get_object_or_404(Question, pk=question_id) ```
    
- 템플릿 하드코딩 URL 제거
    - ```<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>```
    - --> ``` <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li> ```