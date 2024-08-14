{% raw %}
```python

# 세팅1
'''
python -m venv myenv

.\venv\Scripts\activate

'''

# 세팅2
'''
pip install django

pip install openai python-dotenv gunicorn

echo "OPENAI_API_KEY='시크릿 키'" > .env

# .env UTF-8로 변경 필요

django-admin startproject chatgpt_project

cd chatgpt_project

python manage.py startapp chat
'''




# chatgpt_project/settings.py 'chat'추가
'''
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'chat',
]
'''
# chatgpt_project/settings.py api키 추가
'''
import os
from dotenv import load_dotenv

load_dotenv()  # .env 파일에서 환경 변수 로드

# 환경 변수 가져오기
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")

# 올바르게 로드되었는지 확인 (디버깅용)
if OPENAI_API_KEY:
    print("API Key loaded successfully.")
else:
    print("Failed to load API Key.")
'''

# api키 정상적으로 불러오는지 확인
'''
python chatgpt_project/settings.py
'''


# chatgpt_project/urls.py
'''
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('chat.urls')),  # chat 앱의 URL 연결
]
'''

# chat/urls.py
'''
from django.urls import path
from . import views

urlpatterns = [
    path('', views.ChatView.as_view(), name='chat'),  # 기본 URL
]
'''


# chat/views.py
'''
from django.views.generic import TemplateView

class ChatView(TemplateView):
    template_name = "chat/index.html"

'''

# chat/templates/chat/base.html
'''
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>{% block title %}ChatGPT{% endblock %}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-dark bg-dark">
        <a class="navbar-brand" href="#">{% block header %}ChatGPT{% endblock %}</a>
    </nav>
    <div class="container mt-4">
        {% block content %}{% endblock %}
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.1/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>

'''
# chat/templates/chat/index.html

'''
{% extends 'chat/base.html' %}

{% block title %}ChatGPT Integration{% endblock %}

{% block header %}Welcome to the ChatGPT Integration{% endblock %}

{% block content %}
<p>This is a simple chat interface using ChatGPT.</p>
{% endblock %}
'''


# 마이그레이션

'''
python manage.py makemigrations
python manage.py migrate
'''

# 런 서버
'''
python manage.py runserver
'''

# 2. 클래스 기반 뷰(CBV) 개념 소개 및 적용 ############


# chat/forms.py
'''
from django import forms

# label: 사용자에게 보여지는 레이블
    # max_length: 사용자가 입력할 수 있는 최대 길이
    # widget: 사용자 입력창의 디자인을 변경할 수 있는 옵션
    # from-control: 입력창의 크기를 조절하는 클래스
    # placeholder: 입력창에 미리 보여지는 텍스트
class ChatForm(forms.Form):
    user_input = forms.CharField(label = "사용자 입력창", max_length = 100,
                                 widget = forms.TextInput(attrs = {
                                     'class': 'form-control',
                                     'placeholder': '무엇을 도와드릴까요?'
                                 }))
'''

# chat/views.py 수정

'''
from django.views.generic.edit import FormView
from .forms import ChatForm
from openai import OpenAI
from django.conf import settings

# 클라이언트 인스턴스 생성
client = OpenAI(api_key=settings.OPENAI_API_KEY)

def get_completion(prompt, model="gpt-3.5-turbo"):
    try:
        chat_completion = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=100,
            temperature=0.5
        )
        return chat_completion.choices[0].message.content.strip()
    except Exception as e:
        return f"An error occurred: {e}"

class ChatView(FormView):
    template_name = 'chat/index.html'
    form_class = ChatForm
    success_url = '/'  # 폼 제출 후 다시 메인 페이지로 리디렉션

    def form_valid(self, form):
        user_input = form.cleaned_data['user_input']
        result = get_completion(user_input)
        return self.render_to_response(self.get_context_data(result=result))
''' 


# chat/templates/chat/index.html 수정

'''
{% extends 'chat/base.html' %}

{% block title %}ChatGPT Integration{% endblock %}

{% block header %}
    Simple Chat Interface
{% endblock %}

{% block content %}
    <p>This is a simple chat interface using ChatGPT.</p>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="btn btn-primary">Send</button>
    </form>

    {% if result %}
        <div class="alert alert-info mt-3">
            <strong>ChatGPT:</strong> {{ result }}
        </div>
    {% endif %}
{% endblock %}

'''

# 3. ChatGPT API 연동 기능 및 로그인 로그아웃 기능 구현 #######################

#  프로젝트 구조
'''
chatgpt_project/
├── chat/
│   ├── migrations/
│   ├── templates/
│   │   ├── chat/
│   │   │   ├── base.html
│   │   │   ├── index.html
│   │   │   └── login.html
│   ├── forms.py
│   ├── urls.py
│   ├── views.py
│   └── models.py
├── chatgpt_project/
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
└── .env
'''

# chatgpt_project/urls.py 수정
'''
from django.contrib import admin
from django.urls import path, include
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('chat.urls')),  # chat 앱의 URL 연결
    path('login/', auth_views.LoginView.as_view(template_name='chat/login.html'), name='login'),
    path('logout/', auth_views.LogoutView.as_view(next_page='/login/'), name='logout'),
]
'''

# chat/templates/chat/login.html 생성

'''
{% extends 'chat/base.html' %}

{% block title %}Login{% endblock %}

{% block header %}
    <h2>Please log in to continue</h2>
{% endblock %}

{% block content %}
    <div class="col-md-6 mx-auto mt-5">
        <form method="post" class="card p-4">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit" class="btn btn-primary w-100 mt-4">Login</button>
        </form>
        
        {% if form.errors %}
            <div class="alert alert-danger mt-3">
                Invalid login credentials, please try again.
            </div>
        {% endif %}
    </div>
{% endblock %}
'''

# chat/views.py 수정
'''
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic.edit import FormView
from .forms import ChatForm
from openai import OpenAI
from django.conf import settings
from django.views.generic import TemplateView


# 클라이언트 인스턴스 생성
client = OpenAI(api_key=settings.OPENAI_API_KEY)

def get_completion(prompt, model="gpt-3.5-turbo"):
    try: # ChatGPT API 호출
        chat_completion = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=100,
            temperature=0.5
        )
        # 응답에서 메시지 추출
        return chat_completion.choices[0].message.content.strip()
    except Exception as e:
        return f"An error occurred: {e}"

class ChatView(LoginRequiredMixin,FormView):
    template_name = 'chat/index.html'
    form_class = ChatForm
    success_url = '/'  # 폼 제출 후 다시 메인 페이지로 리디렉션

    def form_valid(self, form):
        user_input = form.cleaned_data['user_input']
        result = get_completion(user_input)
        return self.render_to_response(self.get_context_data(result=result))


class HomeView(TemplateView):
    template_name = 'chat/home.html'
'''

# chat/templates/chat/index.html 수정
'''
{% extends 'chat/base.html' %}

{% block title %}ChatGPT Integration{% endblock %}

{% block header %}
    Simple Chat Interface
{% endblock %}

{% block content %}
    <p>This is a simple chat interface using ChatGPT.</p>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="btn btn-primary">Send</button>
    </form>

    {% if messages %}
        {% for message in messages %}
            <div class="alert alert-danger mt-3">{{ message }}</div>
        {% endfor %}
    {% endif %}

    {% if result %}
        <div class="alert alert-info mt-3">
            <strong>ChatGPT:</strong> {{ result }}
        </div>
    {% endif %}
{% endblock %}

'''

# chat/templates/chat/home.html 생성
'''
{% extends 'chat/base.html' %}

{% block title %}Home{% endblock %}

{% block header %}
    <h2>Welcome to ChatGPT Service</h2>
{% endblock %}

{% block content %}
    <div class="text-center">
        {% if user.is_authenticated %}
            <p>Welcome, {{ user.username }}! You can now access the <a href="{% url 'chat' %}">ChatGPT service</a>.</p>
            <a href="{% url 'logout' %}" class="btn btn-secondary mt-3">Logout</a>
        {% else %}
            <p>You need to <a href="{% url 'login' %}">log in</a> to access the ChatGPT service.</p>
            <a href="{% url 'login' %}" class="btn btn-primary mt-3">Login</a>
        {% endif %}
    </div>
{% endblock %}
'''

# chatgpt_project/urls.py 수정
'''
from django.contrib import admin
from django.urls import path, include
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('chat.urls')),  # chat 앱의 URL 연결
    path('login/', auth_views.LoginView.as_view(template_name='chat/login.html'), name='login'),  # 로그인 페이지
    path('logout/', auth_views.LogoutView.as_view(next_page='/login/'), name='logout'),  # 로그아웃 후 리디렉션
]
'''

# chat/urls.py 수정
'''
from django.urls import path
from . import views

urlpatterns = [
    path('', views.ChatView.as_view(), name='chat'),  # 기본 URL
    path('chat/', views.ChatView.as_view(), name='chat'),  # ChatGPT 서비스 페이지
]
'''

# chat/templates/chat/home.html 수정
'''
{% extends 'chat/base.html' %}

{% block title %}Home{% endblock %}

{% block header %}
    <h2>Welcome to ChatGPT Service</h2>
{% endblock %}

{% block content %}
    <div class="text-center">
        {% if user.is_authenticated %}
            <p>Welcome, {{ user.username }}! You can now access the <a href="{% url 'chat' %}">ChatGPT service</a>.</p>

            <!-- 로그아웃 버튼을 POST 요청으로 처리 -->
            <form method="post" action="{% url 'logout' %}">
                {% csrf_token %}
                <button type="submit" class="btn btn-secondary mt-3">Logout</button>
            </form>
        {% else %}
            <p>You need to <a href="{% url 'login' %}">log in</a> to access the ChatGPT service.</p>
            <a href="{% url 'login' %}" class="btn btn-primary mt-3">Login</a>
        {% endif %}
    </div>
{% endblock %}
'''

# chat/urls.py 수정
'''
from django.urls import path
from . import views


urlpatterns = [
    path('', views.HomeView.as_view(), name='home'),  # 홈 페이지
    path('chat/', views.ChatView.as_view(), name='chat'),  # ChatGPT 서비스 페이지
]
'''

chatgpt_project/settings.py 내용 추가
'''
# 로그인 후 리디렉션할 URL 설정
LOGIN_REDIRECT_URL = '/'
'''

# 4. UI 개선 및 오류 처리 ######################

# chat/templates/chat/base.html 수정
'''
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>{% block title %}ChatGPT{% endblock %}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.1/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body {
            background-color: #f0fff0;
        }
        .navbar {
            background-color: white !important;
            border-bottom: 1px solid #e0e0e0;
        }
        .navbar-brand {
            color: black !important;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <nav class="navbar navbar-light">
        <div class="container">
            <a class="navbar-brand" href="#">{% block header %}두뇌에 터보엔진을 달고 지식을 습득하세요{% endblock %}</a>
        </div>
    </nav>
    <div class="container mt-4">
        {% block content %}{% endblock %}
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.1/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
'''

# chat/templates/chat/index.html 수정
'''
{% extends 'chat/base.html' %}
{% block title %}ChatGPT Integration{% endblock %}
{% block header %}Simple Chat Interface{% endblock %}
{% block content %}
<div class="search-container mb-4">
    <input type="text" class="form-control search-input" placeholder="찾고 싶은 지식을 입력하세요">
    <div class="category-buttons mt-2">
        <button class="btn btn-outline-secondary">검색</button>
        <button class="btn btn-warning">영상</button>
        <button class="btn btn-outline-secondary">웹사이트</button>
        <button class="btn btn-outline-secondary">PDF</button>
        <button class="btn btn-outline-secondary">녹음</button>
        <button class="btn btn-outline-secondary">텍스트</button>
    </div>
</div>

<p>This is a simple chat interface using ChatGPT.</p>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit" class="btn btn-primary">Send</button>
</form>
{% if messages %}
    {% for message in messages %}
        <div class="alert alert-danger mt-3">{{ message }}</div>
    {% endfor %}
{% endif %}
{% if result %}
    <div class="alert alert-info mt-3">
        <strong>ChatGPT:</strong> {{ result }}
    </div>
{% endif %}
{% endblock %}
'''

# 5. 프롬프팅을 활용한 기능 구현 ####################

chat/forms.py 수정
'''
from django import forms

# label: 사용자에게 보여지는 레이블
    # max_length: 사용자가 입력할 수 있는 최대 길이
    # widget: 사용자 입력창의 디자인을 변경할 수 있는 옵션
    # from-control: 입력창의 크기를 조절하는 클래스
    # placeholder: 입력창에 미리 보여지는 텍스트
    
class ChatForm(forms.Form):
    text_input = forms.CharField(
        label='글 내용 요약',
        max_length=1000,
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'placeholder': '요약할 내용을 입력하세요...',
            'rows': 4,
        }),
        required=False
    )
    file_input = forms.FileField(
        label='PDF 파일 요약',
        widget=forms.ClearableFileInput(attrs={'class': 'form-control'}),
        required=False
    )
'''

# chat/views.py
'''
from django.views.generic.edit import FormView
from .forms import ChatForm
from openai import OpenAI
from django.conf import settings
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView

# 클라이언트 인스턴스 생성
client = OpenAI(api_key=settings.OPENAI_API_KEY)

def get_completion(prompt, model="gpt-3.5-turbo"):
    try:
        chat_completion = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=100,
            temperature=0.5
        )
        return chat_completion.choices[0].message.content.strip()
    except Exception as e:
        return f"An error occurred: {e}"

def generate_prompt(text_input):
    if text_input:
        return f"다음 내용을 10줄 이내로 요약해줘: {text_input}"
    else:
        return "요약할 내용이 없습니다."

class ChatView(LoginRequiredMixin, FormView):
    template_name = 'chat/index.html'
    form_class = ChatForm
    success_url = '/'  # 폼 제출 후 다시 메인 페이지로 리디렉션

    def form_valid(self, form):
        text_input = form.cleaned_data.get('text_input')
        prompt = generate_prompt(text_input)
        result = get_completion(prompt)
        return self.render_to_response(self.get_context_data(result=result))

class HomeView(TemplateView):
    template_name = 'chat/home.html'
'''


# chat/templates/chat/index.html 수정
'''
{% extends 'chat/base.html' %}
{% block title %}ChatGPT Integration{% endblock %}
{% block header %}Simple Chat Interface{% endblock %}
{% block content %}
<div class="search-container mb-4">
    <input type="text" class="form-control search-input" placeholder="찾고 싶은 지식을 입력하세요">
    <div class="category-buttons mt-2">
        <button class="btn btn-outline-secondary">검색</button>
        <button class="btn btn-warning">영상</button>
        <button class="btn btn-outline-secondary">웹사이트</button>
        <button class="btn btn-outline-secondary">PDF</button>
        <button class="btn btn-outline-secondary">녹음</button>
        <button class="btn btn-outline-secondary">텍스트</button>
    </div>
</div>

<p>This is a simple chat interface using ChatGPT.</p>
    <form method="post" class="mb-4">
        {% csrf_token %}
        <div class="form-group">
            {{ form.text_input.label_tag }}
            {{ form.text_input }}
        </div>
        <button type="submit" class="btn btn-primary w-100 mt-3">요약 생성</button>
    </form>

    {% if messages %}
        {% for message in messages %}
            <div class="alert alert-danger">{{ message }}</div>
        {% endfor %}
    {% endif %}

    {% if result %}
        <div class="alert alert-info mt-3">
            <strong>Summary:</strong> {{ result }}
        </div>
{% endif %}
{% endblock %}

'''

# PyMuPDF설치
'''
pip install pymupdf
'''

# chat/forms.py 수정
'''
from django import forms

# label: 사용자에게 보여지는 레이블
    # max_length: 사용자가 입력할 수 있는 최대 길이
    # widget: 사용자 입력창의 디자인을 변경할 수 있는 옵션
    # from-control: 입력창의 크기를 조절하는 클래스
    # placeholder: 입력창에 미리 보여지는 텍스트
    
class ChatForm(forms.Form):
    text_input = forms.CharField(
        label='글 내용 요약',
        max_length=1000,
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'placeholder': '요약할 내용을 입력하세요...',
            'rows': 4,
        }),
        required=False
    )
    file_input = forms.FileField(
        label='PDF 파일 요약',
        widget=forms.ClearableFileInput(attrs={'class': 'form-control'}),
        required=False
    )
'''

# chat/views.py 수정
'''
from django.views.generic.edit import FormView
from .forms import ChatForm
from openai import OpenAI
from django.conf import settings
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView
import fitz  # PyMuPDF 라이브러리


# 클라이언트 인스턴스 생성
client = OpenAI(api_key=settings.OPENAI_API_KEY)

def get_completion(prompt, model="gpt-3.5-turbo"):
    try:
        chat_completion = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=100,
            temperature=0.5
        )
        return chat_completion.choices[0].message.content.strip()
    except Exception as e:
        return f"An error occurred: {e}"

# 이 부분 수정
def generate_prompt(text_input=None, file_input=None):
    if text_input:
        return f"다음 내용을 10줄 이내로 요약해줘: {text_input}"
    elif file_input:
        file_content = extract_text_from_pdf(file_input)
        return f"다음 PDF 파일 내용을 10줄 이내로 요약해줘: {file_content}"
    else:
        return "요약할 내용이 없습니다."
    
# 이 부분도 수정
class ChatView(LoginRequiredMixin, FormView):
    template_name = 'chat/index.html'
    form_class = ChatForm
    success_url = '/'  # 폼 제출 후 다시 메인 페이지로 리디렉션

    def form_valid(self, form):
        text_input = form.cleaned_data.get('text_input')
        file_input = form.cleaned_data.get('file_input')

        prompt = generate_prompt(text_input, file_input)
        result = get_completion(prompt)
        return self.render_to_response(self.get_context_data(result=result))

class HomeView(TemplateView):
    template_name = 'chat/home.html'


def extract_text_from_pdf(pdf_file):
    """
    PDF 파일에서 텍스트를 추출하는 함수.
    """
    text = ""
    with fitz.open(stream=pdf_file.read(), filetype="pdf") as doc:
        for page in doc:
            text += page.get_text()
    return text
'''

# chat/templates/chat/index.html 수정
'''
{% extends 'chat/base.html' %}

{% block title %}ChatGPT Integration{% endblock %}

{% block header %}
    <h2>Advanced Chat Interface</h2>
{% endblock %}

{% block content %}
    <div class="row mb-4">
        <div class="col-md-8 mx-auto">
            <div class="search-container mb-3">
                <input type="text" class="form-control search-input" placeholder="찾고 싶은 지식을 입력하세요">
            </div>
            <div class="category-buttons mb-3">
                <button class="btn btn-outline-secondary">검색</button>
                <button class="btn btn-warning">영상</button>
                <button class="btn btn-outline-secondary">웹사이트</button>
                <button class="btn btn-outline-secondary">PDF</button>
                <button class="btn btn-outline-secondary">녹음</button>
                <button class="btn btn-outline-secondary">텍스트</button>
            </div>
            <p>This is an advanced chat interface using ChatGPT with file upload capability.</p>
            <form method="post" enctype="multipart/form-data" class="mb-4">
                {% csrf_token %}
                <div class="form-group mb-3">
                    {{ form.text_input.label_tag }}
                    {{ form.text_input }}
                </div>
                <div class="form-group mb-3">
                    {{ form.file_input.label_tag }}
                    {{ form.file_input }}
                </div>
                <button type="submit" class="btn btn-primary w-100">요약 생성</button>
            </form>
        </div>
    </div>

    {% if result %}
        <div class="row">
            <div class="col-md-8 mx-auto">
                <div class="alert alert-info mt-3">
                    <strong>Summary:</strong> {{ result }}
                </div>
            </div>
        </div>
    {% endif %}
{% endblock %}
'''

# pip install youtube-transcript-api 설치
'''
pip install youtube-transcript-api
'''

# chat/forms.py
'''
from django import forms

# label: 사용자에게 보여지는 레이블
    # max_length: 사용자가 입력할 수 있는 최대 길이
    # widget: 사용자 입력창의 디자인을 변경할 수 있는 옵션
    # from-control: 입력창의 크기를 조절하는 클래스
    # placeholder: 입력창에 미리 보여지는 텍스트
    
class ChatForm(forms.Form):
    text_input = forms.CharField(
        label='글 내용 요약',
        max_length=1000,
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'placeholder': '요약할 내용을 입력하세요...',
            'rows': 4,
        }),
        required=False
    )
    file_input = forms.FileField(
        label='PDF 파일 요약',
        widget=forms.ClearableFileInput(attrs={'class': 'form-control'}),
        required=False
    )
    youtube_url = forms.URLField(
        label='YouTube URL 요약',
        widget=forms.URLInput(attrs={
            'class': 'form-control',
            'placeholder': 'YouTube URL을 입력하세요...'
        }),
        required=False
    )
'''

# chat/views.py 수정
'''
from django.views.generic.edit import FormView
from .forms import ChatForm
from openai import OpenAI
from django.conf import settings
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView
import fitz  # PyMuPDF 라이브러리
from youtube_transcript_api import YouTubeTranscriptApi


# 클라이언트 인스턴스 생성
client = OpenAI(api_key=settings.OPENAI_API_KEY)

def get_completion(prompt, model="gpt-3.5-turbo"):
    try:
        chat_completion = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=100,
            temperature=0.5
        )
        return chat_completion.choices[0].message.content.strip()
    except Exception as e:
        return f"An error occurred: {e}"

# 이 부분 수정
def generate_prompt(text_input=None, file_input=None, youtube_url=None):
    if text_input:
        return f"다음 내용을 10줄 이내로 요약해줘: {text_input}"
    elif file_input:
        file_content = extract_text_from_pdf(file_input)
        return f"다음 PDF 파일 내용을 10줄 이내로 요약해줘: {file_content}"
    elif youtube_url:
        youtube_content = extract_text_from_youtube(youtube_url)
        return f"다음 YouTube 동영상 내용을 10줄 이내로 요약해줘: {youtube_content}"
    else:
        return "요약할 내용이 없습니다."
    
# 이 부분 수정
class ChatView(LoginRequiredMixin, FormView):
    template_name = 'chat/index.html'
    form_class = ChatForm
    success_url = '/'  # 폼 제출 후 다시 메인 페이지로 리디렉션

    def form_valid(self, form):
        text_input = form.cleaned_data.get('text_input')
        file_input = form.cleaned_data.get('file_input')
        youtube_url = form.cleaned_data.get('youtube_url')

        prompt = generate_prompt(text_input, file_input, youtube_url)
        result = get_completion(prompt)
        return self.render_to_response(self.get_context_data(result=result))

class HomeView(TemplateView):
    template_name = 'chat/home.html'


def extract_text_from_pdf(pdf_file):
    """
    PDF 파일에서 텍스트를 추출하는 함수.
    """
    text = ""
    with fitz.open(stream=pdf_file.read(), filetype="pdf") as doc:
        for page in doc:
            text += page.get_text()
    return text

def extract_text_from_youtube(url):
    """
    YouTube URL에서 동영상의 스크립트를 추출하는 함수.
    """
    # Youtube URL에서 자막(스크립트를)을 추출하는 함수
    # http://www.youtube.com/watch?v=
    video_id = url.split('v=')[-1] # Youtube URL에서  video_id 추출
    # get_transcript 해당하는 비디오 ID를 기반으로 동영상 자막을 가져오게 됨
    transcript = YouTubeTranscriptApi.get_transcript(video_id)
    # 자막 텍스트를 하나의 문자열로 결합하는 과정 -> 리스트컴프리핸션! 공백으로 연결해서 하나의 텍스트로 만들어줌
    transcript_text = ' '.join([item['text'] for item in transcript])
    return transcript_text
'''

# chat/templates/chat/index.html수정
'''
{% extends 'chat/base.html' %}

{% block title %}ChatGPT Integration{% endblock %}

{% block header %}
    <h2>Advanced Chat Interface</h2>
{% endblock %}

{% block content %}
<div class="container">
    <div class="row mb-4">
        <div class="col-md-8 mx-auto">
            <div class="search-container mb-3">
                <input type="text" class="form-control search-input" placeholder="찾고 싶은 지식을 입력하세요">
            </div>
            <p>This is an advanced chat interface using ChatGPT with file upload and YouTube URL support.</p>
            <form method="post" enctype="multipart/form-data" class="mb-4">
                {% csrf_token %}
                <div class="form-group mb-3">
                    {{ form.text_input.label_tag }}
                    {{ form.text_input }}
                </div>
                <div class="form-group mb-3">
                    {{ form.file_input.label_tag }}
                    {{ form.file_input }}
                </div>
                <div class="form-group mb-3">
                    {{ form.youtube_url.label_tag }}
                    {{ form.youtube_url }}
                </div>
                <button type="submit" class="btn btn-primary w-100">요약 생성</button>
            </form>
        </div>
    </div>

    {% if result %}
        <div class="row">
            <div class="col-md-8 mx-auto">
                <div class="alert alert-info mt-3">
                    <strong>Summary:</strong> {{ result }}
                </div>
            </div>
        </div>
    {% endif %}
</div>
{% endblock %}
'''

# chat/views.py (번역 함수 추가)
'''
from django.views.generic.edit import FormView
from .forms import ChatForm
from openai import OpenAI
from django.conf import settings
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView
import fitz  # PyMuPDF 라이브러리
from youtube_transcript_api import YouTubeTranscriptApi


# 클라이언트 인스턴스 생성
client = OpenAI(api_key=settings.OPENAI_API_KEY)

def get_completion(prompt, model="gpt-3.5-turbo"):
    try:
        chat_completion = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=100,
            temperature=0.5
        )
        return chat_completion.choices[0].message.content.strip()
    except Exception as e:
        return f"An error occurred: {e}"

# 이 부분 수정
def generate_prompt(text_input=None, file_input=None, youtube_url=None):
    if text_input:
        return f"다음 내용을 10줄 이내로 요약해줘: {text_input}"
    elif file_input:
        file_content = extract_text_from_pdf(file_input)
        return f"다음 PDF 파일 내용을 10줄 이내로 요약해줘: {file_content}"
    elif youtube_url:
        youtube_content = extract_text_from_youtube(youtube_url)
        return f"다음 YouTube 동영상 내용을 10줄 이내로 요약해줘: {youtube_content}"
    else:
        return "요약할 내용이 없습니다."
    
# 이 부분 수정
class ChatView(LoginRequiredMixin, FormView):
    template_name = 'chat/index.html'
    form_class = ChatForm
    success_url = '/'  # 폼 제출 후 다시 메인 페이지로 리디렉션

    def form_valid(self, form):
        text_input = form.cleaned_data.get('text_input')
        file_input = form.cleaned_data.get('file_input')
        youtube_url = form.cleaned_data.get('youtube_url')

        prompt = generate_prompt(text_input, file_input, youtube_url)
        summary_result = get_completion(prompt)

        # 영어로 출력된 요약을 한글로 번역
        translation_result = translate_to_korean(summary_result)

        return self.render_to_response(self.get_context_data(
            summary_result=summary_result,
            translation_result=translation_result
        ))

class HomeView(TemplateView):
    template_name = 'chat/home.html'


def extract_text_from_pdf(pdf_file):
    """
    PDF 파일에서 텍스트를 추출하는 함수.
    """
    text = ""
    with fitz.open(stream=pdf_file.read(), filetype="pdf") as doc:
        for page in doc:
            text += page.get_text()
    return text

def extract_text_from_youtube(url):
    """
    YouTube URL에서 동영상의 스크립트를 추출하는 함수.
    """
    # Youtube URL에서 자막(스크립트를)을 추출하는 함수
    # http://www.youtube.com/watch?v=
    video_id = url.split('v=')[-1] # Youtube URL에서  video_id 추출
    # get_transcript 해당하는 비디오 ID를 기반으로 동영상 자막을 가져오게 됨
    transcript = YouTubeTranscriptApi.get_transcript(video_id)
    # 자막 텍스트를 하나의 문자열로 결합하는 과정 -> 리스트컴프리핸션! 공백으로 연결해서 하나의 텍스트로 만들어줌
    transcript_text = ' '.join([item['text'] for item in transcript])
    return transcript_text

def translate_to_korean(text, model="gpt-3.5-turbo"):
    try:
        translation_prompt = f"Translate the following text to Korean: {text}"
        translation_completion = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": translation_prompt}],
            max_tokens=150,
            temperature=0.5
        )
        return translation_completion.choices[0].message.content.strip()
    except Exception as e:
        return f"An error occurred during translation: {e}" 
'''

# chat/templates/chat/index.html 수정
'''
{% extends 'chat/base.html' %}

{% block title %}ChatGPT Integration{% endblock %}

{% block header %}
    <h2>고급 요약 인터페이스</h2>
{% endblock %}

{% block content %}
<div class="container">
    <div class="row mb-4">
        <div class="col-md-8 mx-auto">
            <div class="category-buttons mb-3">
                <button class="btn btn-outline-primary active" data-target="general">일반 요약</button>
                <button class="btn btn-outline-primary" data-target="long">긴글 요약</button>
                <button class="btn btn-outline-primary" data-target="pdf">PDF 요약</button>
                <button class="btn btn-outline-primary" data-target="youtube">유튜브 요약</button>
            </div>
            <form method="post" enctype="multipart/form-data" class="mb-4">
                {% csrf_token %}
                <div id="general-input" class="input-section">
                    <div class="form-group mb-3">
                        {{ form.text_input.label_tag }}
                        {{ form.text_input }}
                    </div>
                </div>
                <div id="long-input" class="input-section" style="display:none;">
                    <div class="form-group mb-3">
                        <label for="long-text">긴 텍스트 입력:</label>
                        <textarea class="form-control" id="long-text" name="long_text" rows="10"></textarea>
                    </div>
                </div>
                <div id="pdf-input" class="input-section" style="display:none;">
                    <div class="form-group mb-3">
                        {{ form.file_input.label_tag }}
                        {{ form.file_input }}
                    </div>
                </div>
                <div id="youtube-input" class="input-section" style="display:none;">
                    <div class="form-group mb-3">
                        {{ form.youtube_url.label_tag }}
                        {{ form.youtube_url }}
                    </div>
                </div>
                <button type="submit" class="btn btn-primary w-100">요약 생성</button>
            </form>
        </div>
    </div>

    {% if summary_result %}
        <div class="row">
            <div class="col-md-8 mx-auto">
                <div class="alert alert-info mt-3">
                    <strong>요약 (영어):</strong> {{ summary_result }}
                </div>
                <div class="alert alert-success mt-3">
                    <strong>요약 (한글):</strong> {{ translation_result }}
                </div>
            </div>
        </div>
    {% endif %}
</div>

<script>
document.addEventListener('DOMContentLoaded', function() {
    const buttons = document.querySelectorAll('.category-buttons button');
    const inputSections = document.querySelectorAll('.input-section');

    buttons.forEach(button => {
        button.addEventListener('click', function() {
            const target = this.getAttribute('data-target');
            
            // Deactivate all buttons and hide all input sections
            buttons.forEach(btn => btn.classList.remove('active'));
            inputSections.forEach(section => section.style.display = 'none');
            
            // Activate clicked button and show corresponding input section
            this.classList.add('active');
            document.getElementById(`${target}-input`).style.display = 'block';
        });
    });
});
</script>
{% endblock %}
'''

# 6. 사용자 검색기록 입력 #######################

# chat/models.py 수정
'''
from django.db import models
from django.contrib.auth.models import User

class SearchHistory(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    url = models.URLField(max_length=200, blank=True, null=True)  # 사용자가 입력한 URL (YouTube 등)
    text_input = models.TextField(blank=True, null=True)  # 사용자가 입력한 텍스트
    file_name = models.CharField(max_length=200, blank=True, null=True)  # 업로드한 파일명
    summary_result = models.TextField(blank=True, null=True)  # 영어 요약 결과
    translation_result = models.TextField(blank=True, null=True)  # 한글 번역 결과
    created_at = models.DateTimeField(auto_now_add=True)  # 레코드 생성 시간

    def __str__(self):
        return f"Search by {self.user.username} on {self.created_at}"
'''

# 마이그레이션
'''
python manage.py makemigrations
python manage.py migrate
'''

# chat/views.py 수정 (SearchHistoryView)
'''
from django.views.generic.edit import FormView
from .forms import ChatForm
from .models import SearchHistory
from openai import OpenAI
from django.conf import settings
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView,ListView

import fitz  # PyMuPDF 라이브러리
from youtube_transcript_api import YouTubeTranscriptApi


# 클라이언트 인스턴스 생성
client = OpenAI(api_key=settings.OPENAI_API_KEY)

def get_completion(prompt, model="gpt-3.5-turbo"):
    try:
        chat_completion = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=100,
            temperature=0.5
        )
        return chat_completion.choices[0].message.content.strip()
    except Exception as e:
        return f"An error occurred: {e}"

# 이 부분 수정
def generate_prompt(text_input=None, file_input=None, youtube_url=None):
    if text_input:
        return f"다음 내용을 10줄 이내로 요약해줘: {text_input}"
    elif file_input:
        file_content = extract_text_from_pdf(file_input)
        return f"다음 PDF 파일 내용을 10줄 이내로 요약해줘: {file_content}"
    elif youtube_url:
        youtube_content = extract_text_from_youtube(youtube_url)
        return f"다음 YouTube 동영상 내용을 10줄 이내로 요약해줘: {youtube_content}"
    else:
        return "요약할 내용이 없습니다."
    
class ChatView(LoginRequiredMixin, FormView):
    template_name = 'chat/index.html'
    form_class = ChatForm
    success_url = '/'  # 폼 제출 후 다시 메인 페이지로 리디렉션

    def form_valid(self, form):
        text_input = form.cleaned_data.get('text_input')
        file_input = form.cleaned_data.get('file_input')
        youtube_url = form.cleaned_data.get('youtube_url')

        prompt = generate_prompt(text_input, file_input, youtube_url)
        summary_result = get_completion(prompt)

        # 영어로 출력된 요약을 한글로 번역
        translation_result = translate_to_korean(summary_result)

        # 검색 기록을 DB에 저장
        SearchHistory.objects.create(
            user=self.request.user,
            url=youtube_url,
            text_input=text_input,
            file_name=file_input.name if file_input else None,
            summary_result=summary_result,
            translation_result=translation_result,
        )

        return self.render_to_response(self.get_context_data(
            summary_result=summary_result,
            translation_result=translation_result
        ))

class HomeView(TemplateView):
    template_name = 'chat/home.html'


def extract_text_from_pdf(pdf_file):
    """
    PDF 파일에서 텍스트를 추출하는 함수.
    """
    text = ""
    with fitz.open(stream=pdf_file.read(), filetype="pdf") as doc:
        for page in doc:
            text += page.get_text()
    return text

def extract_text_from_youtube(url):
    """
    YouTube URL에서 동영상의 스크립트를 추출하는 함수.
    """
    # Youtube URL에서 자막(스크립트를)을 추출하는 함수
    # http://www.youtube.com/watch?v=
    video_id = url.split('v=')[-1] # Youtube URL에서  video_id 추출
    # get_transcript 해당하는 비디오 ID를 기반으로 동영상 자막을 가져오게 됨
    transcript = YouTubeTranscriptApi.get_transcript(video_id)
    # 자막 텍스트를 하나의 문자열로 결합하는 과정 -> 리스트컴프리핸션! 공백으로 연결해서 하나의 텍스트로 만들어줌
    transcript_text = ' '.join([item['text'] for item in transcript])
    return transcript_text

def translate_to_korean(text, model="gpt-3.5-turbo"):
    try:
        translation_prompt = f"Translate the following text to Korean: {text}"
        translation_completion = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": translation_prompt}],
            max_tokens=150,
            temperature=0.5
        )
        return translation_completion.choices[0].message.content.strip()
    except Exception as e:
        return f"An error occurred during translation: {e}"

class SearchHistoryView(LoginRequiredMixin, ListView):
    model = SearchHistory
    template_name = 'chat/search_history.html'
    context_object_name = 'search_histories'

    def get_queryset(self):
        return SearchHistory.objects.filter(user=self.request.user).order_by('-created_at')   
'''

# chat/urls.py
'''
from django.urls import path
from . import views

urlpatterns = [
    path('', views.HomeView.as_view(), name='home'),
    path('chat/', views.ChatView.as_view(), name='chat'),
    path('history/', views.SearchHistoryView.as_view(), name='search_history'),
    # 로그인/로그아웃 URL 등 추가
]
'''

# chat/templates/chat/search_history.html 생성
'''
{% extends 'chat/base.html' %}

{% block title %}Your Search History{% endblock %}

{% block header %}
    Your Search History
{% endblock %}

{% block content %}
    <table class="table table-striped">
        <thead>
            <tr>
                <th>URL</th>
                <th>Text Input</th>
                <th>File Name</th>
                <th>Summary (English)</th>
                <th>Translation (Korean)</th>
                <th>Date</th>
            </tr>
        </thead>
        <tbody>
            {% for history in search_histories %}
                <tr>
                    <td>{{ history.url }}</td>
                    <td>{{ history.text_input }}</td>
                    <td>{{ history.file_name }}</td>
                    <td>{{ history.summary_result }}</td>
                    <td>{{ history.translation_result }}</td>
                    <td>{{ history.created_at }}</td>
                </tr>
            {% empty %}
                <tr>
                    <td colspan="6">No history found.</td>
                </tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
'''
```
{% endraw %}
