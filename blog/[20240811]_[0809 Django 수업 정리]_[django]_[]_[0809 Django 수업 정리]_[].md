```python

....................참고 사항.......................

CSRF -> 악의적인 웹사이트 사용자가
사용자의 브라우저를 활용해서 사용자의 인증상태를 확인하고, 다른 웹사이트 URL -> 요청을 보내는 하는 방법!

CSRF 토큰 -> 출처가 제대로 된 무언가를 확인!
보통은 POST???

GET 데이터를 조회 -> 데이터 조회! -> 상태를 변경x GET 조회 -> 악의 -> 변경x -> 위험하지 않다.
POST 데이터를 조회 -> 상태 변경 -> 악의적인 형식 -> 사용자 계정 정보 바꾸거나 출금! 

GET 사용자의 상태를 변경하지 않는다. 주 원칙! -> POST요청! PUT /DELETE



............................수업 내용.................

시작부터 빡셈

# 폴더 생성
DJANGO_MODU_02


# 환경 설정
'''
python -m venv venv

.\venv\Scripts\activate

pip install django
'''

# 새로운 프로젝트 생성
django-admin startproject todoproject

#경로 변경
cd todoproject

#앱 생성
python manage.py startapp todoapp


#settings.py 

'''
INSTALLED_APPS = [

"todoapp", 
]


LANGUAGE_CODE = 'ko-kr'

TIME_ZONE = 'Asia/Seoul'
'''



# models.py 작성

# 마이그레이션
'''
python manage.py makemigrations
python manage.py migrate
'''


#쉘로 데이터 입력
'''
from todoapp.models import List, Task
from django.utils import timezone

# 쇼핑 목록 생성
shopping_list = List.objects.create(name="쇼핑 목록", description="주간 쇼핑 목록")
print(f"생성된 목록: {shopping_list}")

# 공부 목록 생성
study_list = List.objects.create(name="공부 목록", description="이번 주 공부할 내용")
print(f"생성된 목록: {study_list}")


# 쇼핑 목록에 할 일 추가
task1 = Task.objects.create(
    title="우유 구매",
    description="1리터 우유 2개 구매",
    due_date=timezone.now() + timezone.timedelta(days=1),
    list=shopping_list
)
print(f"생성된 할 일: {task1}")

task2 = Task.objects.create(
    title="빵 구매",
    description="통밀빵 1개 구매",
    due_date=timezone.now() + timezone.timedelta(days=1),
    list=shopping_list
)
print(f"생성된 할 일: {task2}")

# 너무 내용이 길어서 task2까지 우선 입력해보자
# 공부 목록에 할 일 추가
task3 = Task.objects.create(
    title="Django ORM 학습",
    description="Django ORM 기본 쿼리 학습하기",
    due_date=timezone.now() + timezone.timedelta(days=3),
    list=study_list
)
print(f"생성된 할 일: {task3}")

'''


# 데이터 조회해서 제대로 insert되었는지 확인
'''
List.objects.all()

shopping_list.tasks.all()

study_list.tasks.all()

Task.objects.filter(due_date__lte=timezone.now() + timezone.timedelta(days=2))
'''

# todoproject/urls.py
'''
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("todoapp.urls")),
]
'''

# todoapp에 urls.py 생성해서 아래의 내용 작성
'''
from django.urls import path
from . import views

urlpatterns = [
    path('', views.task_list, name='task_list'),
]
'''

# todoapp/views.py
'''
from django.shortcuts import render
from .models import Task

def task_list(request):
    tasks = Task.objects.all().order_by('-created_date')
    return render(request, 'todoapp/task_list.html', {'tasks': tasks})
'''

# todoapp/templates/todoapp/task_list.html 작성
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>할 일 목록</title>
    <style>
        .completed {
            text-decoration: line-through;
            color: gray;
        }
    </style>
</head>
<body>
    <h1>할 일 목록</h1>
    <ul>
        {% for task in tasks %}
            <li class="{% if task.completed %}completed{% endif %}">
                {{ task.title }}
                {% if task.due_date %}
                    (마감일: {{ task.due_date|date:"Y-m-d" }})
                {% endif %}
            </li>
        {% empty %}
            <li>할 일이 없습니다.</li>
        {% endfor %}
    </ul>
</body>
</html>
'''

# todoapp/forms.py 작성
'''
from django import forms
from .models import Task

class TaskForm(forms.ModelForm):
    class Meta:
        model = Task
        fields = ['title', 'description', 'due_date', 'list']
        widgets = {
            'due_date': forms.DateTimeInput(attrs={'type': 'datetime-local'}),
        }
'''

# todoapp/views.py 수정(전체 복붙 가능)
'''
from django.shortcuts import render, redirect
from .models import Task
from .forms import TaskForm

def task_list(request):
    tasks = Task.objects.all().order_by('-created_date')
    return render(request, 'todoapp/task_list.html', {'tasks': tasks})


def add_task(request):
    if request.method == 'POST': # 폼이 제출 되었을때 POST 요청을 처리한다. 처리가됨
        form = TaskForm(request.POST) # POST 요청으로 전달된 데이터를 사용해 TaskForm 인스턴스를 생성한다.
        if form.is_valid(): # form의 데이터가 유효한지 검사 -> 모델/제약조건을 기반으로 자동 유효성 검증을 수행한다.
            form.save() # form의 데이터를 데이터베이스에 저장한다.
            return redirect('task_list') # task_list 뷰로 이동한다.
    else:
        form = TaskForm() # GET 요청일때는 빈 TaskForm 인스턴스를 생성한다.
    return render(request, 'todoapp/add_task.html', {'form': form}) # 폼을 포함한 add_task.html 템플릿을 렌더링한다.
'''

# todoapp/templates/todoapp/add_list.html 작성
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>할 일 추가</title>
</head>
<body>
    <h1>할 일 추가</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">추가</button>
    </form>
    <a href="{% url 'task_list' %}">목록으로 돌아가기</a>
</body>
</html>
'''

# todoapp/urls.py 수정(전체 복붙 가능)
'''
from django.urls import path
from . import views

urlpatterns = [
    path('', views.task_list, name='task_list'),
    path('add/', views.add_task, name='add_task'),
]
'''

# todoapp/templates/todoapp/task_list.html 수정(전체 복붙 가능)
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>할 일 목록</title>
    <style>
        .completed {
            text-decoration: line-through;
            color: gray;
        }
    </style>
</head>
<body>
    <h1>할 일 목록</h1>
    <a href="{% url 'add_task' %}">새 할 일 추가</a>
    <ul>
        {% for task in tasks %}
            <li class="{% if task.completed %}completed{% endif %}">
                {{ task.title }}
                {% if task.due_date %}
                    (마감일: {{ task.due_date|date:"Y-m-d" }})
                {% endif %}
            </li>
        {% empty %}
            <li>할 일이 없습니다.</li>
        {% endfor %}
    </ul>
</body>
</html>
'''

# todoapp/views.py 수정(전체 복붙 가능)
'''
from django.shortcuts import render, redirect, get_object_or_404
from .models import Task
from .forms import TaskForm

def task_list(request):
    tasks = Task.objects.all().order_by('-created_date')
    return render(request, 'todoapp/task_list.html', {'tasks': tasks})


def add_task(request):
    if request.method == 'POST': # 폼이 제출 되었을때 POST 요청을 처리한다. 처리가됨
        form = TaskForm(request.POST) # POST 요청으로 전달된 데이터를 사용해 TaskForm 인스턴스를 생성한다.
        if form.is_valid(): # form의 데이터가 유효한지 검사 -> 모델/제약조건을 기반으로 자동 유효성 검증을 수행한다.
            form.save() # form의 데이터를 데이터베이스에 저장한다.
            return redirect('task_list') # task_list 뷰로 이동한다.
    else:
        form = TaskForm() # GET 요청일때는 빈 TaskForm 인스턴스를 생성한다.
    return render(request, 'todoapp/add_task.html', {'form': form}) # 폼을 포함한 add_task.html 템플릿을 렌더링한다.

def edit_task(request, task_id):
    task = get_object_or_404(Task, id=task_id) # pk가 주어진 Task 인스턴스를 가져온다. 없으면 404 에러를 발생시킨다.
    if request.method == 'POST':
        form = TaskForm(request.POST, instance=task) # POST 요청으로 전달된 데이터와 task 인스턴스를 사용해 TaskForm 인스턴스를 생성한다.
        # instance=task -> 기존의 task내용을 가져와서 수정할 수 있게 해준다.
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task) # GET 요청일때는 task 인스턴스를 사용해 TaskForm 인스턴스를 생성한다.
    return render(request, 'todoapp/edit_task.html', {'form': form, 'task': task}) # 폼과 task 인스턴스를 포함한 edit_task.html 템플릿을 렌더링한다.

def delete_task(request, task_id):
    task = get_object_or_404(Task, id=task_id)
    if request.method == 'POST':
        task.delete()
        return redirect('task_list')
    return render(request, 'todoapp/delete_task.html', {'task': task})
'''

# todoapp/templates/todoapp/edit_task.html 작성
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>할 일 수정</title>
</head>
<body>
    <h1>할 일 수정</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">수정</button>
    </form>
    <a href="{% url 'task_list' %}">목록으로 돌아가기</a>
</body>
</html>
'''

# todoapp/templates/todoapp/delete_task.html 작성
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>할 일 삭제</title>
</head>
<body>
    <h1>할 일 삭제</h1>
    <p>정말로 "{{ task.title }}" 할 일을 삭제하시겠습니까?</p>
    <form method="post">
        {% csrf_token %}
        <button type="submit">예, 삭제합니다</button>
    </form>
    <a href="{% url 'task_list' %}">아니오, 목록으로 돌아갑니다</a>
</body>
</html>
'''

# todoapp/templates/todoapp/task_list.html 수정(전체 복붙 가능)
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>할 일 목록</title>
    <style>
        .completed {
            text-decoration: line-through;
            color: gray;
        }
    </style>
</head>
<body>
    <h1>할 일 목록</h1>
    <a href="{% url 'add_task' %}">새 할 일 추가</a>
    <ul>
        {% for task in tasks %}
            <li class="{% if task.completed %}completed{% endif %}">
                {{ task.title }}
                {% if task.due_date %}
                    (마감일: {{ task.due_date|date:"Y-m-d" }})
                {% endif %}
                <a href="{% url 'edit_task' task.id %}">수정</a>
                <a href="{% url 'delete_task' task.id %}">삭제</a>
            </li>
        {% empty %}
            <li>할 일이 없습니다.</li>
        {% endfor %}
    </ul>
</body>
</html>
'''

# todoapp/admin.py 작성(전체 복붙 가능)
'''
from django.contrib import admin
from .models import List, Task

admin.site.register(List)
admin.site.register(Task)
'''

# 슈퍼 유저 생성
'''
python manage.py createsuperuser
'''

# todoapp/forms 수정(전체 복붙 가능)
'''
from django import forms
from .models import Task, List

class TaskForm(forms.ModelForm):
    list = forms.ModelChoiceField(queryset=List.objects.all(), required=False)

    class Meta:
        model = Task
        fields = ['title', 'description', 'due_date', 'list']
        widgets = {
            'due_date': forms.DateTimeInput(attrs={'type': 'datetime-local'}),
        }

'''

# todoapp/views.py 수정(전체 복붙 가능)
'''
from django.shortcuts import render, redirect, get_object_or_404
from .models import Task, List
from .forms import TaskForm

def task_list(request):
    tasks = Task.objects.all().order_by('-created_date')
    lists = List.objects.all()
    return render(request, 'todoapp/task_list.html', {'tasks': tasks, 'lists': lists}) # task_list.html 템플릿을 렌더링한다.


def add_task(request):
    if request.method == 'POST': # 폼이 제출 되었을때 POST 요청을 처리한다. 처리가됨
        form = TaskForm(request.POST) # POST 요청으로 전달된 데이터를 사용해 TaskForm 인스턴스를 생성한다.
        if form.is_valid(): # form의 데이터가 유효한지 검사 -> 모델/제약조건을 기반으로 자동 유효성 검증을 수행한다.
            form.save() # form의 데이터를 데이터베이스에 저장한다.
            return redirect('task_list') # task_list 뷰로 이동한다.
    else:
        form = TaskForm() # GET 요청일때는 빈 TaskForm 인스턴스를 생성한다.
    lists = List.objects.all()
    return render(request, 'todoapp/add_task.html', {'form': form, 'lists':lists}) # 폼을 포함한 add_task.html 템플릿을 렌더링한다.

def edit_task(request, task_id):
    task = get_object_or_404(Task, id=task_id) # pk가 주어진 Task 인스턴스를 가져온다. 없으면 404 에러를 발생시킨다.
    if request.method == 'POST':
        form = TaskForm(request.POST, instance=task) # POST 요청으로 전달된 데이터와 task 인스턴스를 사용해 TaskForm 인스턴스를 생성한다.
        # instance=task -> 기존의 task내용을 가져와서 수정할 수 있게 해준다.
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task) # GET 요청일때는 task 인스턴스를 사용해 TaskForm 인스턴스를 생성한다.
    return render(request, 'todoapp/edit_task.html', {'form': form, 'task': task}) # 폼과 task 인스턴스를 포함한 edit_task.html 템플릿을 렌더링한다.

def delete_task(request, task_id):
    task = get_object_or_404(Task, id=task_id)
    if request.method == 'POST':
        task.delete()
        return redirect('task_list')
    return render(request, 'todoapp/delete_task.html', {'task': task})
'''

# todoapp/templates/todoapp/task_list.html 수정(전체 복붙 가능)
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>할 일 목록</title>
    <style>
        .completed {
            text-decoration: line-through;
            color: gray;
        }
    </style>
</head>
<body>
    <h1>할 일 목록</h1>
    <a href="{% url 'add_task' %}">새 할 일 추가</a>
    <ul>
        {% for task in tasks %}
            <li class="{% if task.completed %}completed{% endif %}">
                {{ task.title }}
                {% if task.due_date %}
                    (마감일: {{ task.due_date|date:"Y-m-d" }})
                {% endif %}
                <a href="{% url 'edit_task' task.id %}">수정</a>
                <a href="{% url 'delete_task' task.id %}">삭제</a>
            </li>
        {% empty %}
            <li>할 일이 없습니다.</li>
        {% endfor %}
    </ul>
    <!--기존 내용 유지-->
    <h2>할 일 목록</h2>
{% for list in lists %}
    <h3>{{ list.name }}</h3>
    <ul>
        {% for task in tasks %}
            {% if task.list == list %}
                <li>{{ task.title }} - {{ task.due_date|date:"Y-m-d" }}</li>
            {% endif %}
        {% endfor %}
    </ul>
{% endfor %}
</body>
</html>
'''

# todoapp/models.py 수정 (전체 복붙 가능)
'''
from django.db import models

# Create your models here.

class List(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)

    def __str__(self):
        return self.name


class Task(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    created_date = models.DateTimeField(auto_now_add=True)
    due_date = models.DateTimeField(null=True, blank=True)
    completed = models.BooleanField(default=False)
    list = models.ForeignKey(List, on_delete=models.CASCADE, related_name='tasks', null=True, blank=True)

    def __str__(self):
        return self.title
'''

# 마이그레이션
'''
python manage.py makemigrations
python manage.py migrate
'''

# todoapp/views.py 수정(전체 복붙 가능)
'''
from django.shortcuts import render, redirect, get_object_or_404
from .models import Task, List
from .forms import TaskForm

def task_list(request):
    tasks = Task.objects.all().order_by('-created_date')
    lists = List.objects.all()
    return render(request, 'todoapp/task_list.html', {'tasks': tasks, 'lists': lists}) # task_list.html 템플릿을 렌더링한다.


def add_task(request):
    if request.method == 'POST': # 폼이 제출 되었을때 POST 요청을 처리한다. 처리가됨
        form = TaskForm(request.POST) # POST 요청으로 전달된 데이터를 사용해 TaskForm 인스턴스를 생성한다.
        if form.is_valid(): # form의 데이터가 유효한지 검사 -> 모델/제약조건을 기반으로 자동 유효성 검증을 수행한다.
            task = form.save(commit=False) # form의 데이터를 데이터베이스에 저장한다.
            if not task.list:
                default_list, created = List.objects.get_or_create(name="기본 목록")
                task.list = default_list
            task.save()
            return redirect('task_list') # task_list 뷰로 이동한다.
    else:
        form = TaskForm() # GET 요청일때는 빈 TaskForm 인스턴스를 생성한다.
    lists = List.objects.all()
    return render(request, 'todoapp/add_task.html', {'form': form, 'lists':lists}) # 폼을 포함한 add_task.html 템플릿을 렌더링한다.


def edit_task(request, task_id):
    task = get_object_or_404(Task, id=task_id) # pk가 주어진 Task 인스턴스를 가져온다. 없으면 404 에러를 발생시킨다.
    if request.method == 'POST':
        form = TaskForm(request.POST, instance=task) # POST 요청으로 전달된 데이터와 task 인스턴스를 사용해 TaskForm 인스턴스를 생성한다.
        # instance=task -> 기존의 task내용을 가져와서 수정할 수 있게 해준다.
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task) # GET 요청일때는 task 인스턴스를 사용해 TaskForm 인스턴스를 생성한다.
    return render(request, 'todoapp/edit_task.html', {'form': form, 'task': task}) # 폼과 task 인스턴스를 포함한 edit_task.html 템플릿을 렌더링한다.

def delete_task(request, task_id):
    task = get_object_or_404(Task, id=task_id)
    if request.method == 'POST':
        task.delete()
        return redirect('task_list')
    return render(request, 'todoapp/delete_task.html', {'task': task})
'''

# todoapp/forms.py 수정(전체 복붙 가능)
'''
from django import forms
from .models import Task, List

class TaskForm(forms.ModelForm):
    list = forms.ModelChoiceField(queryset=List.objects.all(), empty_label=None)

    class Meta:
        model = Task
        fields = ['title', 'description', 'due_date', 'list']
        widgets = {
            'due_date': forms.DateTimeInput(attrs={'type': 'datetime-local'}),
        }

'''

# todoapp/templates/todoapp/add_task.html 수정(전체 복붙 가능)
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>할 일 추가</title>
</head>
<body>
    <h1>할 일 추가</h1>
    <form method="post">
        {% csrf_token %}
        {% for field in form %}
            <p>
                {{ field.label_tag }}
                {% if field.name == 'list' %}
                    <select name="list" required>
                        <option value="">---------</option>
                        {% for list in lists %}
                            <option value="{{ list.id }}">{{ list.name }}</option>
                        {% endfor %}
                    </select>
                {% else %}
                    {{ field }}
                {% endif %}
            </p>
        {% endfor %}
        <button type="submit">추가</button>
    </form>
    <a href="{% url 'task_list' %}">목록으로 돌아가기</a>
</body>
</html>
'''


################### 추가 기능 1 ########################


# todoapp/models.py 추가(전체 복붙 가능)
'''
from django.db import models

# Create your models here.

class List(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)

    def __str__(self):
        return self.name


class Task(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    created_date = models.DateTimeField(auto_now_add=True)
    due_date = models.DateTimeField(null=True, blank=True)
    completed = models.BooleanField(default=False)
    list = models.ForeignKey(List, on_delete=models.CASCADE, related_name='tasks', null=True, blank=True)

    #우선순위 model
    PRIORITY_CHOICES = [
        (1, '낮음'),
        (2, '중간'),
        (3, '높음'),        
    ]
    priority = models.IntegerField(choices=PRIORITY_CHOICES, default=2)

    def __str__(self):
        return f"{self.title} (우선순위: {self.get_priority_display()})"
'''

# todoapp/views.py 수정(전체 복붙 가능)
'''
from django.shortcuts import render, redirect, get_object_or_404
from .models import Task, List
from .forms import TaskForm

def task_list(request):
    tasks = Task.objects.all().order_by('-created_date')
    lists = List.objects.all()
    return render(request, 'todoapp/task_list.html', {'tasks': tasks, 'lists': lists}) # task_list.html 템플릿을 렌더링한다.


def add_task(request):
    if request.method == 'POST': # 폼이 제출 되었을때 POST 요청을 처리한다. 처리가됨
        form = TaskForm(request.POST) # POST 요청으로 전달된 데이터를 사용해 TaskForm 인스턴스를 생성한다.
        if form.is_valid(): # form의 데이터가 유효한지 검사 -> 모델/제약조건을 기반으로 자동 유효성 검증을 수행한다.
            task = form.save(commit=False) # form의 데이터를 데이터베이스에 저장한다.
            if not task.list:
                default_list, created = List.objects.get_or_create(name="기본 목록")
                task.list = default_list
            task.save()
            return redirect('task_list') # task_list 뷰로 이동한다.
    else:
        form = TaskForm() # GET 요청일때는 빈 TaskForm 인스턴스를 생성한다.
    lists = List.objects.all()
    return render(request, 'todoapp/add_task.html', {'form': form, 'lists':lists}) # 폼을 포함한 add_task.html 템플릿을 렌더링한다.



''' 백업
def add_task(request):
    if request.method == 'POST': # 폼이 제출 되었을때 POST 요청을 처리한다. 처리가됨
        form = TaskForm(request.POST) # POST 요청으로 전달된 데이터를 사용해 TaskForm 인스턴스를 생성한다.
        if form.is_valid(): # form의 데이터가 유효한지 검사 -> 모델/제약조건을 기반으로 자동 유효성 검증을 수행한다.
            form.save() # form의 데이터를 데이터베이스에 저장한다.
            return redirect('task_list') # task_list 뷰로 이동한다.
    else:
        form = TaskForm() # GET 요청일때는 빈 TaskForm 인스턴스를 생성한다.
    lists = List.objects.all()
    return render(request, 'todoapp/add_task.html', {'form': form, 'lists':lists}) # 폼을 포함한 add_task.html 템플릿을 렌더링한다.
'''


def edit_task(request, task_id):
    task = get_object_or_404(Task, id=task_id) # pk가 주어진 Task 인스턴스를 가져온다. 없으면 404 에러를 발생시킨다.
    if request.method == 'POST':
        form = TaskForm(request.POST, instance=task) # POST 요청으로 전달된 데이터와 task 인스턴스를 사용해 TaskForm 인스턴스를 생성한다.
        # instance=task -> 기존의 task내용을 가져와서 수정할 수 있게 해준다.
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task) # GET 요청일때는 task 인스턴스를 사용해 TaskForm 인스턴스를 생성한다.
    return render(request, 'todoapp/edit_task.html', {'form': form, 'task': task}) # 폼과 task 인스턴스를 포함한 edit_task.html 템플릿을 렌더링한다.

def delete_task(request, task_id):
    task = get_object_or_404(Task, id=task_id)
    if request.method == 'POST':
        task.delete()
        return redirect('task_list')
    return render(request, 'todoapp/delete_task.html', {'task': task})

def set_priority(request, task_id):
    task = get_object_or_404(Task, id=task_id)
    if request.method == 'POST':
        priority = request.POST.get('priority')
        if priority in ['1', '2', '3']:
            task.priority = int(priority)
            task.save()
    return redirect('task_list') # task_list 뷰로 리다이렉트한다.
'''


# todoapp/templates/todoapp/task_list.html 수정(전체 복붙 가능)
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <!-- 기존 내용 유지 -->
    <style>
        .completed { text-decoration: line-through; color: gray; }
        .priority-high { color: red; }
        .priority-medium { color: orange; }
        .priority-low { color: green; }
    </style>
</head>
<body>
    <h1>할 일 목록</h1>
    <a href="{% url 'add_task' %}">새 할 일 추가</a>
    <ul>
        {% for task in tasks %}
            <li class="{% if task.completed %}completed{% endif %} priority-{{ task.get_priority_display|lower }}">
                {{ task.title }} - 우선순위: {{ task.get_priority_display }}
                {% if task.due_date %}
                    (마감일: {{ task.due_date|date:"Y-m-d" }})
                {% endif %}
                <form method="post" action="{% url 'set_priority' task.id %}" style="display: inline;">
                    {% csrf_token %}
                    <select name="priority" onchange="this.form.submit()">
                        <option value="1" {% if task.priority == 1 %}selected{% endif %}>낮음</option>
                        <option value="2" {% if task.priority == 2 %}selected{% endif %}>중간</option>
                        <option value="3" {% if task.priority == 3 %}selected{% endif %}>높음</option>
                    </select>
                </form>
                <a href="{% url 'edit_task' task.id %}">수정</a>
                <a href="{% url 'delete_task' task.id %}">삭제</a>
            </li>
        {% empty %}
            <li>할 일이 없습니다.</li>
        {% endfor %}
    </ul>
</body>
</html>
'''

# todoapp/urls.py 수정(전체 복붙 가능)
'''
from django.urls import path
from . import views

urlpatterns = [
    path('', views.task_list, name='task_list'),
    path('add/', views.add_task, name='add_task'),
    path('edit/<int:task_id>/', views.edit_task, name='edit_task'),
    path('delete/<int:task_id>/', views.delete_task, name='delete_task'),
    path('set_priority/<int:task_id>/', views.set_priority, name='set_priority'),
]
'''

# 마이그레이션
'''
python manage.py makemigrations
python manage.py migrate
'''

# 서버 실행
'''
python manage.py runserver
'''


#################### 통계 기능 ################


# todoapp/views.py 수정(전체 복붙 가능)
'''
from django.shortcuts import render, redirect, get_object_or_404
from .models import Task, List
from .forms import TaskForm
from django.db.models import Count

def task_list(request):
    tasks = Task.objects.all().order_by('-created_date')
    lists = List.objects.all()
    return render(request, 'todoapp/task_list.html', {'tasks': tasks, 'lists': lists}) # task_list.html 템플릿을 렌더링한다.


def add_task(request):
    if request.method == 'POST': # 폼이 제출 되었을때 POST 요청을 처리한다. 처리가됨
        form = TaskForm(request.POST) # POST 요청으로 전달된 데이터를 사용해 TaskForm 인스턴스를 생성한다.
        if form.is_valid(): # form의 데이터가 유효한지 검사 -> 모델/제약조건을 기반으로 자동 유효성 검증을 수행한다.
            task = form.save(commit=False) # form의 데이터를 데이터베이스에 저장한다.
            if not task.list:
                default_list, created = List.objects.get_or_create(name="기본 목록")
                task.list = default_list
            task.save()
            return redirect('task_list') # task_list 뷰로 이동한다.
    else:
        form = TaskForm() # GET 요청일때는 빈 TaskForm 인스턴스를 생성한다.
    lists = List.objects.all()
    return render(request, 'todoapp/add_task.html', {'form': form, 'lists':lists}) # 폼을 포함한 add_task.html 템플릿을 렌더링한다.


def edit_task(request, task_id):
    task = get_object_or_404(Task, id=task_id) # pk가 주어진 Task 인스턴스를 가져온다. 없으면 404 에러를 발생시킨다.
    if request.method == 'POST':
        form = TaskForm(request.POST, instance=task) # POST 요청으로 전달된 데이터와 task 인스턴스를 사용해 TaskForm 인스턴스를 생성한다.
        # instance=task -> 기존의 task내용을 가져와서 수정할 수 있게 해준다.
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task) # GET 요청일때는 task 인스턴스를 사용해 TaskForm 인스턴스를 생성한다.
    return render(request, 'todoapp/edit_task.html', {'form': form, 'task': task}) # 폼과 task 인스턴스를 포함한 edit_task.html 템플릿을 렌더링한다.

def delete_task(request, task_id):
    task = get_object_or_404(Task, id=task_id)
    if request.method == 'POST':
        task.delete()
        return redirect('task_list')
    return render(request, 'todoapp/delete_task.html', {'task': task})

def set_priority(request, task_id):
    task = get_object_or_404(Task, id=task_id)
    if request.method == 'POST':
        priority = request.POST.get('priority')
        if priority in ['1', '2', '3']:
            task.priority = int(priority)
            task.save()
    return redirect('task_list') # task_list 뷰로 리다이렉트한다.

def task_statistics(request):
    tasks = Task.objects.all()

    # 통계 계산
    statistics = {
        'total_tasks': tasks.count(),
        'completed_tasks': tasks.filter(completed=True).count(),
        'incomplete_tasks': tasks.filter(completed=False).count(),
        'high_priority_tasks': tasks.filter(priority=3).count(),
        'medium_priority_tasks': tasks.filter(priority=2).count(),
        'low_priority_tasks': tasks.filter(priority=1).count(),
    }

    # 우선순위별 완료율 계산
    for priority in [1, 2, 3]:
        priority_tasks = tasks.filter(priority=priority)
        total = priority_tasks.count()
        completed = priority_tasks.filter(completed=True).count()
        statistics[f'priority_{priority}_completion_rate'] = (completed / total * 100 if total > 0 else 0)

    return render(request, 'todoapp/task_statistics.html', {'statistics': statistics})
'''

# todoapp/urls.py 수정(전체 복붙 가능)
'''
from django.urls import path
from . import views

urlpatterns = [
    path('', views.task_list, name='task_list'),
    path('add/', views.add_task, name='add_task'),
    path('edit/<int:task_id>/', views.edit_task, name='edit_task'),
    path('delete/<int:task_id>/', views.delete_task, name='delete_task'),
    path('set_priority/<int:task_id>/', views.set_priority, name='set_priority'),
    path('statistics/', views.task_statistics, name='task_statistics'),
]
'''

# todoapp/templates/todoapp/task_statistics.html 작성
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>작업 통계</title>
    <style>
        .statistics { margin-bottom: 20px; }
        .stat-item { margin-bottom: 10px; }
    </style>
</head>
<body>
    <h1>작업 통계</h1>
    <div class="statistics">
        <div class="stat-item">전체 작업: {{ statistics.total_tasks }}</div>
        <div class="stat-item">완료된 작업: {{ statistics.completed_tasks }}</div>
        <div class="stat-item">미완료 작업: {{ statistics.incomplete_tasks }}</div>
        <div class="stat-item">높은 우선순위 작업: {{ statistics.high_priority_tasks }}</div>
        <div class="stat-item">중간 우선순위 작업: {{ statistics.medium_priority_tasks }}</div>
        <div class="stat-item">낮은 우선순위 작업: {{ statistics.low_priority_tasks }}</div>
    </div>
    <h2>우선순위별 완료율</h2>
    <div class="statistics">
        <div class="stat-item">높은 우선순위 완료율: {{ statistics.priority_3_completion_rate|floatformat:2 }}%</div>
        <div class="stat-item">중간 우선순위 완료율: {{ statistics.priority_2_completion_rate|floatformat:2 }}%</div>
        <div class="stat-item">낮은 우선순위 완료율: {{ statistics.priority_1_completion_rate|floatformat:2 }}%</div>
    </div>
    <a href="{% url 'task_list' %}">작업 목록으로 돌아가기</a>
</body>
</html>
'''

# todoapp/templates/todoapp/task_list.html에 통계 보기 링크 추가
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>할 일 목록</title>
    <style>
        .completed { text-decoration: line-through; color: gray; }
        .priority-high { color: red; }
        .priority-medium { color: orange; }
        .priority-low { color: green; }
    </style>
</head>
<body>
    <h1>할 일 목록</h1>
    <a href="{% url 'add_task' %}">새 할 일 추가</a>
    <a href="{% url 'task_statistics' %}">작업 통계 보기</a>
    <ul>
        {% for task in tasks %}
        <li class="{% if task.completed %}completed{% endif %} priority-{{ task.get_priority_display|lower }}">
            {{ task.title }} - 우선순위: {{ task.get_priority_display }}
            {% if task.due_date %}
                (마감일: {{ task.due_date|date:"Y-m-d" }})
            {% endif %}
            <form method="post" action="{% url 'set_priority' task.id %}" style="display: inline;">
                {% csrf_token %}
                <select name="priority" onchange="this.form.submit()">
                    <option value="1" {% if task.priority == 1 %}selected{% endif %}>낮음</option>
                    <option value="2" {% if task.priority == 2 %}selected{% endif %}>중간</option>
                    <option value="3" {% if task.priority == 3 %}selected{% endif %}>높음</option>
                </select>
            </form>
                <a href="{% url 'edit_task' task.id %}">수정</a>
                <a href="{% url 'delete_task' task.id %}">삭제</a>
            </li>
        {% empty %}
            <li>할 일이 없습니다.</li>
        {% endfor %}
    </ul>
    <!--기존 내용 유지-->
    <h2>할 일 목록</h2>
{% for list in lists %}
    <h3>{{ list.name }}</h3>
    <ul>
        {% for task in tasks %}
            {% if task.list == list %}
                <li>{{ task.title }} - {{ task.due_date|date:"Y-m-d" }}</li>
            {% endif %}
        {% endfor %}
    </ul>
{% endfor %}
</body>
</html>
'''


```