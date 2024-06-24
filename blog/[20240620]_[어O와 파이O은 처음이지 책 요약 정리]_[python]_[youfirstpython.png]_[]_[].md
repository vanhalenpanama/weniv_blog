########ch2. 변수와 계산

```

###   변수의 이름 규칙
"""
1. 변수 이름은 숫자로 시작할 수 없다.
2. 변수의 이름에는 공백을 포함할 수 없다.
3. 내부 함수와 같은 익숙하고 예약된 키워드는 변수로 사용할 수 없다.
  ex) and or if while for def class
4-1. 카멜 표기법: 변수 이름 첫 글자는 소문자로, 나머지 단어의 첫글자는
   대문자로 ex) myNewCar
4-2. 파스칼 케이스(일반):모든 첫 글자는 대문자로 PascalCase
4-3. 스네이크 케이스:모든 단어는 언더바로 구분 snake_case
5. 파스칼 케이스나 스네이크를 많이 사용하고 카멜 케이스는 잘 사용하지 않는다고 함
   """
###   산술 연산자
"""
곱셈 *
정수 나눗셈 //             ex) 7 // 4 = 1
실수 나눗셈 /              ex) 7 / 4 = 1.75
나머지 %                   ex) 7 % 3 = 1
"""

### 변수의 값과 문자열을 혼합하여 출력하려면 f-문자열을 사용한다
#    ex) print(f"{x}+{y}={result}")
#        print(f"상품의 가격은 {price}원입니다.")
#        print(f"원주율={pi:.2f}") 소수 두번째 자리까지 출력



### 진수 표현과 부동소수점
"""
16진수는 앞에 0x를 붙인다
8진수는 앞에 0o를 붙인다
2진수는 앞에 0b를 붙인다

부동소수점수 표현

   1.23e2 = 1.23 * 10의 2승

"""
## 큰 따옴표안에 큰 따옴표를 넣거나 작은 따옴표안에 작은 따옴표가 올 수 없다. 큰 따옴표 이스케이프 문자  \" 앞이나 뒤에 표시할 따옴표를
""" 사용 예:
print("\"He said, 'Hello!'\"")
 결과 "He said, 'Hello!'"
"""

### print(text * 3) -> text를 한줄로 3번 출력

### 문자열 메소드

"""
len(): 문자열 길이를 계산하는 내장 함수
find("찾고자 하는 문자"): 찾고자하는 단어의 시작 위치의 인덱스를 반환(0부터 시작)

"""


### 이스케이프 문자로 사용하는 백슬레시를 그대로 출력하고자 할때
### print(r\c:\\) 문자열 앞에 r을 붙이면 된다

### 인덱스는 왼쪽 첫번째부터 0으로 시작 맨끝에서는 -1로 시작한다
### 문자열은 인덱스로 일부분만 수정하려고 하면 에러가 발생한다

### 이스케이프 문자
"""
\\ 백슬래시
\' 작은따옴표
\" 큰따옴표
\n 줄바꿈 문자
\t 탭 문자

"""

### 리스트 입력, 최소값, 최대값, 역순으로 변환
"""
리스트=[]

리스트.append(리스트에 입력할 값)

min(리스트)
max(리스트)

리스트.reverse()

tuple(리스트) # 리스트를 튜플!로 변환한다

1 Indexing 𝑂 1 arr[i] 리스트의 특정 인덱스의 값 얻기
2 Storing 𝑂 1 arr[i] = x 리스트의 특정 인덱스에 값 저장하기
3 Append 𝑂 1 arr.append(5) 리스트의 가장 뒤에 데이터 넣기
4 Pop 𝑂 1 arr.pop() 리스트의 가장 뒤에서 원소 꺼내기
5 Length 𝑂 1 len(arr) 리스트의 길이 얻기
6 Clear 𝑂 1 arr.clear() 리스트 내 모든 원소 제거하기
7 Slicing 𝑂 𝑏 − 𝑎 arr[a:b]리스트에서 인덱스 a부터 b-1까지의원소만 꺼내 새 리스트 만들기
8 Extend 𝑂 𝑙𝑒𝑛 𝑜𝑡ℎ𝑒𝑟 arr.extend(other) 기존 리스트에 다른 리스트를 이어 붙이기
9 Insertion 𝑂 𝑁 arr.insert(index, x) 특정 인덱스에 데이터 x를 삽입하기
10 Delete 𝑂 𝑁 del arr[index] 특정 인덱스의 데이터 삭제하기
11 Construction 𝑂 𝑙𝑒𝑛 𝑜𝑡ℎ𝑒𝑟 arr = list(other)다른 자료구조의 원소들을 이용해 리스트로 만들기
12 In 𝑂 𝑁 x in arr: 데이터 x가 리스트에 존재하는지 확인
13 Not in 𝑂 𝑁 x not in arr: 데이터 x가 리스트에 존재하지 않는지 확인
14 Pop 𝑂 𝑁 arr.pop(index) 특정 인덱스의 데이터를 꺼내기 / 단, 가장 뒤 원소를 꺼내는 경우 O(1)
15 Remove 𝑂 𝑁 arr.remove(x) 리스트 내에 존재하는 데이터 x를 삭제
16 Copy 𝑂 𝑁 arr.copy() 리스트를 복제
17 Min 𝑂 𝑁 min(arr) 리스트 내에 존재하는 가장 작은 원소
18 Max 𝑂 𝑁 max(arr) 리스트 내에 존재하는 가장 큰 원소
19 Iteration 𝑂 𝑁 for x in arr: 리스트 내에 존재하는 모든 원소 순회
20 Multiply 𝑂 𝑘 ∗ 𝑁 arr * k 리스트를 k번 반복하여 길게 만들기
21 Sort 𝑂 𝑁𝑙𝑜𝑔𝑁 arr.sort() 리스트 내 존재하는 원소를 정렬

"""

### id(변수) 변수가 가지고 있는 참조값

### abs(변수) 변수의 절대값을 반환

### math.isclose(비교할 변수1,비교할 변수2) 두 변수를 비교하여 true, false를 반환한다. 데이터 타입 bool
"""
import math
a=3
b=3

print(math.isclose(a,b))

"""

### 1e-9 ???

### 변수 = input("콘솔에 표시할 문구") 타이핑해서 변수값을 저장

### 논리 연산자 not x는 x가 참이면 거짓, 거짓이면 참이다


### if문의 and, or절 사용 예시
"""
year = input("연도 입력하시오: ")

if ((year % 4 == 0 and year $ 100 != 0) or (year % 400 == 0)) :
   print(year,"년은 윤년입니다.")
else :
   print(year, "년은 윤년이 아닙니다.")


"""

### if문 elif 사용 예시

"""
score = 91
if score >= 90:
   grade = "A"
elif score >= 80:
   grade = "B"
else :
   grade = "F"

print(score)

"""

### while 무한 반복문 사용 예시

"""
tOrf = "True"

while tOrf :  # 대문자 T로 시작되야 True로 인식
   command = input("f,q중에 입력: ")
   if command.lower() == 'f' :
      print(command)
   elif command.lower() == 'q' :
      break
   else :
      print("잘못된 입력입니다 다시 입력해주세요.")

"""

### in , not in 연산자 사용 예시(true, false 반환)

"""
num = [1,2,3]

print(3 in num)

if 3 in num :
   print("num안에 3이 있습니다")
else :
   print("3이 없습니다.")

"""

### for 반복문 출력을 한 줄에 다 할 것이냐, 각각 출력할 것인가

"""
#리스트의 값마다 출력
 for name in ['철수', '영희'] :
   print("안녕?" + name)

#줄 바꾸지 말고 구분자를 써서 한 줄에 전부 출력

for x in [0,1,2]
   print(x, end=" ")

"""

###  range() 사용 예시

"""
sum = 0

for x in range(10) : # range(10)은 0부터 9까지 정수를 생성한다

for y in range(0, 10) : # 0에서 9까지 반복문을 실행한다

for z in range(0,10,2) : # 0에서 9사이에서 2씩 건너뛰면서 실행한다


"""

### round(c, 2)는 변수 c의 값을 소수점 두 번째 자리까지 출력하는 것을 의미

# isalpha 변수가 알파벳문자인지 확인하고 true false를 반환. 숫자 혼용된 변수는 false
"""
original  = "abㄱㄴㄷ123"

print(original.isalpha())

"""

### randint() 범위 내의 난수 반환. 자바의 mathrandom과 유사

"""
randint(1,6) 1부터 6 사이의 난수를 반환한다.

"""

### random.choice() 변수에서 무작위로 한 글자를 선택하여 반환한다
""" 사용 예시
import random

text = "abcd"
password = ""

for x in range(6) :
   password += random.choice(text)

print(password)
"""


### 함수가 값을 반환하지 않는 경우, None이라는 특별한 값을 반환한다


"""
for _ in range(6) # _는 변수가 필요한 곳이지만 실제로는 사용되지 않을때 사용한다

"""

### 가변 인자 *args, **kwargs 사용 예시
"""
def args_example(*args):
   for arg in args :
      print(arg, end=" ")

args_example(1,2,3)

def kwargs_example(**kwargs):
   for key, value in kwargs.items():
      print(f"{key}: {value}", end=" ")

kwargs_example(a=1, b=2)

"""

### 1.정수와 문자열은 함수로 변경이 불가능한 객체이다
"""
def modify(n):
   n =  n +1

k=10

modify(k)
print(k) #결과는 11이 아닌 10
"""

### 2. 함수내에 global변수를 사용하겠다고 선언하면 변경가능하다.
"""
s="사과가 좋다"

def modify():
   global s
   s="포도는 더 좋다"

modify()
print(s)
"""

### split() 사용 예시 str타입을 list타입으로 변경하여 문자열을 각각 분리 (기본값 공백)
"""
text="공백을, 기준으로 문자열을 분리"

print(text.split())

print(text.split(",")) # 구분자, 기준으로 분리
"""

### os 디렉토리 관련 내장 함수와 순환(recursion) 사용 예시
"""
import os

file="C:\Program Files (x86)\Battle.net\Battle.net.exe"
dir="C:\Program Files (x86)\Battle.net"

print(os.path.getsize(file)) #파일이면 파일의 용량을 반환한다. 디렉토리(폴더)면 용량이 0
print(os.listdir(dir)) # 디렉토리(폴더)의 하위 디렉토리와 파일을 리스트 형식으로 반환
#os.scandir(dir) 모든 하위 디렉토리와 파일을 이터레이터 형식으로 스캔하는 함수scandir

def get_dir_size(path='.'):
    total = 0
    with os.scandir(path) as it:
        for entry in it:
            if entry.is_file(): # 파일이면 True 반환
                total += entry.stat().st_size #파일의 용량을 total에 대입. total += os.stat(entry).st_size로도 가능
            elif entry.is_dir():
                total += get_dir_size(entry.path) #디렉토리면 하위 디렉토리와 파일을 순환시킨다.
    return total


print(get_dir_size(dir))

# def get_dir_size2(path='.'):
#     total = 0
#     for x in os.scandir(path) :
#         print(x.name,x.path, os.stat(x).st_size) name과 path가 내장되어있다
"""

### main()함수를 사용하는 프로그램 디자인 예시
"""
def readList():
   nlist=[]
   return nlist

def processList(nlist):
   None

def printList(nlist):
   None

def main():
   nlist = readList()
   processList(nlist)
   printList(nlist)
"""
### 리스트 추가 내용

"""
1.리스트에는 다양한 형식의 데이터를 저장할 수 있다
myList=['apple',[8,4,6]]

2.list 클래스의 생성자를 이용하는 방법
list1 = list() -> list1 = []
list2 = list("Hello") -> list2 = ['H','e','l','l',o]
list3 = list(range(0,5)) -> list3 = [0,1,2,3,4]
"""

### 약간 일관성이 없어보이는 리스트 좌표 예시(인덱스는 0부터인데?)
"""
list1=[1,2,3,4,5]
list1[:3] -> [1,2,3]
list1[2:] -> [3,4,5]
list1[:] -> [1,2,3,4,5]

list2=[0,1,4,9,16,25,36,48]
list2[7] = 49 -> [0,1,4,9,16,25,36,49] #이건 인덱스처럼 0부터..

list3=['a','b','c','d','e','f','g']
list3[2:5] = ['C','D','E'] -> ['a','b','C','D','E','f','g'] #[A:B] A는 0부터,B는 B 그대로;
list3[2:5] = [] -> ['a','b','f','g']

print(list3.index(b)) -> 1
"""


### 리스트.sort()와 sorted(리스트)의 차이
### sort()는 원본을 변경하고 sorted()는 정렬만해서 출력한다
"""
역순으로 정렬할때는 reverse=True를 추가한다
sorted([4,2,3,1,5], reverse=True)
-> [5,4,3,2,1]

람다식 사용 예시 (가장 끝 글자를 알파벳순으로 정렬)

f_list = ['orange','mango','papaya']

print(sorted(f_list, key=lambda x:x[-1]) ) 
"""

### 얕은 복사(shallow copy)와 ,깊은 복사(depp copy)

"""
얕은 복사
scores=[100,80]
values=list(score)

깊은 복사
import copy

original_list = [1,2,[3,4,5]]
deep_copied_list = copy.deepcopy(original_list)
"""

### call by value, call by reference 개념
"""
값으로 호출하기:함수로 변수를 전달할때 변수의 값이 복사되는 방식, 많이 사용되는 방식이다
참조로 호출하기:함수로 변수를 전달할때 변수의 참조가 전달되는 방식
"""

### x**2 -> 2제곱

### 2차원 리스트 생성 예시
"""
rows=3
cols=5

s=[]
for row in range(rows) :
   s += [[0]*cols]

print("s = ", s)  #-> s= [[0,0,0,0,0],[0,0,0,0,0],[0,0,0,0,0]]

#####리스트 함축 사용 시 M = [x for x in range(10) if x % 2 ==0]

s = [([0])*cols] for row in range(rows) ]

"""
### 튜플은 ()로 선언하며 한번 만들어지면 변경이 불가능하다.
### 딕셔너리는 key와 value를 쌍으로 데이터를 저장하는 자료구조이다.
""" 딕셔너리 = { 키1:값1, 키2,값2, ...}
딕셔너리를 읽기 전용으로 변환

from types import MappingProxyType

d = {'key1':'value1'}

d_readonly = MappingProxyType(d)

"""

### setdefault() 튜플-> 딕셔너리 변환 예시 
"""
source = (('k1','val1'),
          ('k1','val2'),
          ('k2','val3'),
          ('k2','val4'),
          ('k2','val5'))

new_dic1 ={}
new_dic2 ={}

for k, v in source:
    if k in new_dic1:
        new_dic1[k].append(v)
    else:
        new_dic1[k] = [v]

print(new_dic1)

for k, v in source: 
    new_dic2.setdefault(k,[]).append(v)

    
print(new_dic2)
"""

### 부모 클래스 생성자 호출하기
"""
class 자식클래스(부모클래스명) :
   def __init__(self, 속성1, 속성2, 속성3) :
      super().__init__(속성1,속성2) #부모클래스의 생성자를 명시적으로 호출한다.
"""

#######################인프런 파이썬 level.2 #####


### 클래스 생성자 사용 예시

'''
class Car():
   """           객체.__doc__를 호출하면 출력할 내용
   Car class
   Author : kim
   Date:2019.11.08
   """
   price_per_raise = 1.0 # 클래스 변수

   def __init__(self, company, details):
      self._company = company  # 인스턴스 변수
      self._details = details

   def __str__(self):
      return 'str : {} - {}'.format(self._company,self._details)

   def __repr__(self):
      return 'repr : {} - {}'.format(self._company,self._details)
'''
### str은 비공식적인, 프린트문으로 어떤 사용자 입장의 문자열 출력을 원할때,
### repr은 자료형의 타입에 따른 객체를 그대로 표시해줄때 사용
### 둘 다 사용해도 좋음


### dir(객체)  객체의 모든 속성(인스턴스 변수)을 출력

### 객체.__dict__  key:value  모든 값을 출력

"""
객체의 특정 내부 속성을 지정하여 가져오는 경우:
 self.__속성__.get(내부 속성명)

인스턴스 네임스페이스(인스턴스 변수 선언)에 없으면 
상위(클래스 변수, 부모 클래스 변수)에서 검색 
"""
### class, static, instance method
"""

instance method : class에서 def 로 선언한 method



인스턴스 변수를 접근하는 것은 위험하므로  
get_속성 형태로 method를 만든다

class method : 클래스 내부에 @classmethod로 선언 
self 대신 cls를 인자로 받는다

예시 
@classmethod
def 메소드명(cls,입력받을 인자명):
     cls.클래스변수명

static method
정형화된 인자를 받지않고 자유롭게 인자를 받는다

예시
@staticmethod
def 메소드명(입력받을 인자명):
     입력받을 인자명._인스턴스변수명(클래스변수가 와도 될듯)

매직 매소드

n + 100 -> n.__add__(100)

bool(n) ->n.__bool__()

n*100 -> n.__mul__(100)

"""

# ord(문자형 변수) 변수의 유니코드 정수를 반환
# chr(정수형 변수) 변수의 유니코드 문자를 반환

#chapter 4-1

"""
시퀀스형
컨테이너( 서로 다른 자료형[list,tuple, collection])
플랫(Flat: 한 개의 자료형[str,bytes,bytearray, array.array, memoryview])
가변(list,bytearray,array.array, memoryview, deque)
불변(tuple, str, bytes)
"""

###제네레이터 생성 예시
"""
# Generator 생성 
import array

# Generator : 한 번에 한 개의 항목을 생성(메모리 유지X)
tuple_g = (ord(s) for s in chars)
# Array
array_g = array.array('I',  (ord(s) for s in chars))


print(array_g.tolist()) #리스트로 변경
"""

### 리스트 주의할 점 (결론 for 문으로 만들자)
"""
marks1 = [['~'] * 5 for n in range(5)]
marks2 = [['~'] * 5] * 5

print(marks1)
print(marks2)

print()

# 수정
marks1[0][1] = 'X'
marks2[0][1] = 'X'

print(marks1)
print(marks2)

# 증명
print([id(i) for i in marks1])
print([id(i) for i in marks2])

#증명 결과 첨부
>>> print([id(i) for i in marks1])
[2074082708544, 2074082708608, 2074082708800, 2074082708672, 2074082708480]
>>> print([id(i) for i in marks2])
[2074082709312, 2074082709312, 2074082709312, 2074082709312, 2074082709312]
"""


### divmod(x, y) = (x // y, x % y)  첫번 째 숫자(피제수)를 두번 째 숫자(제수)로 나눈 몫과 나머지를 튜플(tuple) 형태로 반환
"""
divmod(8, 2)
->(4, 0)
"""

### 클로저 예시: 메모리를 공유하지 않고 메시지 전달로 처리
"""
def Closer():
    #free variable 클로저 영역(프리 변수) Avger_cls.__code__.co_freevars(프리 변수명 확인)/ Avger_cls.__closure__[0].cell_contents(프리 변수 내용 확인)
    series = []

    def Avger(v): 
        nonlocal series   #프리 변수임을 선언
        series.append(v)
        return  series
    return Avger

Avger_cls = Closer()

print(Avger_cls("a"))
print(Avger_cls("b"))      #->['a'] ['a', 'b'] 'a'를 기억 
print(Avger_cls.__code__.co_freevars)
print(Avger_cls.__closure__[0].cell_contents)
"""

### 이터레이터 이해를 위한 풀어쓴 코드
"""
t = 'abcdef'

# while
w =  iter(t)

while True:
    try:
        print(next(w))
    except StopIteration:
        break
    
"""
### 제너레이터/코루틴 이해를 위한 yield, yield from절 사용 예시
"""
# next 사용
class WordSplitter:
    def __init__(self, text):
        self._idx = 0
        self._text = text.split(' ')
    
    def __next__(self):
        # print('Called __next__')
        try:
            word = self._text[self._idx]
        except IndexError:
            raise StopIteration('Stopped Iteration.')
        self._idx += 1
        return word

    def __repr__(self):
        return 'WordSplit(%s)' % (self._text)

wi = WordSplitter('Do today what you could do tomorrow')

print(wi)
print(next(wi))
print(next(wi))

# Generator 패턴
# 1.지능형 리스트, 딕셔너리, 집합 -> 데이터 양 증가 후 메모리 사용량 증가 -> 제네레이터 사용 권장
# 2.단위 실행 가능한 코루틴(Coroutine) 구현과 연동
# 3.작은 메모리 조각 사용

class WordSplitGenerator:
    def __init__(self, text):
        self._text = text.split(' ')
    
    def __iter__(self):
        # print('Called __iter__')
        for word in self._text:
           yield word # 제네레이터
        return

   #  yield from절을 사용하는 경우   
   #  def __iter__(self):
   #      # print('Called __iter__')       
   #      yield from self._text
   #      return

    def __repr__(self):
        return 'WordSplit(%s)' % (self._text)


wg = WordSplitGenerator('Do today what you could do tomorrow')

wt = iter(wg)

print(wt)
print(next(wt))
print(next(wt))
"""

# Generator (중요 함수)
# filterfalse, accumulate, chain, product, product, groupby
"""
import itertools

# 조건
gen2 = itertools.takewhile(lambda n : n < 1000, itertools.count(1, 2.5))

for v in gen2:
    print(v)

# 필터 반대
gen3 = itertools.filterfalse(lambda n : n < 3, [1,2,3,4,5])

for v in gen3:
    print(v)


# 누적 합계
gen4 = itertools.accumulate([x for x in range(1, 101)])

for v in gen4:
    print(v)

# 연결1
gen5 = itertools.chain('ABCDE', range(1,11,2))

print(list(gen5))

# 연결2

gen6 = itertools.chain(enumerate('ABCDE'))

print(list(gen6))

# 개별
gen7 = itertools.product('ABCDE')

print(list(gen7))

# 연산(경우의 수)
gen8 = itertools.product('ABCDE', repeat=2)

print(list(gen8))

# 그룹화
gen9 = itertools.groupby('AAABBCCCCDDEEE')

# print(list(gen9))

for chr, group in gen9:
    print(chr, ' : ', list(group))
"""

```