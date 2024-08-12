{% raw %}

```

# 환경 설정
'''
python -m venv venv

.\venv\Scripts\activate

pip install django

django-admin startproject bookproject
cd bookproject
python manage.py startapp books
'''

# 마이그레이션
'''
python manage.py makemigrations
python manage.py migrate
'''

# 폴더 생성
'''
G:\DJANGO_MODU_03\bookproject> mkdir static
'''

# 슈퍼유저 생성
'''
python manage.py createsuperuser
'''

# 세션 2: ListView를 활용한 도서 목록 구현 #########


# books/views.py 수정
'''
from django.views.generic import ListView
from .models import Book

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
'''

#URL  패턴을 지정!
# Template를 구성해서 사용자가 접근할 수 있도록 함!

# books/urls.py 생성
'''
from django.urls import path
from .views import BookListView

app_name = 'books'

urlpatterns = [
    path('', BookListView.as_view(), name='book_list'),
]
'''

# bookproject/urls.py 수정
'''
from django.contrib import admin
from django.urls import path,include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('books/', include('books.urls')),
]
'''

# templates/books/book_list.html
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>도서 목록</title>
</head>
<body>
    <h1>도서 목록</h1>
    <ul>
        {% for book in books %}
            <li>
                <strong>{{ book.title }}</strong> - {{ book.author }} ({{ book.publication_date }})
            </li>
        {% endfor %}
    </ul>
</body>
</html>
'''


# templates/base.html
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}도서 관리 시스템{% endblock %}</title>
</head>
<body>
    <header>
        <h1>도서 관리 시스템</h1>
        <nav>
            <ul>
                <li><a href="{% url 'main' %}">홈</a></li>
                <li><a href="{% url 'books:book_list' %}">도서 목록</a></li>
            </ul>
        </nav>
    </header>
    <main>
        {% block content %}
        <!-- 여기에 개별 페이지의 내용이 추가됩니다 -->
        {% endblock %}
    </main>
    <footer>
        <p>&copy; 2024 도서 관리 시스템</p>
    </footer>
</body>
</html>
'''

#  templates/main.html 생성
'''
{% extends 'base.html' %}

{% block title %}홈{% endblock %}

{% block content %}
    <h2>홈페이지</h2>
    <p>도서 관리 시스템에 오신 것을 환영합니다.</p>
    <p>상단 메뉴를 통해 원하는 페이지로 이동할 수 있습니다.</p>
{% endblock %}
'''

# templates/books/book_list.html 수정
'''
{% extends 'base.html' %}

{% block title %}도서 목록{% endblock %}

{% block content %}
    <h2>도서 목록</h2>
    <ul>
        {% for book in books %}
            <li>
                <strong>{{ book.title }}</strong> - {{ book.author }} ({{ book.publication_date }})
            </li>
        {% endfor %}
    </ul>
{% endblock %}
'''

# bookproject/urls.py 수정
'''
from django.contrib import admin
from django.urls import path, include
from books.views import MainView  # MainView를 임포트

urlpatterns = [
    path('admin/', admin.site.urls),
    path('books/', include('books.urls')),  # books 앱의 URL 패턴 포함
    path('', MainView.as_view(), name='main'),  # 메인 페이지 URL 패턴
]
'''

# book/views.py 수정
'''
from django.views.generic import ListView
from .models import Book
from django.views.generic import TemplateView

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
    
class MainView(TemplateView):
    template_name = 'main.html'  # 메인 페이지로 사용할 템플릿 파일 지정    
'''


# 세션 3: DetailView를 활용한 도서 상세 페이지 구현 ######################

# book/views.py 수정
'''
from django.views.generic import ListView
from .models import Book
from django.views.generic import TemplateView,DetailView

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
    
class MainView(TemplateView):
    template_name = 'main.html'  # 메인 페이지로 사용할 템플릿 파일 지정    
    
class BookDetailView(DetailView):
    model = Book  # 사용할 모델 지정
    template_name = 'books/book_detail.html'  # 템플릿 파일 지정
    context_object_name = 'book'  # 템플릿에서 사용할 컨텍스트 이름 지정   
'''

# books/urls.py 수정
'''
from django.urls import path
from .views import BookListView, BookDetailView

app_name = 'books'

urlpatterns = [
    path('', BookListView.as_view(), name='book_list'),
    path('<int:pk>/', BookDetailView.as_view(), name='book_detail'),
]
'''


# templates/books/book_detail.html 생성
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ book.title }} - 상세 정보</title>
</head>
<body>
    <h1>{{ book.title }}</h1>
    <p><strong>저자:</strong> {{ book.author }}</p>
    <p><strong>출판일:</strong> {{ book.publication_date }}</p>
    <a href="{% url 'books:book_list' %}">도서 목록으로 돌아가기</a>
</body>
</html>
'''

# templates/books/book_list.html 수정
'''
{% extends 'base.html' %}

{% block title %}도서 목록{% endblock %}

{% block content %}
    <h2>도서 목록</h2>
    <ul>
        {% for book in books %}
        <li>
            <strong><a href="{% url 'books:book_detail' book.pk %}">{{ book.title }}</a></strong> - {{ book.author }} ({{ book.publication_date }})
        </li>
        {% endfor %}
    </ul>
{% endblock %}
'''

# 세션 4: CreateView를 활용한 도서 생성 기능 구현 ########

# books/forms.py 생성
'''
from django import forms
from .models import Book

class BookForm(forms.ModelForm):
    class Meta:
        model = Book
        fields = ['title', 'author', 'publication_date']
        widgets = {
            'publication_date': forms.DateInput(attrs={'type': 'date'}),  # 날짜 선택기를 위한 위젯 설정
        }
        labels = {
            'title': '제목',
            'author': '저자',
            'publication_date': '출판일',
        }
'''

# books/views.py 수정
'''
from django.views.generic import TemplateView,DetailView,ListView
from django.views.generic.edit import CreateView
from django.urls import reverse_lazy
from .models import Book
from .forms import BookForm

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
    paginate_by = 1  # 한 페이지에 1개의 항목을 표시 (추가)
    
class MainView(TemplateView):
    template_name = 'main.html'  # 메인 페이지로 사용할 템플릿 파일 지정    
    
class BookDetailView(DetailView):
    model = Book  # 사용할 모델 지정
    template_name = 'books/book_detail.html'  # 템플릿 파일 지정
    context_object_name = 'book'  # 템플릿에서 사용할 컨텍스트 이름 지정 

class BookCreateView(CreateView):
    model = Book
    form_class = BookForm
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('books:book_list')       
'''

# books/urls.py 수정
'''
from django.urls import path
from .views import BookListView, BookDetailView, BookCreateView

app_name = 'books' # 앱의 네임스페이스를 지정 프로젝트 내에서 이름이 겹치지 않게 하기 위함

urlpatterns = [
    path('', BookListView.as_view(), name='book_list'), # books/로 접속하면 BookListView가 실행되도록 함
    path('<int:pk>/', BookDetailView.as_view(), name = 'book_detail'), # books/숫자/로 접속하면 BookDetailView가 실행되도록 함
    path('new/', BookCreateView.as_view(), name='book_create'),
]
'''

#  templates/books/book_form.html 생성
'''
{% extends 'base.html' %}

{% block title %}도서 추가{% endblock %}

{% block content %}
    <h2>도서 추가</h2>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">저장</button>
    </form>
    <a href="{% url 'books:book_list' %}">도서 목록으로 돌아가기</a>
{% endblock %}
'''
# 세션 5: UpdateView를 활용한 도서 수정 기능 구현 ########


# books/views.py 수정
'''
from django.views.generic import TemplateView,DetailView,ListView
from django.views.generic.edit import CreateView, UpdateView
from django.urls import reverse_lazy
from .models import Book
from .forms import BookForm

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
    paginate_by = 1  # 한 페이지에 1개의 항목을 표시 (추가)
    
class MainView(TemplateView):
    template_name = 'main.html'  # 메인 페이지로 사용할 템플릿 파일 지정    
    
class BookDetailView(DetailView):
    model = Book  # 사용할 모델 지정
    template_name = 'books/book_detail.html'  # 템플릿 파일 지정
    context_object_name = 'book'  # 템플릿에서 사용할 컨텍스트 이름 지정 

class BookCreateView(CreateView):
    model = Book
    form_class = BookForm
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('books:book_list')       

class BookUpdateView(UpdateView):
    model = Book  # 수정할 모델
    form_class = BookForm  # 사용할 폼 클래스
    template_name = 'books/book_form.html'  # 템플릿 파일
    # success_url = reverse_lazy('books:book_list')  # 수정 후 리다이렉트할 URL    
    
    def get_success_url(self):
        return reverse_lazy('books:book_detail', kwargs={'pk': self.object.pk})
'''

# books/urls.py 수정
'''
from django.urls import path
from .views import BookListView, BookDetailView, BookCreateView ,BookUpdateView

app_name = 'books' # 앱의 네임스페이스를 지정 프로젝트 내에서 이름이 겹치지 않게 하기 위함

urlpatterns = [
    path('', BookListView.as_view(), name='book_list'), # books/로 접속하면 BookListView가 실행되도록 함
    path('<int:pk>/', BookDetailView.as_view(), name = 'book_detail'), # books/숫자/로 접속하면 BookDetailView가 실행되도록 함
    path('new/', BookCreateView.as_view(), name='book_create'),
    path('<int:pk>/edit/', BookUpdateView.as_view(), name='book_update'),
]
'''

#  templates/books/book_form.html 수정
'''
{% extends 'base.html' %}

{% block title %}
    {% if view.object.pk %}도서 정보 수정{% else %}도서 추가{% endif %}
{% endblock %}

{% block content %}
    <h2>
        {% if view.object.pk %}도서 정보 수정{% else %}도서 추가{% endif %}
    </h2>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">
            {% if view.object.pk %}수정{% else %}추가{% endif %}
        </button>
    </form>
    <a href="{% url 'books:book_list' %}">도서 목록으로 돌아가기</a>
{% endblock %}
'''

# templates/books/book_detail.html 수정
'''
{% extends 'base.html' %}

{% block title %}{{ book.title }} - 상세 정보{% endblock %}

{% block content %}
    <h1>{{ book.title }}</h1>
    <p><strong>저자:</strong> {{ book.author }}</p>
    <p><strong>출판일:</strong> {{ book.publication_date }}</p>
    <a href="{% url 'books:book_update' book.pk %}">도서 정보 수정</a>
    <br>
    <a href="{% url 'books:book_list' %}">목록으로 돌아가기</a>
{% endblock %}
'''

''' 첨부
# get_success_url이 왜 필요한지?

UpdateView → 이미 존재하는 객체에 수정하는 작업

수정된 객체가 무엇인지에 대해 정확히 알아야하는데, 수정 후에 어디로 리디렉션이 되어야하는지? → 제대로 수행이 완료가 되었는지! 

수정이 제대로 되었는지 확인도 하고, form 수행한 이후에 원래 페이지로 돌아가는 용도!

# Kwargs의 역할

kwargs={'pk': [self.object.pk](http://self.object.pk/)}
'''

# 세션 6: DeleteView를 활용한 도서 삭제 기능 구현########

# books.views.py 수정
'''
from django.views.generic import TemplateView,DetailView,ListView
from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Book
from .forms import BookForm

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
    paginate_by = 1  # 한 페이지에 1개의 항목을 표시 (추가)
    
class MainView(TemplateView):
    template_name = 'main.html'  # 메인 페이지로 사용할 템플릿 파일 지정    
    
class BookDetailView(DetailView):
    model = Book  # 사용할 모델 지정
    template_name = 'books/book_detail.html'  # 템플릿 파일 지정
    context_object_name = 'book'  # 템플릿에서 사용할 컨텍스트 이름 지정 

class BookCreateView(CreateView):
    model = Book
    form_class = BookForm
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('books:book_list')       

class BookUpdateView(UpdateView):
    model = Book  # 수정할 모델
    form_class = BookForm  # 사용할 폼 클래스
    template_name = 'books/book_form.html'  # 템플릿 파일
    # success_url = reverse_lazy('books:book_list')  # 수정 후 리다이렉트할 URL    
    
    def get_success_url(self):
        return reverse_lazy('books:book_detail', kwargs={'pk': self.object.pk})
    
class BookDeleteView(DeleteView):
    model = Book
    template_name = 'books/book_confirm_delete.html'
    success_url = reverse_lazy('books:book_list')    
'''

# books/urls.py 수정
'''
from django.urls import path
from .views import BookListView, BookDetailView, BookCreateView ,BookUpdateView,BookDeleteView

app_name = 'books' # 앱의 네임스페이스를 지정 프로젝트 내에서 이름이 겹치지 않게 하기 위함

urlpatterns = [
    path('', BookListView.as_view(), name='book_list'), # books/로 접속하면 BookListView가 실행되도록 함
    path('<int:pk>/', BookDetailView.as_view(), name = 'book_detail'), # books/숫자/로 접속하면 BookDetailView가 실행되도록 함
    path('new/', BookCreateView.as_view(), name='book_create'),
    path('<int:pk>/edit/', BookUpdateView.as_view(), name='book_update'),
    path('<int:pk>/delete/', BookDeleteView.as_view(), name='book_delete'),
]
'''


# templates/books/book_confirm_delete.html 생성
'''
{% extends 'base.html' %}

{% block title %}도서 삭제{% endblock %}

{% block content %}
    <h2>도서 삭제</h2>
    <p>정말로 이 도서를 삭제하시겠습니까?</p>
    <p><strong>{{ book.title }}</strong> - {{ book.author }}</p>
    <form method="post">
        {% csrf_token %}
        <button type="submit">삭제</button>
        <a href="{% url 'books:book_detail' book.pk %}">취소</a>
    </form>
{% endblock %}
'''

# templates/books/book_detail.html 수정
'''
{% extends 'base.html' %}

{% block title %}{{ book.title }} - 상세 정보{% endblock %}

{% block content %}
    <h1>{{ book.title }}</h1>
    <p><strong>저자:</strong> {{ book.author }}</p>
    <p><strong>출판일:</strong> {{ book.publication_date }}</p>
    <a href="{% url 'books:book_update' book.pk %}">도서 정보 수정</a>
    <br>
    <a href="{% url 'books:book_delete' book.pk %}">도서 삭제</a> <!-- 삭제 링크 추가 -->
    <br>
    <a href="{% url 'books:book_list' %}">목록으로 돌아가기</a>
{% endblock %}
'''


# 도전과제 #######################

# books.models.py 수정
'''
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200, verbose_name='제목') # 책 제목, 최대 100자 문자열
    author = models.CharField(max_length=100, verbose_name='저자') 
    publication_date = models.DateField(verbose_name='출판일')
    genre = models.CharField(max_length=100, verbose_name='장르') # 책 장르, 최대 100자 문자열
    summary = models.TextField(verbose_name='요약') # 책 요약, 문자열 길이 제한 없음

    def __str__(self): # 객체를 문자열로 표현할 때 사용하는 함수(매직 메소드)
        return self.title

    class Meta:
        verbose_name = '도서' # 모델의 단수 표현!
        verbose_name_plural = '도서들' # 모델의 복수 표현!
'''

# 마이그레이션 (선택지가 나오는데 exit하지 말고 1로 밀고감)
'''
python manage.py makemigrations
python manage.py migrate
'''

# templates/books/book_detail.html 수정
'''
{% extends 'base.html' %}

{% block title %}{{ book.title }} - 상세 정보{% endblock %}

{% block content %}
    <h1>{{ book.title }}</h1>
    <p><strong>저자:</strong> {{ book.author }}</p>
    <p><strong>출판일:</strong> {{ book.publication_date }}</p>
    <p><strong>장르:</strong> {{ book.genre }}</p>
    <p><strong>요약:</strong> {{ book.summary }}</p>
    <a href="{% url 'books:book_update' book.pk %}">도서 정보 수정</a>
    <br>
    <a href="{% url 'books:book_delete' book.pk %}">도서 삭제</a> <!-- 삭제 링크 추가 -->
    <br>
    <a href="{% url 'books:book_list' %}">목록으로 돌아가기</a>
{% endblock %}
'''

# books/forms.py 수정
'''
from django import forms
from .models import Book

class BookForm(forms.ModelForm):
    class Meta:
        model = Book
        fields = ['title', 'author', 'publication_date', 'genre', 'summary']  # 모든 필드 포함
        widgets = {
            'publication_date': forms.DateInput(attrs={'type': 'date'}),  # 날짜 선택기를 위한 위젯 설정
        }
        labels = {
            'title': '제목',
            'author': '저자',
            'publication_date': '출판일',
            'genre' : '장르',
            'summary' : "요약"
        }
'''

#  이미지 추가하기 ##############

# pip install Pillow
'''
pip install Pillow
'''

# books.models.py 수정
'''
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200, verbose_name='제목') # 책 제목, 최대 100자 문자열
    author = models.CharField(max_length=100, verbose_name='저자') 
    publication_date = models.DateField(verbose_name='출판일')
    genre = models.CharField(max_length=100, verbose_name='장르') # 책 장르, 최대 100자 문자열
    summary = models.TextField(verbose_name='요약') # 책 요약, 문자열 길이 제한 없음
    image = models.ImageField(upload_to='book_images/', blank=True, null=True)  # 이미지 필드 추가

    def __str__(self): # 객체를 문자열로 표현할 때 사용하는 함수(매직 메소드)
        return self.title

    class Meta:
        verbose_name = '도서' # 모델의 단수 표현!
        verbose_name_plural = '도서들' # 모델의 복수 표현!
'''

# 마이그레이션 
'''
python manage.py makemigrations
python manage.py migrate
'''

# bookproject/settings.py 내용 추가 (전체 복붙 불가, 반드시 아래의 내용을 추가할것 )
'''
import os

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

'''

# bookproject/urls.py 수정(전체 복붙 가능)
'''
from django.contrib import admin
from django.urls import path, include
from books.views import MainView  # MainView를 임포트
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('books/', include('books.urls')),
    path('', MainView.as_view(), name='main'),  # 메인 페이지 URL 패턴
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
'''


# books/forms.py 수정 (전체 복붙 가능)
'''
from django import forms
from .models import Book

class BookForm(forms.ModelForm):
    class Meta:
        model = Book
        fields = ['title', 'author', 'publication_date', 'genre', 'summary','image']  # 모든 필드 포함
        widgets = {
            'publication_date': forms.DateInput(attrs={'type': 'date'}),  # 날짜 선택기를 위한 위젯 설정
        }
        labels = {
            'title': '제목',
            'author': '저자',
            'publication_date': '출판일',
            'genre' : '장르',
            'summary' : '요약',
            'image' : '이미지'
        }
'''

# templates/books/book_forms.html 
'''
{% extends 'base.html' %}

{% block title %}
    {% if view.object.pk %}도서 정보 수정{% else %}도서 추가{% endif %}
{% endblock %}

{% block content %}
    <h2>
        {% if view.object.pk %}도서 정보 수정{% else %}도서 추가{% endif %}
    </h2>
    <form method="post" enctype="multipart/form-data">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">
            {% if view.object.pk %}수정{% else %}추가{% endif %}
        </button>
    </form>
    <a href="{% url 'books:book_list' %}">도서 목록으로 돌아가기</a>
{% endblock %}
'''

# templates/books/book_detail.html
'''
{% extends 'base.html' %}

{% block title %}{{ book.title }} - 상세 정보{% endblock %}

{% block content %}
    <h1>{{ book.title }}</h1>
    <p><strong>저자:</strong> {{ book.author }}</p>
    <p><strong>출판일:</strong> {{ book.publication_date }}</p>
    <p><strong>장르:</strong> {{ book.genre }}</p>
    <p><strong>요약:</strong> {{ book.summary }}</p>
    {% if book.image %}
    <img src="{{ book.image.url }}" alt="{{ book.title }} 이미지">
    {% else %}
    <p>이미지가 없습니다.</p>
    {% endif %}
    <a href="{% url 'books:book_update' book.pk %}">도서 정보 수정</a>
    <br>
    <a href="{% url 'books:book_delete' book.pk %}">도서 삭제</a> <!-- 삭제 링크 추가 -->
    <br>
    <a href="{% url 'books:book_list' %}">목록으로 돌아가기</a>
{% endblock %}
'''

```

{% endraw %}