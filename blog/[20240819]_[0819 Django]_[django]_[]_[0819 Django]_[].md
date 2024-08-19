''' git config core.autocrlf true '''

{% raw %}

```python
실전에는 migrate는 나중에 실행


1. 공통 설정
# window (shift+insert키) 

mkdir request_response
cd request_response
python -m venv venv
.\venv\Scripts\activate
pip install django
django-admin startproject request_response .
python manage.py migrate
python manage.py startapp main

# mac

mkdir request_response
cd request_response
python3 -m venv venv
source venv/bin/activate
pip install django
django-admin startproject request_response .
python manage.py migrate
python manage.py startapp main

# 2. 

# 인스톨된 django 버전 관리
pip freeze > requirements.txt

# requirements.txt에 있는 버전을 설치
# pip install -r requirements.txt


# request_response/urls.py

'''
from django.contrib import admin
from django.urls import path
from django.http import HttpResponse

db = [{
    "title": "hello 1",
    "content": "hello world 1",
    },
    {
    "title": "hello 2",
    "content": "hello world 2",
    }
]

# def index(request):
#     return HttpResponse("Hello, World!")

def index(request):
    posts = db[0]
    return HttpResponse(f"<h1>블로그 상세 페이지</h1><p>{posts['title']}</p><p>{posts['content']}</p>")

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/1", index),
]
'''


# 계속 진행
'''
from django.contrib import admin
from django.urls import path
from django.http import HttpResponse

db = [
    {
        "title": "hello 1",
        "content": "hello world 1",
    },
    {
        "title": "hello 2",
        "content": "hello world 2",
    },
]

# def index(request):
#     return HttpResponse("Hello, World!")


# def index(request):
#     """
#     이 HttpResponse 함수 + f-string이 하는 역할이 render입니다!
#     """
#     posts = db[0]
#     return HttpResponse(
#         f"<h1>블로그 상세 페이지</h1><p>{posts['title']}</p><p>{posts['content']}</p>"
#     )


def index(request):
    """
    템플릿 상속이란 무엇인가?
    """
    posts = db[0]
    header = "<h1>블로그</h1>"
    footer = "<p>블로그 끝</p>"
    return HttpResponse(
        f"{header}<p>{posts['title']}</p><p>{posts['content']}</p>{footer}"
    )


urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/1", index),
]
'''

# 계속 진행

'''
from django.contrib import admin
from django.urls import path
from django.http import HttpResponse
from django.template.loader import render_to_string

db = [
    {
        "title": "hello 1",
        "content": "hello world 1",
    },
    {
        "title": "hello 2",
        "content": "hello world 2",
    },
]

# def index(request):
#     return HttpResponse("Hello, World!")


# def index(request):
#     """
#     이 HttpResponse 함수 + f-string이 하는 역할이 render입니다!
#     """
#     posts = db[0]
#     return HttpResponse(
#         f"<h1>블로그 상세 페이지</h1><p>{posts['title']}</p><p>{posts['content']}</p>"
#     )


def index(request):
    """
    템플릿 상속이란 무엇인가?
    """
    posts = db[0]
    header = "<h1>블로그</h1>"
    footer = "<p>블로그 끝</p>"
    main = render_to_string("main.txt", {"posts": posts})
    return HttpResponse(f"{header}{main}{footer}")


urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/1", index),
]
'''

# settings.py
'''
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'main',
]

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS':[BASE_DIR / "templates"],
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
'''

# urls.py 계속 

'''
from django.contrib import admin
from django.urls import path
from django.http import HttpResponse
from django.template.loader import render_to_string

db = [
    {
        "title": "hello 1",
        "content": "hello world 1",
    },
    {
        "title": "hello 2",
        "content": "hello world 2",
    },
]

# def index(request):
#     return HttpResponse("Hello, World!")


# def index(request):
#     """
#     이 HttpResponse 함수 + f-string이 하는 역할이 render입니다!
#     """
#     posts = db[0]
#     return HttpResponse(
#         f"<h1>블로그 상세 페이지</h1><p>{posts['title']}</p><p>{posts['content']}</p>"
#     )


def index(request):
    """
    템플릿 상속이란 무엇인가? (텍스트로 각각의 템플릿들을 불러와서 + 데이터를 합치는 것) + 여러개를 합치는 것, 마치 아래와 같은 형태
    
    (텍스트로 각각의 템플릿들을 불러와서 + 데이터를 합치는 것) +
    (텍스트로 각각의 템플릿들을 불러와서 + 데이터를 합치는 것) +
    (텍스트로 각각의 템플릿들을 불러와서 + 데이터를 합치는 것)
    """
    posts = db[0]
    header = "<h1>블로그</h1>"
    footer = "<p>블로그 끝</p>"
    main = render_to_string("main.txt", {"posts": posts})
    return HttpResponse(f"{header}{main}{footer}")

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/1", index),
]
'''


# urls.py 계속 진행
'''
from django.contrib import admin
from django.urls import path
from django.http import HttpResponse, JsonResponse
from django.template.loader import render_to_string

db = [
    {
        "title": "hello 1",
        "content": "hello world 1",
    },
    {
        "title": "hello 2",
        "content": "hello world 2",
    },
]


def index(request):
    """
    마이크로 서비스 패턴을 사용한 블로그
    JSON 형식으로 포스트 목록을 반환
    """
    posts = db
    return JsonResponse(posts, safe=False)


urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/1", index),
]
'''

# django와 관련이 없는 서버에서 별개로 동작 FE/index.html
'''
'''

# urls.py 계속 진행

'''
from django.contrib import admin
from django.urls import path
from django.http import HttpResponse


def index(request):
    # http://127.0.0.1:8000/blog/?q=hello
    # 공식문서 : https://docs.djangoproject.com/en/5.1/ref/request-response/
    print(request)
    print(dir(request))
    print(type(request))
    # 여기에서 출력되는 것들은 templates에서도 출력 가능합니다.
    # {{request.user}}
    print(request.user)
    print(type(request.user))
    print(dir(request.user))
    print(request.user.is_authenticated)
    print(request.user.id)  # user id는 1번
    print(request.user.username)
    print(request.user.is_superuser)
    # print(request.user.password)
    print(request.GET)
    print(request.GET.get("q"))
    print(request.GET.get("q"))
    print(dir(request.GET))
    print(request.GET.get("hello", "world"))  # hello라는 쿼리가 없으면 world를 출력
    print(request.POST)
    print(request.FILES)
    print(request.COOKIES)  # Application > Cookies에 저장되어 있는 정보
    print(request.path)
    print(request.method)
    print(request.get_full_path_info())
    print(request.get_host())
    return HttpResponse("hello world")


urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/1", index),
]
'''

# urls.py 계속 진행
'''
from django.contrib import admin
from django.urls import path
from django.http import HttpResponse


def index(request):
    # http://127.0.0.1:8000/api/1?q=hello
    # http://127.0.0.1:8000/api/1?q=hello&name=world
    # 공식문서 : https://docs.djangoproject.com/en/5.1/ref/request-response/
    print("시작합니다!")
    print(request)
    print(dir(request))
    print(type(request))
    # 여기에서 출력되는 것들은 templates에서도 출력 가능합니다.
    # {{request.user}}
    print(request.user)
    print(type(request.user))
    print(dir(request.user))
    print(request.user.is_authenticated)
    print(request.user.id)  # user id는 1번
    print(request.user.username)
    print(request.user.is_superuser)
    # print(request.user.password)
    print(request.GET)
    print(request.GET.get("q"))
    print(request.GET.get("name"))
    print(dir(request.GET))
    print(request.GET.get("hello", "world"))  # hello라는 쿼리가 없으면 world를 출력
    print(request.POST)
    print(request.FILES)
    print(request.COOKIES)  # Application > Cookies에 저장되어 있는 정보
    print(request.path)
    print(request.method)
    print(request.get_full_path_info())
    print(request.get_host())
    return HttpResponse("hello world")


urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/1", index),
]
'''

# 0819아래 CBV폴더 생성 ################3교시#########
'''
cd CBV

python -m venv venv
.\venv\Scripts\activate

pip install django
pip install pillow

django-admin startproject config .

python manage.py migrate
python manage.py startapp blog
'''

# config/settings.py
'''
ALLOWED_HOSTS = ["*"]

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',
]
'''

########################################
# urls 기획

1. 다음 url이 실제 작동하도록 해주세요.
1.1 'blog/'                     : 블로그 글 목록
1.2 'blog/<int:pk>/'            : 블로그 글 읽기
1.3 'blog/write/'               : 블로그 글 작성
1.4 'blog/edit/<int:pk>/'       : 블로그 글 업데이트
1.5 'blog/delete/<int:pk>/'     : 블로그 글 삭제

########################################
# 각 url에 대한 view 함수 기획
앱이름: blog                views 함수이름   html 파일이름  비고
'blog/'                     blog_list        blog_list.html    
'blog/<int:pk>'             blog_details     blog_details.html
'blog/write/'               blog_write       blog_write.html
'blog/edit/<int:pk>/'       blog_edit        blog_edit.html
'blog/delete/<int:pk>/'     blog_delete      blog_delete.html


# GPT 프롬프트
'''
내가 django blog 앱을 기획하고 있어. 아래 글을 보고 models.py를 기획해주고 기획한 것을 바탕으로 ERD를 머메이드로 그려줘.
'''

########################################
# 각 앱에 대한 models.py 기획(기획 + ERD를 손으로 하는 시대가 지나갔습니다. 이제는 GPT + 머에드로 '프롬프트로' 그립니다.)
# GPT 프롬프트: 내가 django blog 앱을 기획하고 있어. 아래 글을 보고 models.py를 기획해주고 기획한 것을 바탕으로 ERD를 머메이드로 그려줘.

네, 기꺼이 도와드리겠습니다. Django blog 앱을 위한 models.py를 기획하고 ERD를 Mermaid로 그리겠습니다. 먼저 모델을 설계한 다음 ERD를 생성하겠습니다.

Django blog 앱을 위한 models.py 기획:

1. User 모델:
   - Django의 기본 User 모델을 사용하거나 확장할 수 있습니다.

2. Post 모델:
   - title: 게시글 제목
   - content: 게시글 내용
   - created_at: 작성 일시
   - updated_at: 수정 일시
   - author: User 모델과의 외래 키 관계
   - category: Category 모델과의 외래 키 관계

3. Category 모델:
   - name: 카테고리 이름
   - slug: URL에 사용할 문자열

4. Comment 모델:
   - content: 댓글 내용
   - created_at: 작성 일시
   - updated_at: 수정 일시
   - author: User 모델과의 외래 키 관계
   - post: Post 모델과의 외래 키 관계

5. Tag 모델:
   - name: 태그 이름

6. PostTag 모델 (다대다 관계를 위한 중간 모델):
   - post: Post 모델과의 외래 키 관계
   - tag: Tag 모델과의 외래 키 관계

이제 이 모델 구조를 바탕으로 ERD를 Mermaid로 그리겠습니다.



이 ERD는 Django blog 앱의 데이터 모델 구조를 시각화한 것입니다. 주요 특징은 다음과 같습니다:

1. User 모델은 Post와 Comment와 일대다 관계를 가집니다.
2. Post 모델은 Category와 다대일 관계를 가집니다.
3. Post와 Comment 사이에는 일대다 관계가 있습니다.
4. Post와 Tag 사이의 다대다 관계는 PostTag 중간 모델을 통해 구현됩니다.


# config/urls.py
'''
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("blog/", include("blog.urls")),
]
'''

#blog/urls.py

'''
from django.urls import path
from . import views

# from .views import BlogListView, BlogDetailView, BlogCreateView, BlogUpdateView, BlogDeleteView
# 장점: 필요한 것만 가져오고, 목록 파악 용이
# from . import views
# 장점: 가져올 것이 많을 때, 특히 유지보수를 자주하게될 경우 용이(위에를 수정할 필요가 없기 때문에)

urlpatterns = [
    path("", views.blog_list, name="blog_list"),
    path("<int:pk>/", views.blog_detail, name="blog_detail"),
    path("new/", views.blog_new, name="blog_new"),
    path("<int:pk>/edit/", views.blog_edit, name="blog_edit"),
    path("<int:pk>/delete/", views.blog_delete, name="blog_delete"),
    path("test/", views.test, name="test"),
]
'''

# blog/views.py

'''
from django.shortcuts import render
from django.views.generic import (
    ListView,
    DetailView,
    CreateView,
    UpdateView,
    DeleteView,
)
from django.urls import reverse_lazy
from django.http import HttpResponse
from .models import Post

# 클래스 기반 뷰가 꼭 제네릭 뷰는 아닙니다.
# 클래스로 HttpResponse를 반환하게 하면 그것도 클래스 기반 뷰입니다.
# 실무에서는 클래스 기반 뷰를 제네릭 뷰라고 부르는 경우가 많습니다.
# 제네릭 뷰는 장고에서 제공하는 여러가지 기능을 미리 구현해 놓은 클래스 기반 뷰입니다.


class PostList(ListView):
    model = Post
    ordering = "-created_at"  # "-pk"도 가능
    # template_name = "blog/내가_원하는_파일명.html" # default: blog/post_list.html
    # context_object_name = "posts" # default: object_list
    # paginate_by = 5 # 페이지네이션(한 페이지에 몇 개를 보여줄지)


class PostDetail(DetailView):
    model = Post
    # template_name = "blog/내가_원하는_파일명.html" # default: blog/post_detail.html
    # context_object_name = "post" # default: object


class PostCreate(CreateView):
    model = Post
    fields = "__all__"
    success_url = reverse_lazy("blog_list")  # 성공했을 때 이동할 URL
    # fields = ["title", "content", "author"] # 이렇게 필드를 지정해줄 수도 있다.
    # template_name = "blog/내가_원하는_파일명.html" # default: blog/post_form.html
    # success_url = "/blog/" # default: reverse_lazy("blog_list")
    # reverse_lazy("blog_list")를 하는 이유는 object가 생성이 되고 나서 url로 이동해야 하는데 reverse는 함수이기 때문에 함수가 실행되는 시점에 url로 이동하게 되어버린다. 그래서 post가 생성된 후에 url로 이동하게 하기 위해서 기다리겠다는 함수가 reverse_lazy를 사용한다.


class PostUpdate(UpdateView):
    model = Post
    fields = "__all__"  # "__all__"도 가능
    success_url = reverse_lazy("blog_list")  # 성공했을 때 이동할 URL
    # template_name = "blog/내가_원하는_파일명.html" # default: blog/post_form.html
    # success_url = "/blog/" # default: reverse_lazy("blog_list")


class PostDelete(DeleteView):
    model = Post
    success_url = reverse_lazy("blog_list")  # 성공했을 때 이동할 URL
    # template_name = "blog/내가_원하는_파일명.html" # default: blog/post_confirm_delete.html
    # success_url = "/blog/" # default: reverse_lazy("blog_list")
    # 삭제되고 다 완료되지 않은 상태에서 blog_list로 넘어가지 않도록 하기 위해서 reverse_lazy를 사용합니다.


class PostTest(CreateView):
    model = Post

    # 메서드 오버라이딩(CreateView의 메서드를 재정의)
    def get(self, request):
        return HttpResponse("get 요청이 왔습니다.")

    def post(self, request):
        return HttpResponse("post 요청이 왔습니다.")


# 아래와 같이 사용하진 않고, urls.py에서 as_view()를 사용하여 사용하는 경우가 많습니다.
blog_list = PostList.as_view()
blog_detail = PostDetail.as_view()
blog_new = PostCreate.as_view()
blog_edit = PostUpdate.as_view()
blog_delete = PostDelete.as_view()
test = PostTest.as_view()
'''

# views.py 설명
'''
Django에서 ListView와 같은 일반적인 Class-Based Views (CBV)를 사용할 때, 템플릿 이름은 기본적으로 다음과 같은 규칙을 따라 자동으로 생성됩니다.

PostList (ListView)
템플릿 이름 규칙: <app_name>/<model_name_소문자>_list.html
여기서의 기본 템플릿: <app_name>/post_list.html
템플릿 접근 방법:
{% for post in object_list %}
    {{ post.title }}
{% endfor %}


PostDetail (DetailView)
템플릿 이름 규칙: <app_name>/<model_name_소문자>_detail.html
여기서의 기본 템플릿: <app_name>/post_detail.html
템플릿 접근 방법: 
{{ object.title }}


PostCreate (CreateView)
템플릿 이름 규칙: <app_name>/<model_name_소문자>_form.html
여기서의 기본 템플릿: <app_name>/post_form.html
템플릿 접근 방법:
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Create</button>
</form>


PostUpdate (UpdateView)
템플릿 이름 규칙: <app_name>/<model_name_소문자>_form.html
여기서의 기본 템플릿: <app_name>/post_form.html
템플릿 접근 방법:
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Update</button>
</form>


PostDelete (DeleteView)
템플릿 이름 규칙:  <app_name>/<model_name_소문자>_confirm_delete.html
여기서의 기본 템플릿: <app_name>/post_confirm_delete.html
템플릿 접근 방법:
<form method="post">
    {% csrf_token %}
    Are you sure you want to delete "{{ object.title }}"?
    <button type="submit">Delete</button>
</form>


* CreateView와 UpdateView는 같은 템플릿 이름 규칙을 사용합니다. 그래서 둘 다 _form.html을 기본으로 사용합니다.
'''

# blog/templates/blog/post_list.html
'''
<h2>Post List</h2>
<ul>
    {% for post in object_list %}
    <li>
        <a href="{% url 'blog_detail' post.pk %}">{{ post.title }}</a>
    </li>
    {% endfor %}
</ul>
'''

# blog/templates/blog/post_detail.html
'''
<h2>{{ object.title }}</h2>
<p>{{ object.content }}</p>
'''




# blog/templates/blog/post_form.html
'''
<h2>Create / Update</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">저장</button>
</form>
'''


# blog/templates/blog/post_confirm_delete.html
'''
<h2>Delete</h2>
<form method="post">
    {% csrf_token %}
    <p>"{{ object.title }}"을 정말로 삭제하시겠습니까?</p>
    <a href="{% url 'blog_detail' object.pk %}">취소</a>
    <button type="submit">삭제</button>
</form>
'''

# blog/models.py

'''
from django.db import models
from django.contrib.auth.models import User


class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name_plural = "Categories"


class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    head_image = models.ImageField(upload_to="blog/images/%Y/%m/%d/", blank=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name="posts")
    category = models.ForeignKey(
        Category, on_delete=models.SET_NULL, null=True, related_name="posts"
    )

    def __str__(self):
        return self.title

    class Meta:
        ordering = ["-created_at"]

# upload_to를 문자열이나 함수로 사용하는 이유
# 1. 보안상
'''

# blog/admin.py

'''
from django.contrib import admin
from .models import Post

admin.site.register(Post)
'''

# 마이그레이션

'''
python manage.py makemigrations
python manage.py migrate
'''


# 슈퍼유저 생성
'''
'''

# settings.py 내용 추가 및 경로 생성 (CBV/media, CBV/static)
'''
STATIC_URL = "static/"
STATICFILES_DIRS = [BASE_DIR / "static"]

MEDIA_URL = "/media/"
MEDIA_ROOT = BASE_DIR / "media"
'''

# config/urls.py
'''
from django.contrib import admin
from django.urls import path, include
from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
    path("admin/", admin.site.urls),
    path("blog/", include("blog.urls")),
]

# 실무에서는 이런식으로 잘 사용하지 않습니다.
# django 서버는 미디어를 serving하는 프레임워크가 아니기때문에
# 이미지나 영상은 nginx, apache같은 웹서버를 이용해서 서빙하거나 AWS s3같은 클라우드 서비스를 이용하는 것이 좋습니다.

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
'''

# 동작 확인
'''
http://127.0.0.1:8000/blog/
http://127.0.0.1:8000/blog/1/
http://127.0.0.1:8000/blog/new/
http://127.0.0.1:8000/blog/1/edit/
http://127.0.0.1:8000/blog/1/delete
'''

# blog/templates/blog/post_list.html 수정(검색 기능 추가(CBV에 검색이 어떤 식으로 작동하는 지 이해하기 위한 예제))
'''
<h2>Post List</h2>
<form action="" method="get">
    <input name="q" type="text">
    <button type="submit">검색</button>
</form>
<ul>
    {% for post in object_list %}
    <li>
        <a href="{% url 'blog_detail' post.pk %}">{{ post.title }}</a>
    </li>
    {% endfor %}
</ul>
'''

# blog/views.py 수정
'''
from django.shortcuts import render
from django.views.generic import (
    ListView,
    DetailView,
    CreateView,
    UpdateView,
    DeleteView,
)
from django.urls import reverse_lazy
from django.http import HttpResponse
from .models import Post
from django.db.models import Q

# 클래스 기반 뷰가 꼭 제네릭 뷰는 아닙니다.
# 클래스로 HttpResponse를 반환하게 하면 그것도 클래스 기반 뷰입니다.
# 실무에서는 클래스 기반 뷰를 제네릭 뷰라고 부르는 경우가 많습니다.
# 제네릭 뷰는 장고에서 제공하는 여러가지 기능을 미리 구현해 놓은 클래스 기반 뷰입니다.


class PostList(ListView):
    model = Post
    ordering = "-created_at"  # "-pk"도 가능
    # template_name = "blog/내가_원하는_파일명.html" # default: blog/post_list.html
    # context_object_name = "posts" # default: object_list
    # paginate_by = 5 # 페이지네이션(한 페이지에 몇 개를 보여줄지)

    def get_queryset(self):
        """
        get요청으로 들어온 데이터를 처리할 때 사용하는 함수
        꼭 검색이 아니라 모든 GET 요청에 대해 처리할 수 있습니다.
        127.0.0.1:8000 GET 이걸로 들어오게 되면?
        127.0.0.1:8000?q=hello GET 이걸로 들어오게 되면?
        127.0.0.1:8000?q=hello&name=hojun GET 이걸로 들어오게 되면?
        """
        queryset = super().get_queryset()  # Post.objects.all()과 같은 역할을 합니다.
        # 템플릿에서는 {{object_list}}로 사용할 수 있습니다.
        q = self.request.GET.get("q", "")  # q라는 이름으로 GET 요청을 받습니다.
        if q:
            # q의 값을 가지고 있는 데이터를 추출합니다.
            # queryset = queryset.filter(title__icontains=q)
            queryset = queryset.filter(
                Q(title__icontains=q) | Q(content__icontains=q)
            ).distinct()
        return queryset


class PostDetail(DetailView):
    model = Post
    # template_name = "blog/내가_원하는_파일명.html" # default: blog/post_detail.html
    # context_object_name = "post" # default: object


class PostCreate(CreateView):
    model = Post
    fields = "__all__"
    success_url = reverse_lazy("blog_list")  # 성공했을 때 이동할 URL
    # fields = ["title", "content", "author"] # 이렇게 필드를 지정해줄 수도 있다.
    # template_name = "blog/내가_원하는_파일명.html" # default: blog/post_form.html
    # success_url = "/blog/" # default: reverse_lazy("blog_list")
    # reverse_lazy("blog_list")를 하는 이유는 object가 생성이 되고 나서 url로 이동해야 하는데 reverse는 함수이기 때문에 함수가 실행되는 시점에 url로 이동하게 되어버린다. 그래서 post가 생성된 후에 url로 이동하게 하기 위해서 기다리겠다는 함수가 reverse_lazy를 사용한다.


class PostUpdate(UpdateView):
    model = Post
    fields = "__all__"  # "__all__"도 가능
    success_url = reverse_lazy("blog_list")  # 성공했을 때 이동할 URL
    # template_name = "blog/내가_원하는_파일명.html" # default: blog/post_form.html
    # success_url = "/blog/" # default: reverse_lazy("blog_list")


class PostDelete(DeleteView):
    model = Post
    success_url = reverse_lazy("blog_list")  # 성공했을 때 이동할 URL
    # template_name = "blog/내가_원하는_파일명.html" # default: blog/post_confirm_delete.html
    # success_url = "/blog/" # default: reverse_lazy("blog_list")
    # 삭제되고 다 완료되지 않은 상태에서 blog_list로 넘어가지 않도록 하기 위해서 reverse_lazy를 사용합니다.


class PostTest(CreateView):
    model = Post

    # 메서드 오버라이딩(CreateView의 메서드를 재정의)
    def get(self, request):
        return HttpResponse("get 요청이 왔습니다.")

    def post(self, request):
        return HttpResponse("post 요청이 왔습니다.")


# 아래와 같이 사용하진 않고, urls.py에서 as_view()를 사용하여 사용하는 경우가 많습니다.
blog_list = PostList.as_view()
blog_detail = PostDetail.as_view()
blog_new = PostCreate.as_view()
blog_edit = PostUpdate.as_view()
blog_delete = PostDelete.as_view()
test = PostTest.as_view()
'''

# blog/admin.py
'''
from django.contrib import admin
from .models import Post, Category

# admin.site.register(Post)
admin.site.register(Category)


class PostAdmin(admin.ModelAdmin):
    list_display = ("title", "content", "created_at")


admin.site.register(Post, PostAdmin)
'''
















# django 5.1 ##########################
'''
장고 5.1 버전 주요 업데이트 내용

인증 기능 개선

새로운 LoginRequiredMiddleware 도입
MIDDLEWARE에 추가하면 모든 뷰에 대해 기본적으로 인증 요구
개발자가 각 뷰마다 인증 설정을 하지 않아도 되어 편리성 향상


접근성 강화

관리자 인터페이스의 스크린 리더 지원 개선
더 의미 있는 HTML 요소 사용
폼 필드셋에서 도움말 텍스트와 레이블 연결 개선
다양한 사용자들의 웹 접근성 향상


querystring 템플릿 태그 추가

템플릿에서 URL 생성 시 쿼리 문자열 처리 간소화
개발자들의 URL 관련 작업 효율성 증대
'''




# QnA

'''
파일과 이미지는 보통 django 서버에 저장하지 않습니다. AWS S3같은 곳에 저장해요 TextField나 FileField
'''

'''
static은 내가 사용할 정적 파일(로고 같은..)
media는 사용자가 업로드할 파일을 저장하는 디렉토리입니다.

'''


# 참고 항목1
'''
########################################

ListView의 기본 get_queryset 메서드는 model 속성에서 정의된 모델의 전체 객체 목록을 반환합니다. PostList 뷰에서 model = Post로 정의되어 있어 기본적으로 Post 모델의 모든 객체를 반환하게 됩니다.

super().get_queryset() 호출은 Post 모델의 전체 객체 목록을 반환합니다.

########################################

공식문서
https://docs.djangoproject.com/en/5.0/ref/class-based-views/
ListVeiw : https://docs.djangoproject.com/en/5.0/ref/class-based-views/generic-display/
CreateView : https://docs.djangoproject.com/en/5.0/ref/class-based-views/generic-editing/

################################

Base views
    View : 최상위 제네릭 뷰, 기본 뷰, django view를 만드는데 필요한 기능 제공
    TemplateView : 템플릿이 주어지면 렌더링을 해주는 뷰
    RedirectView : URL이 주어지면 리다이렉트 해주는 뷰

Generic display views
    DetailView : model과 템플릿 받아 조건에 맞는 상세 오브젝트를 보여줍니다.
    ListView : model과 템플릿 받아 전체 오브젝트를 보여줍니다.

Generic editing views
    FormView : 폼을 보여주고 처리합니다.
    CreateView : 폼을 보여주고 객체를 생성합니다.
    UpdateView : 폼을 조건에 맞게 보여주고 객체를 수정합니다. 
    DeleteView : 객체를 삭제합니다.

Generic date views
    ArchiveIndexView : 조건에 맞는 객체의 날짜 정보를 출력합니다.
    YearArchiveView : 연도에 맞는 객체를 출력합니다.
    MonthArchiveView : 월에 맞는 객체를 출력합니다.
    WeekArchiveView : 주에 맞는 객체를 출력합니다.
    DayArchiveView : 일에 맞는 객체를 출력합니다.
    TodayArchiveView : 오늘 날짜에 객체를 출력합니다.
    DateDetailView : 연, 월, 일 조건에 맞는 객체를 출력합니다.

################################
'''



# 참고 항목 2

# 공식 홈페이지 샘프로 코드 포함

Base views
    View : 최상위 제네릭 뷰, 기본 뷰, django view를 만드는데 필요한 기능 제공
    TemplateView : 템플릿이 주어지면 렌더링을 해주는 뷰
    '''
    from django.views.generic.base import TemplateView
    from articles.models import Article

    class HomePageView(TemplateView):
        template_name = "home.html"

        def get_context_data(self, **kwargs):
            context = super().get_context_data(**kwargs)
            context["latest_articles"] = Article.objects.all()[:5]
            return context
    '''
    RedirectView : URL이 주어지면 리다이렉트 해주는 뷰

Generic display views
    DetailView : model과 템플릿 받아 조건에 맞는 상세 오브젝트를 보여줍니다.
    '''
    from django.utils import timezone
    from django.views.generic.detail import DetailView
    from articles.models import Article

    class ArticleDetailView(DetailView):
        model = Article

        def get_context_data(self, **kwargs):
            context = super().get_context_data(**kwargs)
            context["now"] = timezone.now()
            return context
    '''
    '''
    <h1>{{ object.headline }}</h1>
    <p>{{ object.content }}</p>
    <p>Reporter: {{ object.reporter }}</p>
    <p>Published: {{ object.pub_date|date }}</p>
    <p>Date: {{ now|date }}</p>
    '''
    ListView : model과 템플릿 받아 전체 오브젝트를 보여줍니다.
    '''
    from django.utils import timezone
    from django.views.generic.detail import DetailView
    from articles.models import Article

    class ArticleDetailView(DetailView):
        model = Article

        def get_context_data(self, **kwargs):
            context = super().get_context_data(**kwargs)
            context["now"] = timezone.now()
            return context
    '''
    '''
    <h1>{{ object.headline }}</h1>
    <p>{{ object.content }}</p>
    <p>Reporter: {{ object.reporter }}</p>
    <p>Published: {{ object.pub_date|date }}</p>

    <p>Date: {{ now|date }}</p>
    '''

Generic editing views
    FormView : 폼을 보여주고 처리합니다.
    CreateView : 폼을 보여주고 객체를 생성합니다.
    UpdateView : 폼을 조건에 맞게 보여주고 객체를 수정합니다. 
    DeleteView : 객체를 삭제합니다.

Generic date views
    ArchiveIndexView : 조건에 맞는 객체의 날짜 정보를 출력합니다.
    YearArchiveView : 연도에 맞는 객체를 출력합니다.
    MonthArchiveView : 월에 맞는 객체를 출력합니다.
    WeekArchiveView : 주에 맞는 객체를 출력합니다.
    DayArchiveView : 일에 맞는 객체를 출력합니다.
    TodayArchiveView : 오늘 날짜에 객체를 출력합니다.
    DateDetailView : 연, 월, 일 조건에 맞는 객체를 출력합니다.
    '''
    # views.py
    from django.views.generic.dates import WeekArchiveView
    from myapp.models import Article

    class ArticleWeekArchiveView(WeekArchiveView):
        queryset = Article.objects.all()
        date_field = "pub_date"
        week_format = "%W"
        allow_future = True
    
    # urls.py
    from django.urls import path
    from myapp.views import ArticleWeekArchiveView

    urlpatterns = [
        # Example: /2012/week/23/
        path(
            "<int:year>/week/<int:week>/",
            ArticleWeekArchiveView.as_view(),
            name="archive_week",
        ),
    ]

    # article_archive_week.html:
    <h1>Week {{ week|date:'W' }}</h1>

    <ul>
        {% for article in object_list %}
            <li>{{ article.pub_date|date:"F j, Y" }}: {{ article.title }}</li>
        {% endfor %}
    </ul>

    <p>
        {% if previous_week %}
            Previous Week: {{ previous_week|date:"W" }} of year {{ previous_week|date:"Y" }}
        {% endif %}
        {% if previous_week and next_week %}--{% endif %}
        {% if next_week %}
            Next week: {{ next_week|date:"W" }} of year {{ next_week|date:"Y" }}
        {% endif %}
    </p>
    '''

```
{% endraw %}