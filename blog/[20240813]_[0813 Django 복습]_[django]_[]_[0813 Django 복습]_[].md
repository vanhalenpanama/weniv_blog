{% raw %}
```

python manage.py runserver

.\venv\Scripts\activate

python manage.py makemigrations
python manage.py migrate
...................................

# [Day 2주차 - 2일차]Django CBV 도서관 서비스 구현 (Django CBV)

# 세션 1: ListView 개선 - 페이지네이션 및 검색 기능 구현######

# books/views.py 수정
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
    
    def get_queryset(self):
        queryset = super().get_queryset() # query -> queryset
        
        query = self.request.GET.get('q')
        if query:
            queryset = queryset.filter(title__icontains=query) # query -> queryset

        # 정렬 조건을 처리합니다.
        sort = self.request.GET.get('sort')
        if sort == 'title':
            queryset = queryset.order_by('title')
        elif sort == 'author':
            queryset = queryset.order_by('author')
        elif sort == 'date':
            queryset = queryset.order_by('-publication_date')  # 최신 순으로 정렬
        else:
            queryset = queryset.order_by('-publication_date')  # 기본 정렬: 최신 순

        return queryset
    
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

# templates/books/book_list.html 수정
'''
{% extends 'base.html' %}

{% block title %}도서 목록{% endblock %}

{% block content %}
<!-- 검색 폼 -->
<form method="get" action="{% url 'books:book_list' %}">
    <input type="text" name="q" placeholder="도서 제목 검색" value="{{ request.GET.q }}">
    <button type="submit">검색</button>
</form>
<!-- 검색 및 정렬 폼 -->
<form method="get" action=".">
    <select name="sort">
        <option value="">정렬 선택</option>
        <option value="title" {% if request.GET.sort == 'title' %}selected{% endif %}>제목</option>
        <option value="author" {% if request.GET.sort == 'author' %}selected{% endif %}>저자명</option>
        <option value="date" {% if request.GET.sort == 'date' %}selected{% endif %}>출판일</option>
    </select>
    <input type="text" name="q" placeholder="도서 제목 검색" value="{{ request.GET.q }}">
    <button type="submit">적용</button>
</form>
<a href="{% url 'books:book_create' %}">새 도서 추가</a>

 <!-- 도서 목록 -->
    <h2>도서 목록</h2>
    <a href="{% url 'books:book_create' %}">새 도서 추가</a>
    <ul>
        {% for book in books %}
            <li>
                <strong><a href="{% url 'books:book_detail' book.pk %}">{{ book.title }}</a></strong> - {{ book.author }} ({{ book.publication_date|date:"Y-m-d" }})
            </li>
        {% endfor %}
    </ul>

    <!-- 페이지네이션 링크 추가 -->
    <div class="pagination">
        <span class="step-links">
            {% if page_obj.has_previous %}
                <a href="?page=1">&laquo; 처음</a>
                <a href="?page={{ page_obj.previous_page_number }}">이전</a>
            {% endif %}

            <span class="current">
                페이지 {{ page_obj.number }} / {{ page_obj.paginator.num_pages }}
            </span>

            {% if page_obj.has_next %}
                <a href="?page={{ page_obj.next_page_number }}">다음</a>
                <a href="?page={{ page_obj.paginator.num_pages }}">마지막 &raquo;</a>
            {% endif %}
        </span>
    </div>
{% endblock %}
'''

# books/views.py 수정
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
    paginate_by = 2  # 한 페이지에 1개의 항목을 표시 (추가)
    
    def get_queryset(self):
        # 기본 쿼리셋을 가져옵니다.
        queryset = super().get_queryset()

        # 검색어를 처리합니다.
        query = self.request.GET.get('q')
        if query:
            queryset = queryset.filter(title__icontains=query)

        # 정렬 조건을 처리합니다.
        sort = self.request.GET.get('sort')
        if sort == 'title':
            queryset = queryset.order_by('title')
        elif sort == 'author':
            queryset = queryset.order_by('author')
        elif sort == 'date':
            queryset = queryset.order_by('publication_date')

        # 필터링 조건을 처리합니다.
        genre = self.request.GET.get('genre')
        year = self.request.GET.get('year')
        if genre:
            queryset = queryset.filter(genre=genre)  # 장르로 필터링
        if year:
            queryset = queryset.filter(publication_date__year=year)  # 출판 연도로 필터링

        return queryset
    
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

# templates/books/book_list.html 수정
'''
{% extends 'base.html' %}

{% block title %}도서 목록{% endblock %}

{% block content %}
<!-- 검색 폼 -->
<form method="get" action="{% url 'books:book_list' %}">
    <input type="text" name="q" placeholder="도서 제목 검색" value="{{ request.GET.q }}">
    <button type="submit">검색</button>
</form>
<!-- 검색, 정렬 및 필터링 폼 -->
<form method="get" action=".">
    <select name="sort">
        <option value="">정렬 선택</option>
        <option value="title" {% if request.GET.sort == 'title' %}selected{% endif %}>제목</option>
        <option value="author" {% if request.GET.sort == 'author' %}selected{% endif %}>저자명</option>
        <option value="date" {% if request.GET.sort == 'date' %}selected{% endif %}>출판일</option>
    </select>

    <!-- 필터링 옵션 추가 -->
    <select name="genre">
        <option value="">장르 선택</option>
        <option value="소설" {% if request.GET.genre == '소설' %}selected{% endif %}>소설</option>
        <option value="비소설" {% if request.GET.genre == '비소설' %}selected{% endif %}>비소설</option>
        <option value="과학" {% if request.GET.genre == '과학' %}selected{% endif %}>과학</option>
    </select>

    <select name="year">
        <option value="">출판 연도 선택</option>
        <option value="2024" {% if request.GET.year == '2024' %}selected{% endif %}>2024</option>
        <option value="2023" {% if request.GET.year == '2023' %}selected{% endif %}>2023</option>
        <!-- 추가 연도 옵션 -->
    </select>
    <!-- 필터링 옵션 끝 -->

    <input type="text" name="q" placeholder="도서 제목 검색" value="{{ request.GET.q }}">
    <button type="submit">적용</button>
</form>


<a href="{% url 'books:book_create' %}">새 도서 추가</a>

 <!-- 도서 목록 -->
    <h2>도서 목록</h2>
    <a href="{% url 'books:book_create' %}">새 도서 추가</a>
    <ul>
        {% for book in books %}
            <li>
                <strong><a href="{% url 'books:book_detail' book.pk %}">{{ book.title }}</a></strong> - {{ book.author }} ({{ book.publication_date|date:"Y-m-d" }})
            </li>
        {% endfor %}
    </ul>

    <!-- 페이지네이션 링크 추가 -->
    <div class="pagination">
        <span class="step-links">
            {% if page_obj.has_previous %}
                <a href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page=1">&laquo; 처음</a>
                <a href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page={{ page_obj.previous_page_number }}">이전</a>
            {% endif %}

            <span class="current">
                페이지 {{ page_obj.number }} / {{ page_obj.paginator.num_pages }}
            </span>

            {% if page_obj.has_next %}
                <a href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page={{ page_obj.next_page_number }}">다음</a>
                <a href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page={{ page_obj.paginator.num_pages }}">마지막 &raquo;</a>
            {% endif %}
        </span>
    </div>
{% endblock %}
'''

# 세션 2: 사용자 인증 및 접근 제어 - 로그인/로그아웃 기능 구현 ##################################


# bookproject/urls.py
'''
from django.contrib import admin
from django.urls import path, include
from books.views import MainView  # MainView를 임포트
from django.conf import settings
from django.conf.urls.static import static
from django.contrib.auth.views import LoginView, LogoutView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('books/', include('books.urls')),
    path('', MainView.as_view(), name='main'),  # 메인 페이지 URL 패턴
    path('login/', LoginView.as_view(template_name='login.html'), name='login'),
    path('logout/', LogoutView.as_view(next_page='main'), name='logout'),  # 로그아웃 후 홈 화면으로 리다이렉트
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
'''


# templates/login.html 생성
'''
{% extends 'base.html' %}

{% block title %}로그인{% endblock %}

{% block content %}
    <h2>로그인</h2>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">로그인</button>
    </form>
{% endblock %}
'''



# books/view.py 수정
'''
from django.views.generic import TemplateView,DetailView,ListView
from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Book
from .forms import BookForm
from django.contrib.auth.mixins import LoginRequiredMixin

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
    paginate_by = 2  # 한 페이지에 1개의 항목을 표시 (추가)
    
    def get_queryset(self):
        # 기본 쿼리셋을 가져옵니다.
        queryset = super().get_queryset()

        # 검색어를 처리합니다.
        query = self.request.GET.get('q')
        if query:
            queryset = queryset.filter(title__icontains=query)

        # 정렬 조건을 처리합니다.
        sort = self.request.GET.get('sort')
        if sort == 'title':
            queryset = queryset.order_by('title')
        elif sort == 'author':
            queryset = queryset.order_by('author')
        elif sort == 'date':
            queryset = queryset.order_by('publication_date')

        # 필터링 조건을 처리합니다.
        genre = self.request.GET.get('genre')
        year = self.request.GET.get('year')
        if genre:
            queryset = queryset.filter(genre=genre)  # 장르로 필터링
        if year:
            queryset = queryset.filter(publication_date__year=year)  # 출판 연도로 필터링

        return queryset
    
class MainView(TemplateView):
    template_name = 'main.html'  # 메인 페이지로 사용할 템플릿 파일 지정    
    
class BookDetailView(DetailView):
    model = Book  # 사용할 모델 지정
    template_name = 'books/book_detail.html'  # 템플릿 파일 지정
    context_object_name = 'book'  # 템플릿에서 사용할 컨텍스트 이름 지정 

class BookCreateView(LoginRequiredMixin, CreateView):
    model = Book
    form_class = BookForm
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('books:book_list')       

class BookUpdateView(LoginRequiredMixin, UpdateView):
    model = Book  # 수정할 모델
    form_class = BookForm  # 사용할 폼 클래스
    template_name = 'books/book_form.html'  # 템플릿 파일
    # success_url = reverse_lazy('books:book_list')  # 수정 후 리다이렉트할 URL    
    
    def get_success_url(self):
        return reverse_lazy('books:book_detail', kwargs={'pk': self.object.pk})
    
class BookDeleteView(LoginRequiredMixin, DeleteView):
    model = Book
    template_name = 'books/book_confirm_delete.html'
    success_url = reverse_lazy('books:book_list')    
'''

# templates/base.html 수정
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
                {% if user.is_authenticated %}
                    <li>환영합니다, {{ user.username }}님!</li>
                    <li><a href="{% url 'logout' %}">로그아웃</a></li>
                {% else %}
                    <li><a href="{% url 'login' %}">로그인</a></li>
                {% endif %}
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

# bookproject/urls.py
'''
from django.contrib import admin
from django.urls import path, include
from books.views import MainView  # MainView를 임포트
from django.conf import settings
from django.conf.urls.static import static
from django.contrib.auth.views import LoginView, LogoutView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('books/', include('books.urls')),
    path('', MainView.as_view(), name='main'),  # 메인 페이지 URL 패턴
    path('login/', LoginView.as_view(template_name='login.html', redirect_authenticated_user=True), name='login'),
    path('logout/', LogoutView.as_view(next_page='main'), name='logout'),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
'''

# templates/login.html 수정
'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>로그인</title>
</head>
<body>
    <h1>로그인</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="hidden" name="next" value="{% url 'main' %}">
        <button type="submit">로그인</button>
    </form>
</body>
</html>
'''

# templates/base.html 수정
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
                {% if user.is_authenticated %}
                    <li>환영합니다, {{ user.username }}님!</li>
                    <li>
                        <form id="logout-form" action="{% url 'logout' %}" method="post" style="display: inline;">
                            {% csrf_token %}
                            <button type="submit" style="background: none; border: none; padding: 0; cursor: pointer;">
                                로그아웃
                            </button>
                        </form>
                    </li>
                {% else %}
                    <li><a href="{% url 'login' %}">로그인</a></li>
                {% endif %}
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


# 세션 3: 사용자 등록 기능 구현 - 회원가입 기능 및 사용자 관리 #################

# books/forms.py 수정
'''
from django import forms
from .models import Book
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User


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
        
class CustomUserCreationForm(UserCreationForm):
    email = forms.EmailField(required=True)

    class Meta:
        model = User
        fields = ("username", "email", "password1", "password2")

    def save(self, commit=True):
        user = super().save(commit=False)
        user.email = self.cleaned_data["email"]
        if commit:
            user.save()
        return user        
'''

# books/views.py 수정

'''
from django.views.generic import TemplateView,DetailView,ListView
from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Book
from .forms import BookForm
from django.contrib.auth.mixins import LoginRequiredMixin
from .forms import CustomUserCreationForm

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
    paginate_by = 2  # 한 페이지에 1개의 항목을 표시 (추가)
    
    def get_queryset(self):
        # 기본 쿼리셋을 가져옵니다.
        queryset = super().get_queryset()

        # 검색어를 처리합니다.
        query = self.request.GET.get('q')
        if query:
            queryset = queryset.filter(title__icontains=query)

        # 정렬 조건을 처리합니다.
        sort = self.request.GET.get('sort')
        if sort == 'title':
            queryset = queryset.order_by('title')
        elif sort == 'author':
            queryset = queryset.order_by('author')
        elif sort == 'date':
            queryset = queryset.order_by('publication_date')

        # 필터링 조건을 처리합니다.
        genre = self.request.GET.get('genre')
        year = self.request.GET.get('year')
        if genre:
            queryset = queryset.filter(genre=genre)  # 장르로 필터링
        if year:
            queryset = queryset.filter(publication_date__year=year)  # 출판 연도로 필터링

        return queryset
    
class MainView(TemplateView):
    template_name = 'main.html'  # 메인 페이지로 사용할 템플릿 파일 지정    
    
class BookDetailView(DetailView):
    model = Book  # 사용할 모델 지정
    template_name = 'books/book_detail.html'  # 템플릿 파일 지정
    context_object_name = 'book'  # 템플릿에서 사용할 컨텍스트 이름 지정 

class BookCreateView(LoginRequiredMixin, CreateView):
    model = Book
    form_class = BookForm
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('books:book_list')       

class BookUpdateView(LoginRequiredMixin, UpdateView):
    model = Book  # 수정할 모델
    form_class = BookForm  # 사용할 폼 클래스
    template_name = 'books/book_form.html'  # 템플릿 파일
    # success_url = reverse_lazy('books:book_list')  # 수정 후 리다이렉트할 URL    
    
    def get_success_url(self):
        return reverse_lazy('books:book_detail', kwargs={'pk': self.object.pk})
    
class BookDeleteView(LoginRequiredMixin, DeleteView):
    model = Book
    template_name = 'books/book_confirm_delete.html'
    success_url = reverse_lazy('books:book_list')    
    
class SignUpView(CreateView):
    form_class = CustomUserCreationForm
    template_name = 'registration/signup.html'
    success_url = reverse_lazy('login') 
'''



# templates/registration/signup.html 생성
'''
{% extends 'base.html' %}

{% block title %}회원가입{% endblock %}

{% block content %}
<h1>회원가입</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">가입하기</button>
</form>
<p>이미 계정이 있으신가요? <a href="{% url 'login' %}">로그인</a></p>
{% endblock %}
'''

# bookproject/urls.py 수정
'''
from django.contrib import admin
from django.urls import path, include
from books.views import MainView,SignUpView  # MainView를 임포트
from django.conf import settings
from django.conf.urls.static import static
from django.contrib.auth.views import LoginView, LogoutView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('books/', include('books.urls')),
    path('', MainView.as_view(), name='main'),  # 메인 페이지 URL 패턴
    path('login/', LoginView.as_view(template_name='login.html', redirect_authenticated_user=True), name='login'),
    path('logout/', LogoutView.as_view(next_page='main'), name='logout'),
    path('signup/', SignUpView.as_view(), name='signup'),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
'''

# books/base.html 수정
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
                {% if user.is_authenticated %}
                    <li>환영합니다, {{ user.username }}님!</li>
                    <li>
                        <form id="logout-form" action="{% url 'logout' %}" method="post" style="display: inline;">
                            {% csrf_token %}
                            <button type="submit" style="background: none; border: none; padding: 0; cursor: pointer;">
                                로그아웃
                            </button>
                        </form>
                    </li>
                {% else %}
                    <li><a href="{% url 'login' %}">로그인</a></li>
                    <li><a href="{% url 'signup' %}">회원가입</a></li>
                {% endif %}
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
# 세션 4: 고급 CBV 사용법 - 접근 제어 및 권한 관리#########


books/views.py
'''
from django.views.generic import TemplateView,DetailView,ListView
from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Book
from .forms import BookForm
from django.contrib.auth.mixins import LoginRequiredMixin,  PermissionRequiredMixin
from .forms import CustomUserCreationForm

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
    paginate_by = 2  # 한 페이지에 1개의 항목을 표시 (추가)
    
    def get_queryset(self):
        # 기본 쿼리셋을 가져옵니다.
        queryset = super().get_queryset()

        # 검색어를 처리합니다.
        query = self.request.GET.get('q')
        if query:
            queryset = queryset.filter(title__icontains=query)

        # 정렬 조건을 처리합니다.
        sort = self.request.GET.get('sort')
        if sort == 'title':
            queryset = queryset.order_by('title')
        elif sort == 'author':
            queryset = queryset.order_by('author')
        elif sort == 'date':
            queryset = queryset.order_by('publication_date')

        # 필터링 조건을 처리합니다.
        genre = self.request.GET.get('genre')
        year = self.request.GET.get('year')
        if genre:
            queryset = queryset.filter(genre=genre)  # 장르로 필터링
        if year:
            queryset = queryset.filter(publication_date__year=year)  # 출판 연도로 필터링

        return queryset
    
class MainView(TemplateView):
    template_name = 'main.html'  # 메인 페이지로 사용할 템플릿 파일 지정    
    
class BookDetailView(DetailView):
    model = Book  # 사용할 모델 지정
    template_name = 'books/book_detail.html'  # 템플릿 파일 지정
    context_object_name = 'book'  # 템플릿에서 사용할 컨텍스트 이름 지정 

class BookCreateView(LoginRequiredMixin, CreateView):
    model = Book
    form_class = BookForm
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('books:book_list')       

class BookUpdateView(LoginRequiredMixin, UpdateView):
    model = Book  # 수정할 모델
    form_class = BookForm  # 사용할 폼 클래스
    template_name = 'books/book_form.html'  # 템플릿 파일
    # success_url = reverse_lazy('books:book_list')  # 수정 후 리다이렉트할 URL    
    
    def get_success_url(self):
        return reverse_lazy('books:book_detail', kwargs={'pk': self.object.pk})
    
class BookDeleteView(PermissionRequiredMixin, DeleteView):
    model = Book
    template_name = 'books/book_confirm_delete.html'
    success_url = reverse_lazy('books:book_list')
    permission_required = 'books.delete_book'
    
    
class SignUpView(CreateView):
    form_class = CustomUserCreationForm
    template_name = 'registration/signup.html'
    success_url = reverse_lazy('login')  
'''


# 커스텀 Mixin 생성
# books/mixins.py 생성
'''
from django.contrib.auth.mixins import UserPassesTestMixin
from django.core.exceptions import PermissionDenied

class GroupRequiredMixin(UserPassesTestMixin):
    group_name = None

    def test_func(self):
        if self.group_name:
            return self.request.user.groups.filter(name=self.group_name).exists()
        return False

    def handle_no_permission(self):
        raise PermissionDenied("이 작업을 수행할 권한이 없습니다.")
'''

# books/views.py 수정
'''
from django.views.generic import TemplateView,DetailView,ListView
from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Book
from .forms import BookForm
from django.contrib.auth.mixins import LoginRequiredMixin,  PermissionRequiredMixin
from .forms import CustomUserCreationForm
from .mixins import GroupRequiredMixin

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
    paginate_by = 2  # 한 페이지에 1개의 항목을 표시 (추가)
    
    def get_queryset(self):
        # 기본 쿼리셋을 가져옵니다.
        queryset = super().get_queryset()

        # 검색어를 처리합니다.
        query = self.request.GET.get('q')
        if query:
            queryset = queryset.filter(title__icontains=query)

        # 정렬 조건을 처리합니다.
        sort = self.request.GET.get('sort')
        if sort == 'title':
            queryset = queryset.order_by('title')
        elif sort == 'author':
            queryset = queryset.order_by('author')
        elif sort == 'date':
            queryset = queryset.order_by('publication_date')

        # 필터링 조건을 처리합니다.
        genre = self.request.GET.get('genre')
        year = self.request.GET.get('year')
        if genre:
            queryset = queryset.filter(genre=genre)  # 장르로 필터링
        if year:
            queryset = queryset.filter(publication_date__year=year)  # 출판 연도로 필터링

        return queryset
    
class MainView(TemplateView):
    template_name = 'main.html'  # 메인 페이지로 사용할 템플릿 파일 지정    
    
class BookDetailView(DetailView):
    model = Book  # 사용할 모델 지정
    template_name = 'books/book_detail.html'  # 템플릿 파일 지정
    context_object_name = 'book'  # 템플릿에서 사용할 컨텍스트 이름 지정 

class BookCreateView(LoginRequiredMixin, CreateView):
    model = Book
    form_class = BookForm
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('books:book_list')       

class BookUpdateView(LoginRequiredMixin, UpdateView):
    model = Book  # 수정할 모델
    form_class = BookForm  # 사용할 폼 클래스
    template_name = 'books/book_form.html'  # 템플릿 파일
    # success_url = reverse_lazy('books:book_list')  # 수정 후 리다이렉트할 URL    
    
    def get_success_url(self):
        return reverse_lazy('books:book_detail', kwargs={'pk': self.object.pk})
    
class BookDeleteView(PermissionRequiredMixin, DeleteView):
    model = Book
    template_name = 'books/book_confirm_delete.html'
    success_url = reverse_lazy('books:book_list')
    permission_required = 'books.delete_book'
    
    
class SignUpView(CreateView):
    form_class = CustomUserCreationForm
    template_name = 'registration/signup.html'
    success_url = reverse_lazy('login')    
    
class BookUpdateView(GroupRequiredMixin, UpdateView):
    model = Book
    form_class = BookForm
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('books:book_detail')
    group_name = 'Editor'    
'''

# 어드민 페이지에서 그룹, 사용자 추가해서 권한 확인 테스트


# 세션 5: 부트스트랩을 활용한 템플릿 스타일링 - UI/UX 개선 ################################################

# templates/base.html 수정

'''
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}도서 관리 시스템{% endblock %}</title>
    <!-- Bootstrap 5 CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <header>
        <h1>도서 관리 시스템</h1>
        <nav>
            <ul>
                <li><a href="{% url 'main' %}">홈</a></li>
                <li><a href="{% url 'books:book_list' %}">도서 목록</a></li>
                {% if user.is_authenticated %}
                    <li>환영합니다, {{ user.username }}님!</li>
                    <li>
                        <form id="logout-form" action="{% url 'logout' %}" method="post" style="display: inline;">
                            {% csrf_token %}
                            <button type="submit" style="background: none; border: none; padding: 0; cursor: pointer;">
                                로그아웃
                            </button>
                        </form>
                    </li>
                {% else %}
                    <li><a href="{% url 'login' %}">로그인</a></li>
                    <li><a href="{% url 'signup' %}">회원가입</a></li>
                {% endif %}
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

# templates/books/book_list.html 수정
'''
{% extends 'base.html' %}

{% block title %}도서 목록{% endblock %}

{% block content %}
    <h2 class="my-4">도서 목록</h2>

    <!-- 검색 및 정렬 폼 -->
    <form method="get" action="." class="row g-3 mb-4">
        <div class="col-md-3">
            <select name="sort" class="form-select">
                <option value="">정렬 선택</option>
                <option value="title" {% if request.GET.sort == 'title' %}selected{% endif %}>제목</option>
                <option value="author" {% if request.GET.sort == 'author' %}selected{% endif %}>저자명</option>
                <option value="date" {% if request.GET.sort == 'date' %}selected{% endif %}>출판일</option>
            </select>
        </div>
        <div class="col-md-3">
            <!-- 필터링 옵션 추가 -->
            <select name="genre" class="form-select">
                <option value="">장르 선택</option>
                <option value="소설" {% if request.GET.genre == '소설' %}selected{% endif %}>소설</option>
                <option value="비소설" {% if request.GET.genre == '비소설' %}selected{% endif %}>비소설</option>
                <option value="과학" {% if request.GET.genre == '과학' %}selected{% endif %}>과학</option>
            </select>
        </div>
        <div class="col-md-3">
            <select name="year" class="form-select">
                <option value="">출판 연도 선택</option>
                <option value="2024" {% if request.GET.year == '2024' %}selected{% endif %}>2024</option>
                <option value="2023" {% if request.GET.year == '2023' %}selected{% endif %}>2023</option>
                <!-- 추가 연도 옵션 -->
            </select>
        </div>
        <div class="col-md-3">
            <input type="text" name="q" class="form-control" placeholder="도서 제목 검색" value="{{ request.GET.q }}">
        </div>
        <div class="col-md-12">
            <button type="submit" class="btn btn-primary w-100">적용</button>
        </div>
    </form>

    <a href="{% url 'books:book_create' %}" class="btn btn-success mb-4">새 도서 추가</a>
    <ul class="list-group">
        {% for book in books %}
            <li class="list-group-item">
                <strong><a href="{% url 'books:book_detail' book.pk %}">{{ book.title }}</a></strong> - {{ book.author }} ({{ book.publication_date|date:"Y-m-d" }})
            </li>
        {% endfor %}
    </ul>

    <!-- 페이지네이션 링크 추가 -->
    <nav aria-label="Page navigation">
        <ul class="pagination justify-content-center mt-4">
            {% if page_obj.has_previous %}
                <li class="page-item">
                    <a class="page-link" href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page=1">&laquo; 처음</a>
                </li>
                <li class="page-item">
                    <a class="page-link" href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page={{ page_obj.previous_page_number }}">이전</a>
                </li>
            {% endif %}

            <li class="page-item active">
                <span class="page-link">
                    페이지 {{ page_obj.number }} / {{ page_obj.paginator.num_pages }}
                </span>
            </li>

            {% if page_obj.has_next %}
                <li class="page-item">
                    <a class="page-link" href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page={{ page_obj.next_page_number }}">다음</a>
                </li>
                <li class="page-item">
                    <a class="page-link" href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page={{ page_obj.paginator.num_pages }}">마지막 &raquo;</a>
                </li>
            {% endif %}
        </ul>
    </nav>
{% endblock %}
'''

# templates/books/book_detail.html 수정
'''
{% extends 'base.html' %}

{% block title %}{{ book.title }} - 상세 정보{% endblock %}

{% block content %}
    <div class="container mt-4">
        <h2 class="mb-4">{{ book.title }}</h2>
        
        <p><strong>저자:</strong> {{ book.author }}</p>
        <p><strong>출판일:</strong> {{ book.publication_date|date:"Y-m-d" }}</p>
        <p><strong>장르:</strong> {{ book.genre }}</p>
        <p><strong>요약:</strong> {{ book.summary }}</p>

        {% if book.image %}
            <img src="{{ book.image.url }}" alt="{{ book.title }} 이미지" class="img-fluid rounded mt-3 mb-4">
        {% else %}
            <p class="text-muted">이미지가 없습니다.</p>
        {% endif %}

        <div class="btn-group" role="group">
            <a href="{% url 'books:book_update' book.pk %}" class="btn btn-warning">도서 정보 수정</a>
            <a href="{% url 'books:book_delete' book.pk %}" class="btn btn-danger">도서 삭제</a>
            <a href="{% url 'books:book_list' %}" class="btn btn-secondary">도서 목록으로 돌아가기</a>
        </div>
    </div>
{% endblock %}
'''

# 3. signup.html ########################

# books 아래에 다음과 같은 폴더 구조를 작성
'''
books/
├── templatetags/
│   ├── __init__.py
│   └── form_tags.py
'''

# books/templatetags/form_tags.py 생성
'''
from django import template

register = template.Library()

@register.filter(name='add_class')
def add_class(field, css_class):
    return field.as_widget(attrs={"class": css_class})
'''

# templates/registration/signup.html 수정
'''
{% extends 'base.html' %}
{% load form_tags %}  <!-- 필터 로드 -->

{% block title %}회원가입{% endblock %}

{% block content %}
<div class="container mt-5">
    <h1 class="mb-4">회원가입</h1>
    <form method="post" class="needs-validation" novalidate>
        {% csrf_token %}

        {% for field in form %}
            <div class="mb-3">
                {{ field.label_tag }}
                {{ field|add_class:"form-control" }}
                {% if field.help_text %}
                    <small class="form-text text-muted">{{ field.help_text }}</small>
                {% endif %}
                {% for error in field.errors %}
                    <div class="invalid-feedback d-block">{{ error }}</div>
                {% endfor %}
            </div>
        {% endfor %}

        <button type="submit" class="btn btn-primary w-100">가입하기</button>
    </form>
    <p class="mt-3">이미 계정이 있으신가요? <a href="{% url 'login' %}">로그인</a></p>
</div>
{% endblock %}
'''

# 세션 6. (고급기능)TemplateView, RedirectView, 그리고 FormView의 활용 ######################

# books/models.py 수정
'''
from django.db import models
from django.contrib.auth.models import User
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


class Rental(models.Model):
    book = models.ForeignKey(Book, on_delete=models.CASCADE, verbose_name='대여 도서')
    user = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name='대여자')
    rental_date = models.DateField(auto_now_add=True, verbose_name='대여일')
    return_date = models.DateField(verbose_name='반납일', null=True, blank=True)
    
    def __str__(self):
        return f"{self.user}님이 {self.book}을 대여함"
'''

# 마이그레이션
'''
python manage.py makemigrations
python manage.py migrate
'''


# books/views.py 수정

'''
from django.views.generic import TemplateView,DetailView,ListView,TemplateView
from django.views.generic.edit import CreateView, UpdateView, DeleteView, FormView
from django.urls import reverse_lazy
from .models import Book
from .forms import BookForm, RentalForm
from django.contrib.auth.mixins import LoginRequiredMixin,  PermissionRequiredMixin
from .forms import CustomUserCreationForm
from .mixins import GroupRequiredMixin

# Create your views here.
class BookListView(ListView):
    model = Book # 이 뷰에서 사용할 모델
    template_name = 'books/book_list.html' # 랜더링할 템플릿 파일! 
    context_object_name = 'books' # 컨텍스트 객체 이름! 기본적으로 object_list를 사용함 book_list로 변경
    ordering = ['-publication_date'] # 출판일 기준 내림차순으로 정렬
    paginate_by = 2  # 한 페이지에 1개의 항목을 표시 (추가)
    
    def get_queryset(self):
        # 기본 쿼리셋을 가져옵니다.
        queryset = super().get_queryset()

        # 검색어를 처리합니다.
        query = self.request.GET.get('q')
        if query:
            queryset = queryset.filter(title__icontains=query)

        # 정렬 조건을 처리합니다.
        sort = self.request.GET.get('sort')
        if sort == 'title':
            queryset = queryset.order_by('title')
        elif sort == 'author':
            queryset = queryset.order_by('author')
        elif sort == 'date':
            queryset = queryset.order_by('publication_date')

        # 필터링 조건을 처리합니다.
        genre = self.request.GET.get('genre')
        year = self.request.GET.get('year')
        if genre:
            queryset = queryset.filter(genre=genre)  # 장르로 필터링
        if year:
            queryset = queryset.filter(publication_date__year=year)  # 출판 연도로 필터링

        return queryset
    
class MainView(TemplateView):
    template_name = 'main.html'  # 메인 페이지로 사용할 템플릿 파일 지정    
    
class BookDetailView(DetailView):
    model = Book  # 사용할 모델 지정
    template_name = 'books/book_detail.html'  # 템플릿 파일 지정
    context_object_name = 'book'  # 템플릿에서 사용할 컨텍스트 이름 지정 

class BookCreateView(LoginRequiredMixin, CreateView):
    model = Book
    form_class = BookForm
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('books:book_list')       

class BookUpdateView(LoginRequiredMixin, UpdateView):
    model = Book  # 수정할 모델
    form_class = BookForm  # 사용할 폼 클래스
    template_name = 'books/book_form.html'  # 템플릿 파일
    # success_url = reverse_lazy('books:book_list')  # 수정 후 리다이렉트할 URL    
    
    def get_success_url(self):
        return reverse_lazy('books:book_detail', kwargs={'pk': self.object.pk})
    
class BookDeleteView(PermissionRequiredMixin, DeleteView):
    model = Book
    template_name = 'books/book_confirm_delete.html'
    success_url = reverse_lazy('books:book_list')
    permission_required = 'books.delete_book'
    
    
class SignUpView(CreateView):
    form_class = CustomUserCreationForm
    template_name = 'registration/signup.html'
    success_url = reverse_lazy('login')    
    
class BookUpdateView(GroupRequiredMixin, UpdateView):
    model = Book
    form_class = BookForm
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('books:book_detail')
    group_name = 'editor'    
    
class RentalInfoView(TemplateView):
    template_name = 'books/rental_info.html'    
    
class BookRentalView(LoginRequiredMixin, FormView):
    template_name = 'books/book_rental.html'
    form_class = RentalForm
    success_url = reverse_lazy('books:rental_success')  # 네임스페이스와 함께 사용

    def form_valid(self, form):
        rental = form.save(commit=False)
        rental.user = self.request.user  # 로그인된 사용자를 할당
        rental.save()
        return super().form_valid(form)  
'''

# books/urls.py 수정
'''
from django.urls import path
from .views import BookListView, BookDetailView, BookCreateView ,BookUpdateView,BookDeleteView,RentalInfoView,BookRentalView
from django.views.generic import TemplateView



app_name = 'books' # 앱의 네임스페이스를 지정 프로젝트 내에서 이름이 겹치지 않게 하기 위함

urlpatterns = [
    path('', BookListView.as_view(), name='book_list'), # books/로 접속하면 BookListView가 실행되도록 함
    path('<int:pk>/', BookDetailView.as_view(), name = 'book_detail'), # books/숫자/로 접속하면 BookDetailView가 실행되도록 함
    path('new/', BookCreateView.as_view(), name='book_create'),
    path('<int:pk>/edit/', BookUpdateView.as_view(), name='book_update'),
    path('<int:pk>/delete/', BookDeleteView.as_view(), name='book_delete'),
    path('rental-info/', RentalInfoView.as_view(), name='rental_info'),
    path('rental/', BookRentalView.as_view(), name='book_rental'),
    path('rental/success/', TemplateView.as_view(template_name='books/rental_success.html'), name='rental_success'),
]
'''




# templates/books/rental_info.html 생성
'''
{% extends 'base.html' %}

{% block title %}도서 대여 안내{% endblock %}

{% block content %}
<div class="container">
    <h2>도서 대여 안내</h2>
    <p>도서 대여에 관한 정보를 제공합니다. 도서 대여 절차, 이용 시간, 기타 규칙 등을 안내합니다.</p>
</div>
{% endblock %}
'''

# books/forms.py 수정

'''
from django import forms
from .models import Book,Rental
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User


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
        
class CustomUserCreationForm(UserCreationForm):
    email = forms.EmailField(required=True)

    class Meta:
        model = User
        fields = ("username", "email", "password1", "password2")

    def save(self, commit=True):
        user = super().save(commit=False)
        user.email = self.cleaned_data["email"]
        if commit:
            user.save()
        return user        

class RentalForm(forms.ModelForm):
    class Meta:
        model = Rental
        fields = ['book', 'return_date']
        widgets = {
            'return_date': forms.DateInput(attrs={'type': 'date', 'class': 'form-control'}),
        }
'''


#  templates/books/book_rental.html 생성

'''
{% extends 'base.html' %}

{% block title %}도서 대여{% endblock %}

{% block content %}
<div class="container">
    <h2>도서 대여</h2>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="btn btn-primary">대여하기</button>
    </form>
</div>
{% endblock %}

'''


# templates/books/rental_success.html  생성
'''
{% extends 'base.html' %}

{% block title %}대여 성공{% endblock %}

{% block content %}
<div class="container mt-5 text-center">
    <h2>도서 대여가 완료되었습니다!</h2>
    <p class="lead">도서를 잘 읽으시고, 기한 내에 반납해 주세요.</p>
    <a href="{% url 'books:book_list' %}" class="btn btn-primary">도서 목록으로 돌아가기</a>
</div>
{% endblock %}
'''


# templates/books/book_list.html에 도서 대여 링크 추가
'''
{% extends 'base.html' %}

{% block title %}도서 목록{% endblock %}

{% block content %}
    <h2 class="my-4">도서 목록</h2>

    <!-- 검색 및 정렬 폼 -->
    <form method="get" action="." class="row g-3 mb-4">
        <div class="col-md-3">
            <select name="sort" class="form-select">
                <option value="">정렬 선택</option>
                <option value="title" {% if request.GET.sort == 'title' %}selected{% endif %}>제목</option>
                <option value="author" {% if request.GET.sort == 'author' %}selected{% endif %}>저자명</option>
                <option value="date" {% if request.GET.sort == 'date' %}selected{% endif %}>출판일</option>
            </select>
        </div>
        <div class="col-md-3">
            <!-- 필터링 옵션 추가 -->
            <select name="genre" class="form-select">
                <option value="">장르 선택</option>
                <option value="소설" {% if request.GET.genre == '소설' %}selected{% endif %}>소설</option>
                <option value="비소설" {% if request.GET.genre == '비소설' %}selected{% endif %}>비소설</option>
                <option value="과학" {% if request.GET.genre == '과학' %}selected{% endif %}>과학</option>
            </select>
        </div>
        <div class="col-md-3">
            <select name="year" class="form-select">
                <option value="">출판 연도 선택</option>
                <option value="2024" {% if request.GET.year == '2024' %}selected{% endif %}>2024</option>
                <option value="2023" {% if request.GET.year == '2023' %}selected{% endif %}>2023</option>
                <!-- 추가 연도 옵션 -->
            </select>
        </div>
        <div class="col-md-3">
            <input type="text" name="q" class="form-control" placeholder="도서 제목 검색" value="{{ request.GET.q }}">
        </div>
        <div class="col-md-12">
            <button type="submit" class="btn btn-primary w-100">적용</button>
        </div>
    </form>

    <a href="{% url 'books:book_create' %}" class="btn btn-success mb-4">새 도서 추가</a>
    <a href="{% url 'books:book_rental' %}" class="btn btn-success mb-4">도서 대여</a>
    <ul class="list-group">
        {% for book in books %}
            <li class="list-group-item">
                <strong><a href="{% url 'books:book_detail' book.pk %}">{{ book.title }}</a></strong> - {{ book.author }} ({{ book.publication_date|date:"Y-m-d" }})
            </li>
        {% endfor %}
    </ul>

    <!-- 페이지네이션 링크 추가 -->
    <nav aria-label="Page navigation">
        <ul class="pagination justify-content-center mt-4">
            {% if page_obj.has_previous %}
                <li class="page-item">
                    <a class="page-link" href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page=1">&laquo; 처음</a>
                </li>
                <li class="page-item">
                    <a class="page-link" href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page={{ page_obj.previous_page_number }}">이전</a>
                </li>
            {% endif %}

            <li class="page-item active">
                <span class="page-link">
                    페이지 {{ page_obj.number }} / {{ page_obj.paginator.num_pages }}
                </span>
            </li>

            {% if page_obj.has_next %}
                <li class="page-item">
                    <a class="page-link" href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page={{ page_obj.next_page_number }}">다음</a>
                </li>
                <li class="page-item">
                    <a class="page-link" href="?q={{ request.GET.q }}&sort={{ request.GET.sort }}&genre={{ request.GET.genre }}&year={{ request.GET.year }}&page={{ page_obj.paginator.num_pages }}">마지막 &raquo;</a>
                </li>
            {% endif %}
        </ul>
    </nav>
{% endblock %}
'''

```
{% endraw %}