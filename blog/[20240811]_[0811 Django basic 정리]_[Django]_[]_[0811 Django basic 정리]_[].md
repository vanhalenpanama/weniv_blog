'''python

models.py

데이터베이스에 어떤 값을 넣을 것이냐

views.py
사용자가 접속했을때 어떤 화면을 전달할 것이냐 지표역활

............

urls.py

from test import name, age

현재 경로안에 있는 test.py파일의 name과 age라는 변수를 활용하겠다.

기본 py 파일 외에는 templates 폴더에 ################

1-1.이미지 불러올때 예시

{% static 'jeju.jpg' %}

1-2. settings.py에서 static 경로에서 가져오기위한 설정

staticfiles_dirs = [BASE_DIR / 'static',]

# 참고

DEBUG = True -->에러 발생 시 상세 화면

ALLOWED_HOSTS = ['*'] 허가한 사용자만 서버에 접속 가능

TIME_ZONE = 'UTC' -> 'Asia/Seoul'


.........................9번


# 블록을 가져와서 붙여지는 html

{% block 블록이름 %}
{% endblock 블록이름%}
html header footer처럼 웹페이지 코드를 나눌 수 있다.



# 지정 블록을 보내주는 html

{% extends '프로젝트명/블록선언한html파일명.html' %}{% block 블록이름 %}

코드들

{% endblock 블록이름%}

............................10번

models.py

from django.db import models

class 게시물(model.Model):
	게시물제목 = CharField(max_length=100)
	좋아요 수 = IntegerField()
	조회 수 = IntegerField()
	내용 = TextField()



class Notice(model.Model):
	title = models.CharField(max_length=100)
	likeCount = models.IntegerField()
	viewCount = models.IntegerField()
	contents = models.TextField()

작성 후 터미널에서
python manage.py makemigrations

# 슈퍼계정 생성

python manage.py createsuperuser


..............................11번

admin.py

from .models import Notice
를 작성해줘야 admin페이지에 Notice가 나옴


#admin 페이지 화면 출력 내용 변경


@admin.register(Notice)
class NoticeAdmin(admin.ModelAdmin):
	list_display = ['title','likeCount']
	list_display_links = ['title', 'likeCount']


......................12번


Model - DB와 연결된 Python Class
Template - 사용자에게 response될 Client View
View - Django에서 처리한 데이터를 Template에게 전달


....................13번


views.py


def sample(request):
	d = {'name':'hojun', 'age':10}
	l = [100,200,300]
	return render(request, 'sample.html',{'value':l})


def sample(request):
	d = Notice.objects.all()
	return render(request, 'sample.html',{'value':l})






sample.html


<h1>{{value}}</h1>
<p>{{value.2.title}}</p>


{% for i in value %}
	<h1>{{i.title}}</h1>
	<p>{{i.contents}}</p>
{% endfor %}


//앞에 번호를 매기고 싶다. forloop.counter
// 0번 부터 번호를 매기고 싶다. forloop.counter0
// 역순으로 번호를 매기고 싶다. forloop.revcounter0



{% for i in value %}
	<h1>{{forloop.counter}}-{{i.title}}</h1>
	<p>{{i.contents}}</p>
	<p>{{i.viewCount}}</p>
	{% if %}
	{% elif %}
	{% else %}
	{% endif %}
{% endfor %}





{% for i in value %}
	<h1>{{forloop.counter}}-{{i.title}}</h1>
	<p>{{i.contents}}</p>
	<p>{{i.viewCount}}</p>
	{% if i.viewCount > 7 %}
		<p>뷰 수가 7보다 많습니다.</p>
	{% elif i.viewCount > 5 %}
		<p>뷰 수가 5보다 많습니다.</p>
	{% else %}
		<p>뷰 수가 5보다 적습니다.</p>
	{% endif %}
{% endfor %}

{% with value = 'hello world' %}
	<h1>{{ value }}</h1>
{% endwith %}


{# hello world #}
{% comment '주석' %}

.........................14번

orm


콘솔창->
python managa.py shell
>>>(파이썬 쉘 커맨드 라인)
from main.models. import Notice
Notice.objects.all()

Notice.objects.all().order_by('-pk')

Notice.objects.all().count()

Notice.objects.get(id=1)

Notice.objects.get(id=1).pk

q=Notice.objects.get(id=1)

q

q.id

q.title

Notice.objects.filter(title='test title 1')

Notice.objects.filter(title__contain='test')


#좋아요 수 3미만
Notice.objects.filter(likeCount__lt=3)

#좋아요 수 3초과
Notice.objects.filter(likeCount__gt=3)


#좋아요 수 3이상
Notice.objects.filter(likeCount__gte=3)


# 데이터 입력
q= Notice.objects.(title='sample',likeCount=100, viewCount=100, contents='hello world')


# commit
q.save()


# 특정 데이터 불러와서 업데이트(걍 덮어씌움)
q = Notice.objects.get(title='sample')
q.content = 'hello world!!!!!!!!!!!!!'
q.save()


# 불러온 데이터 삭제(save()를 하지 않아도 삭제된다)
q.delete()

# QuerySet이라는 데이터 타입을 가지며 dir(q)이런 식으로 전체 명령어를 확인할 수 있다.

...................................15번

sqlite 덤프 뜨기



콘솔창-> 
python manage.py dumpdata main --output data.json


#덤프 가져와서 덮어쓰기

python manage.py loaddata data.json



#들여쓰기 적용

python manage.py dumpdata main --output data.json --indent 4


....................................17번

orm의 ImageField 속성은 pillow라는 모듈에 의존,
없으면 설치해야됨 

ImageField(blank=True) : 사용자가 이미지를 업로드하지 않아도 괜찮음

콘솔창->
source venv/bin/activate
pip install pillow


img =  models.ImageField(upload_to='bb/%Y/%m/%d', blank=true)
create_at = models.DateTimeField(auto_now_add=True, null=True)
modified_at = models.DateTimeField(auto_now=True, null=True)

# auto_now_add=True : 자동으로 insert된 시간이 저장됨
# null=True 널을 허용, 입력 받은 데이터가 없으면 Null값이 들어가는 것을 허용
# upload_to='bb/%Y/%m/%d' 이미지 파일이 업로드되어 media에 올라갈때 media 하위에 /연/월/일 별로 경로에 따른 이미지 파일이 관리되도록 설정 
'''