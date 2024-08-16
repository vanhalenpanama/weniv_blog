''' git config core.autocrlf true '''

{% raw %}

```python

# 설정
'''
python -m venv myenv

myenv\\Scripts\\activate


pip install django djangorestframework


django-admin startproject myproject


cd myproject
python manage.py startapp myapp

'''

# myproject/settings.py INSTALLED_APPS 부분 수정
'''
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'myapp',
]
'''


# 프로젝트 구조
'''
myproject/
│
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
│
├── myapp/
│   ├── migrations/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── serializers.py   # DRF 프로젝트에서는 주로 추가되는 파일
│   ├── views.py
│   └── urls.py
│
└── manage.py
'''


# myapp/views.py 수정

'''
# 이 부분은 Rest_framework의 기본 뼈대를 만드는 부분 -> 이 클래스를 상속받아서 HTTP 메서드(Get, Post, Put, Delete)를 구현할 수 있다.
from rest_framework.views import APIView
# API 뷰에서 반환하는 응답객체 -> 딕셔너리를 -> JSON으로 변환하여 반환한다. 
# 클라이언트에게 전달할 메세지를 정해진 양식 (표준화된 API 형식)에 따라 데이터 전달하는 방법
from rest_framework.response import Response

class HelloAPIView(APIView):
    def get(self, request):
        return Response({"message": "Hello, World!"})
'''

# myapp/urls.py 생성

'''
from django.urls import path
from .views import HelloAPIView

urlpatterns = [
    path("hello/", HelloAPIView.as_view(), name = 'hello-api'), # API 뷰를 URL에 연결한다. -> API 뷰를 호출한다.
]
'''


# myproject/urls.py 수정

'''
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('myapp.urls')),  # 'myapp'의 URL을 포함시킴
]
'''

# 마이그레이트 실행

'''
python manage.py migrate

python manage.py runserver
'''

# 3. 영화 정보 API 예제로 Django Rest Framework 기초 개념 살펴보기 #####################


# myapp/models.py 수정

'''
from django.db import models

class Movie(models.Model):
    title = models.CharField(max_length=100)
    genre = models.CharField(max_length=50)
    year = models.IntegerField()

    def __str__(self):
        return self.title
'''


# myapp/serializers.py 생성

'''
from rest_framework import serializers
from .models import Movie

class MovieSerializer(serializers.ModelSerializer):
    class Meta:
        model = Movie
        fields = '__all__'
'''

# myapp/views.py 수정

'''
# 이 부분은 Rest_framework의 기본 뼈대를 만드는 부분 -> 이 클래스를 상속받아서 HTTP 메서드(Get, Post, Put, Delete)를 구현할 수 있다.
from rest_framework.views import APIView
# API 뷰에서 반환하는 응답객체 -> 딕셔너리를 -> JSON으로 변환하여 반환한다. 
# 클라이언트에게 전달할 메세지를 정해진 양식 (표준화된 API 형식)에 따라 데이터 전달하는 방법
from rest_framework.response import Response
from rest_framework import status
from .models import Movie
from .serializers import MovieSerializer

# CBV
# class HelloAPIView(APIView):
#     def get(self, request):
#         return Response({"message": "Hello, World!"})

class MovieListAPIView(APIView):
    def get(self, request):
        movies = Movie.objects.all()
        serializer = MovieSerializer(movies, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = MovieSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class MovieDetailAPIView(APIView):
    def get_object(self, pk):
        try:
            return Movie.objects.get(pk=pk)
        except Movie.DoesNotExist:
            return Response(status=status.HTTP_404_NOT_FOUND)

    def get(self, request, pk):
        movie = self.get_object(pk)
        serializer = MovieSerializer(movie)
        return Response(serializer.data)

    def put(self, request, pk):
        movie = self.get_object(pk)
        serializer = MovieSerializer(movie, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk):
        movie = self.get_object(pk)
        movie.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
'''

# myapp/urls.py 수정

'''
from django.urls import path
# from .views import HelloAPIView
from .views import MovieListAPIView, MovieDetailAPIView

urlpatterns = [
    path('movies/', MovieListAPIView.as_view(), name='movie-list'),  # 클래스 기반 뷰 (CBV)
    path('movies/<int:pk>/', MovieDetailAPIView.as_view(), name='movie-detail'),  # 영화 세부 정보 조회, 수정, 삭제
]
'''


# 관리자 페이지 설정 (선택 사항) myapp/admin.py 수정

'''
from django.contrib import admin 
from .models import Movie

admin.site.register(Movie)
'''

# 마이그레이션 
'''
python manage.py makemigrations
python manage.py migrate
'''

# 서버 실행 후 다음 url에서 확인
'''
http://127.0.0.1:8000/api/movies 접속


# 아래 데이터를 post
"""
{
  "title": "The Matrix Movie", 
  "genre": "Action",
  "year": 1999
}
"""

http://127.0.0.1:8000/api/movies/1에서 확인

'''

# 4. Django REST Framework 심화 개념 ###########


# myapp/views.py 수정(CBV 삭제)

'''
# 이 부분은 Rest_framework의 기본 뼈대를 만드는 부분 -> 이 클래스를 상속받아서 HTTP 메서드(Get, Post, Put, Delete)를 구현할 수 있다.
from rest_framework.views import APIView
# API 뷰에서 반환하는 응답객체 -> 딕셔너리를 -> JSON으로 변환하여 반환한다. 
# 클라이언트에게 전달할 메세지를 정해진 양식 (표준화된 API 형식)에 따라 데이터 전달하는 방법
from rest_framework.response import Response
from rest_framework import status,viewsets
from .models import Movie
from .serializers import MovieSerializer
from django.shortcuts import get_object_or_404 


# viewset

class MovieViewSet(viewsets.ModelViewSet):
    queryset = Movie.objects.all()
    serializer_class = MovieSerializer
'''


# myapp/urls.py 수정

'''
from django.urls import path,include
# from .views import HelloAPIView
from rest_framework.routers import DefaultRouter
from .views import MovieViewSet

router = DefaultRouter()
router.register("movies", MovieViewSet)

urlpatterns = [
    path("", include(router.urls)),
]
'''

####################### 폴더 생성 후 새로 시작#########

# 설정
'''
python -m venv myenv
myenv\\Scripts\\activate
pip install django
django-admin startproject todo_project
cd todo_project
python manage.py startapp todo

#settings.py
INSTALLED_APPS = [
    ...
    'rest_framework',  # Django REST Framework 추가
    'todo',
]
''' 

# models.py 수정
'''
from django.db import models

class Todo(models.Model):
    title = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    is_completed = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
'''

# 마이그레이션
'''
python manage.py makemigrations
python manage.py migrate
'''

# 2. Todo 전체 조회 API 만들기 ##################

# Django DRF 설치
'''
pip install djangorestframework
'''


# todo/serializers.py 생성
'''
from rest_framework import serializers
from .models import Todo

class TodoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Todo
        fields = '__all__'
'''

# todo/views.py 수정

'''
from rest_framework import generics
from .models import Todo
from .serializers import TodoSerializer

class TodoListView(generics.ListAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
'''

# todo/urls.py 생성

'''
from django.urls import path
from .views import TodoListView

urlpatterns = [
    path('todos/', TodoListView.as_view(), name='todo-list'),
]
'''

# todo_project/urls.py 생성

'''
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('todo.urls')),  # 'todo' 앱의 URL 포함
]
'''


# 정상 출력 확인

'''
python manage.py runserver

http://127.0.0.1:8000/api/todos/
'''

# 3. Todo 상세 조회 API 만들기 ###################

# todo/views.py 수정
'''
from rest_framework import generics
from .models import Todo
from .serializers import TodoSerializer

class TodoListView(generics.ListAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer


class TodoDetailView(generics.RetrieveAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'
'''


# todo/urls.py 수정
'''
from django.urls import path
from .views import TodoListView, TodoDetailView

urlpatterns = [
    path('todos/', TodoListView.as_view(), name='todo-list'),
    path('todos/<int:pk>/', TodoDetailView.as_view(), name='todo-detail'),
]
'''

# 테스트 

'''
python manage.py runserver


http://127.0.0.1:8000/api/todos/1/
'''

# Django 쉘에서 Todo 항목 생성하기

'''
python manage.py shell


from todo.models import Todo
new_todo = Todo(title="New Task", description="This is a new task to be completed", is_completed=False)
new_todo.save()
all_todos = Todo.objects.all()
print(all_todos)


exit()
'''

# 4. Todo 생성 API 만들기 ##################

# todo/views.py 수정
'''
from rest_framework import generics
from .models import Todo
from .serializers import TodoSerializer

class TodoListView(generics.ListAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer


class TodoDetailView(generics.RetrieveAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'
    
class TodoCreateView(generics.CreateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
'''


# todo/urls.py 수정

'''
from django.urls import path
from .views import TodoListView, TodoDetailView, TodoCreateView

urlpatterns = [
    path('todos/', TodoListView.as_view(), name='todo-list'),
    path('todos/<int:pk>/', TodoDetailView.as_view(), name='todo-detail'),
    path('todos/create/', TodoCreateView.as_view(), name='todo-create'),
]
'''

# 테스트 
'''
python manage.py runserver

{
    "title": "Learn Django",
    "description": "Complete the Django REST Framework tutorial",
    "is_completed": false
}

http://127.0.0.1:8000/api/todos/create/

'''

# 5. Todo 수정 API 만들기 ####################

# todo/views.py 수정
'''
from rest_framework import generics
from .models import Todo
from .serializers import TodoSerializer

class TodoListView(generics.ListAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer


class TodoDetailView(generics.RetrieveAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'
    
class TodoCreateView(generics.CreateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer    

class TodoUpdateView(generics.UpdateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'    
'''

# todo/urls.py 수정
'''
from django.urls import path
from .views import TodoListView, TodoDetailView, TodoCreateView, TodoUpdateView

urlpatterns = [
    path('todos/', TodoListView.as_view(), name='todo-list'),
    path('todos/<int:pk>/', TodoDetailView.as_view(), name='todo-detail'),
    path('todos/create/', TodoCreateView.as_view(), name='todo-create'),
    path('todos/<int:pk>/update/', TodoUpdateView.as_view(), name='todo-update'),
]
'''

# 수정 테스트 

'''
python manage.py runserver


{
    "title": "Learn Django Updated",
    "description": "Complete the Django REST Framework tutorial with updates",
    "is_completed": true
}

http://127.0.0.1:8000/api/todos/1/update/
'''

# 참고: PUT 또는 PATCH 의 차이점?
'''
전체 업데이트: PUT 요청은 리소스를 전체적으로 업데이트할 때 사용됩니다. 즉, 서버의 리소스를 새로운 데이터로 완전히 대체합니다.


부분 업데이트: PATCH 요청은 리소스를 부분적으로 업데이트할 때 사용됩니다. 즉, 서버의 리소스 중에서 일부 필드만 수정할 때 사용됩니다.
'''


# 6. Todo 완료 API 만들기 #######################

# todo/views.py 수정
'''
from rest_framework import generics,status
from .models import Todo
from .serializers import TodoSerializer
from rest_framework.response import Response
from rest_framework.views import APIView


class TodoListView(generics.ListAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer


class TodoDetailView(generics.RetrieveAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'
    
class TodoCreateView(generics.CreateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer    

class TodoUpdateView(generics.UpdateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'    
    
    
class TodoCompleteView(APIView):
    def post(self, request, pk):
        try:
            todo = Todo.objects.get(pk=pk)
            todo.is_completed = True
            todo.save()
            serializer = TodoSerializer(todo)
            return Response(serializer.data, status=status.HTTP_200_OK)
        except Todo.DoesNotExist:
            return Response(status=status.HTTP_404_NOT_FOUND)   
'''

# todo/urls.py 수정
'''
from django.urls import path
from .views import TodoListView, TodoDetailView, TodoCreateView, TodoUpdateView, TodoCompleteView

urlpatterns = [
    path('todos/', TodoListView.as_view(), name='todo-list'),
    path('todos/<int:pk>/', TodoDetailView.as_view(), name='todo-detail'),
    path('todos/create/', TodoCreateView.as_view(), name='todo-create'),
    path('todos/<int:pk>/update/', TodoUpdateView.as_view(), name='todo-update'),
    path('todos/<int:pk>/complete/', TodoCompleteView.as_view(), name='todo-complete'),
]
'''


# 테스트

'''
python manage.py runserver


http://127.0.0.1:8000/api/todos/1/complete/
'''

# 7. Todo 삭제 API 만들기 ###############

# todo/views.py 수정
'''
from rest_framework import generics,status
from .models import Todo
from .serializers import TodoSerializer
from rest_framework.response import Response
from rest_framework.views import APIView


class TodoListView(generics.ListAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer


class TodoDetailView(generics.RetrieveAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'
    
class TodoCreateView(generics.CreateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer    

class TodoUpdateView(generics.UpdateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'    
    
    
class TodoCompleteView(APIView):
    def post(self, request, pk):
        try:
            todo = Todo.objects.get(pk=pk)
            todo.is_completed = True
            todo.save()
            serializer = TodoSerializer(todo)
            return Response(serializer.data, status=status.HTTP_200_OK)
        except Todo.DoesNotExist:
            return Response(status=status.HTTP_404_NOT_FOUND)
        
class TodoDeleteView(generics.DestroyAPIView):
    queryset = Todo.objects.all()
    lookup_field = 'pk'  
'''

# todo/urls.py 수정
'''
from django.urls import path
from .views import TodoListView, TodoDetailView, TodoCreateView, TodoUpdateView, TodoCompleteView, TodoDeleteView

urlpatterns = [
    path('todos/', TodoListView.as_view(), name='todo-list'),
    path('todos/<int:pk>/', TodoDetailView.as_view(), name='todo-detail'),
    path('todos/create/', TodoCreateView.as_view(), name='todo-create'),
    path('todos/<int:pk>/update/', TodoUpdateView.as_view(), name='todo-update'),
    path('todos/<int:pk>/complete/', TodoCompleteView.as_view(), name='todo-complete'),
    path('todos/<int:pk>/delete/', TodoDeleteView.as_view(), name='todo-delete'),
]
'''

# 테스트
'''
python manage.py runserver


http://127.0.0.1:8000/api/todos/1/delete/

# delete 후 todo list에서 삭제 확인 
'''


# 8. 백엔드 구현 ###############################
'''
todo/
    management/
        commands/
            __init__.py
            insert_initial_data.py
'''




# todo/management/commands/insert_initial_data.py 생성
'''
from django.core.management.base import BaseCommand
from todo.models import Todo

class Command(BaseCommand):
    help = 'Insert initial data into the Todo model'

    def handle(self, *args, **kwargs):
        Todo.objects.create(title='Learn Django', description='Start with Django basics', is_completed=False)
        Todo.objects.create(title='Build API', description='Create a REST API with Django', is_completed=False)
        self.stdout.write(self.style.SUCCESS('Successfully inserted initial data into the Todo model'))
'''

# python manage.py insert_initial_data 실행
'''
python manage.py insert_initial_data
'''

# 코드 정리 #############################################

todo_project/urls.py
'''
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('todo.urls')),  # 'todo' 앱의 URL 포함
    path('', include('todo.urls')),  # 웹 페이지 관련 URL 패턴
]
'''


# 9. 프론트 작업 ###############################

# todo/templates/base.html 생성
'''
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Todo List{% endblock %}</title>
    <link href="<https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css>" rel="stylesheet">
</head>
<body>
    <div class="container">
        {% block content %}
        {% endblock %}
    </div>

    <script src="<https://code.jquery.com/jquery-3.5.1.slim.min.js>"></script>
    <script src="<https://cdn.jsdelivr.net/npm/@popperjs/core@2.5.2/dist/umd/popper.min.js>"></script>
    <script src="<https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js>"></script>
</body>
</html>
'''
# todo/templates/todo_list.html 생성
'''
{% extends 'base.html' %}

{% block title %}Todo List{% endblock %}

{% block content %}
    <h1 class="my-4 text-center">Todo List</h1>

    <form id="add-todo-form" method="POST" class="mb-4">
        {% csrf_token %}
        <div class="form-group">
            <input type="text" name="title" class="form-control" placeholder="Enter new todo">
        </div>
        <button type="submit" class="btn btn-primary btn-block">Add Todo</button>
    </form>

    <ul class="list-group">
        {% for todo in todos %}
        <li class="list-group-item d-flex justify-content-between align-items-center">
            {{ todo.title }}
            {% if not todo.is_completed %}
            <span>
                <a href="{% url 'todo-complete' todo.pk %}" class="btn btn-success btn-sm">Complete</a>
                <a href="{% url 'todo-delete' todo.pk %}" class="btn btn-danger btn-sm">Delete</a>
            </span>
            {% else %}
            <span class="badge badge-success">Completed</span>
            {% endif %}
        </li>
        {% empty %}
        <li class="list-group-item text-center">No todos available.</li>
        {% endfor %}
    </ul>
{% endblock %}
'''

#todo/views.py 수정
'''
from django.shortcuts import render, redirect
from rest_framework import generics,status
from .models import Todo, TodoStatusHistory,TodoComment
from .serializers import TodoSerializer, TodoCommentSerializer
from rest_framework.response import Response
from rest_framework.views import APIView
from django.utils import timezone


class TodoListView(generics.ListAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer


class TodoDetailView(generics.RetrieveAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'
    
class TodoCreateView(generics.CreateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer    

class TodoUpdateView(generics.UpdateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'    
    
    
class TodoCompleteView(APIView):
    def post(self, request, pk):
        try:
            todo = Todo.objects.get(pk=pk)
            todo.is_completed = True
            todo.save()
            serializer = TodoSerializer(todo)
            return Response(serializer.data, status=status.HTTP_200_OK)
        except Todo.DoesNotExist:
            return Response(status=status.HTTP_404_NOT_FOUND)
        
class TodoDeleteView(generics.DestroyAPIView):
    queryset = Todo.objects.all()
    lookup_field = 'pk'            
    
    
class TodoOverdueView(generics.ListAPIView):
    serializer_class = TodoSerializer

    def get_queryset(self):
        return Todo.objects.filter(due_date__lt=timezone.now(), completed=False)

    def patch(self, request, *args, **kwargs):
        overdue_todos = self.get_queryset()
        overdue_todos.update(completed=True)
        return Response({"message": "Overdue todos marked as completed."}, status=status.HTTP_200_OK)
 
 
class TodoUpdateView(generics.UpdateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer

    def perform_update(self, serializer):
        instance = self.get_object()
        previous_status = instance.completed
        updated_instance = serializer.save()
        TodoStatusHistory.objects.create(
            todo=updated_instance,
            previous_status=previous_status,
            new_status=updated_instance.completed
        )

class TodoCommentCreateView(generics.CreateAPIView):
    queryset = TodoComment.objects.all()
    serializer_class = TodoCommentSerializer 
    
class TodoCompletionStatsView(APIView):
    
    def get(self, request, *args, **kwargs):
        total = Todo.objects.count()
        completed = Todo.objects.filter(completed=True).count()
        completion_rate = (completed / total) * 100 if total > 0 else 0
        return Response({"completion_rate": completion_rate})   
    
class TodoCloneView(APIView):
    
    def post(self, request, pk, *args, **kwargs):
        todo = Todo.objects.get(pk=pk)
        todo.pk = None  # 새로운 객체로 복사
        todo.save()
        return Response(TodoSerializer(todo).data, status=status.HTTP_201_CREATED)      
    
def todo_list(request):
    if request.method == 'POST':
        title = request.POST.get('title')
        if title:
            Todo.objects.create(title=title)
        return redirect('todo-list')

    todos = Todo.objects.all()
    return render(request, 'todo_list.html', {'todos': todos})

def todo_complete(request, pk):
    todo = Todo.objects.get(pk=pk)
    todo.is_completed = True
    todo.save()
    return redirect('todo-list')

def todo_delete(request, pk):
    todo = Todo.objects.get(pk=pk)
    todo.delete()
    return redirect('todo-list')  
'''


# Tip_추가 기능 구현 ################################

# todo/models.py 수정
'''
from django.db import models

class Todo(models.Model):
    title = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    is_completed = models.BooleanField(default=False)
    priority = models.IntegerField(default=1)  # 우선순위 필드 추가
    due_date = models.DateTimeField(null=True, blank=True)  # 마감일 필드 추가
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
    
    
class TodoStatusHistory(models.Model):
    todo = models.ForeignKey(Todo, on_delete=models.CASCADE)
    previous_status = models.BooleanField()
    new_status = models.BooleanField()
    changed_at = models.DateTimeField(auto_now_add=True)    
    
class TodoComment(models.Model):
    todo = models.ForeignKey(Todo, on_delete=models.CASCADE, related_name='comments')
    text = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
'''



# todo/serializers.py 수정
'''
from rest_framework import serializers
from .models import Todo,TodoComment

class TodoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Todo
        fields = '__all__'

class TodoCommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = TodoComment
        fields = '__all__'
'''

# todo/views.py 수정
'''
from rest_framework import generics,status
from .models import Todo, TodoStatusHistory,TodoComment
from .serializers import TodoSerializer, TodoCommentSerializer
from rest_framework.response import Response
from rest_framework.views import APIView
from django.utils import timezone


class TodoListView(generics.ListAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer


class TodoDetailView(generics.RetrieveAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'
    
class TodoCreateView(generics.CreateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer    

class TodoUpdateView(generics.UpdateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
    lookup_field = 'pk'    
    
    
class TodoCompleteView(APIView):
    def post(self, request, pk):
        try:
            todo = Todo.objects.get(pk=pk)
            todo.is_completed = True
            todo.save()
            serializer = TodoSerializer(todo)
            return Response(serializer.data, status=status.HTTP_200_OK)
        except Todo.DoesNotExist:
            return Response(status=status.HTTP_404_NOT_FOUND)
        
class TodoDeleteView(generics.DestroyAPIView):
    queryset = Todo.objects.all()
    lookup_field = 'pk'            
    
    
class TodoOverdueView(generics.ListAPIView):
    serializer_class = TodoSerializer

    def get_queryset(self):
        return Todo.objects.filter(due_date__lt=timezone.now(), completed=False)

    def patch(self, request, *args, **kwargs):
        overdue_todos = self.get_queryset()
        overdue_todos.update(completed=True)
        return Response({"message": "Overdue todos marked as completed."}, status=status.HTTP_200_OK)
 
 
class TodoUpdateView(generics.UpdateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer

    def perform_update(self, serializer):
        instance = self.get_object()
        previous_status = instance.completed
        updated_instance = serializer.save()
        TodoStatusHistory.objects.create(
            todo=updated_instance,
            previous_status=previous_status,
            new_status=updated_instance.completed
        )

class TodoCommentCreateView(generics.CreateAPIView):
    queryset = TodoComment.objects.all()
    serializer_class = TodoCommentSerializer 
    
class TodoCompletionStatsView(APIView):
    
    def get(self, request, *args, **kwargs):
        total = Todo.objects.count()
        completed = Todo.objects.filter(completed=True).count()
        completion_rate = (completed / total) * 100 if total > 0 else 0
        return Response({"completion_rate": completion_rate})   
    
class TodoCloneView(APIView):
    
    def post(self, request, pk, *args, **kwargs):
        todo = Todo.objects.get(pk=pk)
        todo.pk = None  # 새로운 객체로 복사
        todo.save()
        return Response(TodoSerializer(todo).data, status=status.HTTP_201_CREATED)      
'''

# todo/urls.py
'''
from django.urls import path
from .views import TodoListView, TodoDetailView, TodoCreateView, TodoUpdateView, TodoCompleteView, TodoDeleteView, TodoOverdueView, TodoCommentCreateView,TodoCompletionStatsView,TodoCloneView

urlpatterns = [
    path('todos/', TodoListView.as_view(), name='todo-list'),
    path('todos/<int:pk>/', TodoDetailView.as_view(), name='todo-detail'),
    path('todos/create/', TodoCreateView.as_view(), name='todo-create'),
    path('todos/<int:pk>/update/', TodoUpdateView.as_view(), name='todo-update'),
    path('todos/<int:pk>/complete/', TodoCompleteView.as_view(), name='todo-complete'),
    path('todos/<int:pk>/delete/', TodoDeleteView.as_view(), name='todo-delete'),
    path('todos/overdue/', TodoOverdueView.as_view(), name='todo-overdue'),
    path('todos/<int:pk>/comments/', TodoCommentCreateView.as_view(), name='todo-comment-create'),
    path('todos/completion-stats/', TodoCompletionStatsView.as_view(), name='todo-completion-stats'),
    path('todos/<int:pk>/clone/', TodoCloneView.as_view(), name='todo-clone'),
]

'''

# 배포 ###############################

# gunicorn가 없으면 Render 배포가 안되니 반드시 추가하자.
'''
pip install gunicorn

pip freeze > requirements.txt
'''

```
{% endraw %}