#Field options

index

- Field options
 - null
 - blank
 - choices
 - db\_column
 - db\_index
 - db\_tablespace
 - default
 - editable
 - error\_messages
 - help\_text
 - primary_key
 - unique
 - unique\_for\_date
 - unique\_for\_month
 - unique\_for\_year
 - verbose\_name
 - validators


>[Model field reference 문서](https://docs.djangoproject.com/en/1.10/ref/models/fields/)

##null
**Field.null**
True로 설정하면 django는 NULL로 비어있는 값을 저장하게된다. 기본값은 False이다.  

CharField와 TextField와 같은 문자열 베이스 필드에서 null을 사용하는 것은 피해야한다. 빈 문자열 값은 항상 NULL이 아닌 `빈 문자열`로 저장되기 때문이다.  
만약 문자열 베이스 필드가 `null=True`라면, '데이터없음'에 대해 두 개의 값(`NULL`, `빈 문자열`)이 가능하다. 대부분 '데이터없음'에 대해 가능한 값이 두개일 필요는 없다. Django의 컨벤션은 null이 아닌 `빈 문자열`을사용하는 것이다. 

문자열 기반 필드와 비 문자열 기반 필드의 경우, null 매개변수는 데이터베이스에만 영향을 주기 때문에 form에서 빈 값을 허용하려면 `blank=True`로 설정해야한다.

##blank
True로 설정하면 해당 field는 공백을 허용하게된다. 기본값은 False이다. (비어있는 string임)

>null과는 다르다.   
>null은 데이터베이스에 관련되어있고 blank는 유효성 검사에 관련되어있다. 만약 field가 blank=True라면, form 유효성 검사는 비어있는 값을 허용할 것이다. field가 blank=False라면 해당 field는 값이 요구된다.

##choices
choices field의 선택사항으로 순회가능한 2차원 튜플을 사용한다. (리스트도 가능함)
이 옵션이 주어지면 기본 form 위젯은 standard text field가 아닌 select box가 될 것이다. 

choices list는 다음과 같이 생겼다 .

```python 
YEAR_IN_SCHOOL_CHOICES = (
	('FR','Freshman'),
	('SO','Sophmore'),
	('JR','Junior'),
	('GR','Graduate'),	
)
```
각 튜플의 첫 번째 요소는 데이터베이스에 저장될 값이다. 두번째 요소는 Model Choice Field나 기본 위젯 형식으로 표현된다. 모델 인스턴스가 주어지면, 선택 필드의 표시 값은 `get_FOO_display()`메소드를 사용해 접근할 수 있다.

```python
# example

from django.db import models

class Person(models.Model):
	SHIRT_SIZES = (
		('S','Samll'),
		('M','Medium'),
		('L','Large'),
	)
	name = models.CharField(max_length=60)
shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
```

```python
>>> p = Person(name="Fred Flinstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```
##db\_column
##db\_index
##db\_tablespace
##default
field의 기본 값이다. 어떠한 값 또는 호출 가능한 객체이다. 만약 호출 가능한 객체라면 새로운 객체가 생성될 때 마다 호출된다.

##editable
##error\_messages
##help\_text
form 위젯과 함께 표시되는 'help'텍스트이다. field가 form에서 사용되지 않더라도 유용하다.
##primary\_key
True라면, 해당 field는 모델의 primarykey가 된다 .

모델에서 어떠한 field에도 `primary_key=True`라고 지정해주지 않는다면, django는 자동으로 primarykey를 가진 `IntegerField`를 생성한다. 기본 설정을 재정의하고 싶은 것이 아니라면 `primary_key=True`라고 설정 할 필요가 없다.   
>아래에  Automatic primary key fields에서 더 설명함.

primary key field는 읽기전용이다. 
기존 객체의 primary key값을 변경하고 저장하면, 이전 객체와 함께 새로운 객체가 만들어진다.

```python
from django.db import models

class Fruit(models.Model):
	name = models.CharField(max_length=100, primary_key=True)
```

```python
>>> fruit = Fruit.objects.create(name='Apple')
>>> fruit.name = 'Pear'
>>> fruit.save()
>>> Fruit.objects.values_list('name', flat=True)
['Apple', 'Pear']
```
##unique
unique가 True라면, 해당 field는 그 테이블에서 고유한 값을 가져야한다.
##unique\_for_date
##unique\_for_month
##unique\_for\_year
##verbose\_name
Field.verbose_name  
사람이 읽기 좋은 필드의 이름이다. verbose name을 지정하지 않으면 django는 필드의 속성 
##validators 