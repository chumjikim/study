#Models


- 각 모델은 django.db.models.Model의 서브클래스이다.
- 각 모델의 속성은 데이터베이스의 field이다.
- Django는 데이터베이스 접근 API를 자동으로 생성한다.(ORM | 이걸 다루는 방법이 쿼리임)  [Making queries](https://docs.djangoproject.com/en/1.10/topics/db/queries/)

##Using models
모델을 정의하면, Dajgno에게 사용하려고 하는 모델을 알려줘야한다. `settings.py`파일의`INSTALLED_APPS`설정에 `models.py`파일이 포함된 모듈의 이름을 추가해야한다. 

```python
# settings.py

INSTALLED_APPS = [
	#...
	'myapp',
	#...
]
```
INSTALLED_APPS에 새로운 app을 추가할 때, `manage.py migrate`를 실행하고, `mange.py makemigrations`(옵셔널)를 사용해 변경사항들을 저장한다.
 
**migration**  
migration은 django가 모델의 변경사항을 저장하는 방법이다. 디스크상의 파일로 존재하며 polls/migrations/0001_initial.py파일로 저장된 새 모델에 대한 migration을 확인할 수 있다.

**migrate**  
migration들을 실행시키고, 자동으로 데이터베이스 스키마를 관리해주는 명령어이다.

##Fields
모델에서 가장 중요하고 유일하게 요구되는 부분은 데이터베이스 fields의 리스트를 정의하는 것이다. Field는 클래스의 속성으로 정의된다. field name을 선택할 때, clean, save, delete와 같은 model API와 충돌을 일으키는 이름은 피해야한다.

```python 
# example

from django.db import models

class Musician(models.Model):
	first_name = models.CharField(max_length=50)
	last_name = models.CharField(max_length=50)
	instrument = models.CharField(max_length=100)
	
class Album(models.Model):
	artist = models.ForeignKey(Musician)
	name = models.CharField(max_length=100)
	release_date = models.DateField()
	num_start = models.IntegerField()
```
###Field types
모델의 각 field는 `Feild` 클래스의 인스턴스이다. Django는 Feild 클래스의 유형에 따라 몇가지를 결정한다.

- column type은 어떤 종류의 데이터가 저장되는지 알려준다. (INTEGER, VARCHAR, TEXT)
- form field를 렌더링 할 때, 기본 HTML 위젯을 사용한다. (`<input type="text">, <select>`)
- 생성되는 form과 django의 admin에서 최소한의 유효성 검증이 자동으로 이루어진다.

>django에는 많은 내장 field type들이 존재한다.   
>[model field reference](https://docs.djangoproject.com/en/1.10/ref/models/fields/#model-field-types)

###Field options

각 field는 특정한 arguments 집합을 갖는다. 
예를 들어, CharField(와 이것의 서브클래스들)는 데이터 저장에 사용되는 VARCHAR 데이터베이스 필드의 크기를 지정하기 위해 max_length argument를 지정한다. 

모든 field 타입에 사용할 수 있는 공통적인 argument들도 존재한다. (optional함)
>[model field reference](https://docs.djangoproject.com/en/1.10/ref/models/fields/#model-field-types)  

**null**  
True로 설정하면 django는 NULL로 비어있는 값을 저장하게된다. 기본값은 False이다.

**blank**  
True로 설정하면 해당 field는 공백을 허용하게된다. 기본값은 False이다. (비어있는 string임)

>null과는 다르다.   
>null은 데이터베이스에 관련되어있고 blank는 유효성 검사에 관련되어있다. 만약 field가 blank=True라면, form 유효성 검사는 비어있는 값을 허용할 것이다. field가 blank=False라면 해당 field는 값이 요구된다.

**choices**  
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
**default**  
field의 기본 값이다. 어떠한 값 또는 호출 가능한 객체이다. 만약 호출 가능한 객체라면 새로운 객체가 생성될 때 마다 호출된다.

**help_text**  
form 위젯과 함께 표시되는 'help'텍스트이다. field가 form에서 사용되지 않더라도 유용하다.

**primary_key**  
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

**unique**  
unique가 True라면, 해당 field는 그 테이블에서 고유한 값을 가져야한다.



###Automatic primary key fields
기본적으로 django는 각 모델에 아래와 같은 field를 갖고있다. 

```python
id = models.AutoField(primary_key=True)
```
이것은 자동으로 증가하는 `primary key`이다.
만약 primary key를 커스터마이징 하고싶다면, field 중 하나에 `primary_key = True`를 지정한다. Django는 `primary_key = True`를 명시적으로 지정했을 경우 auto-increment field를 자동으로 추가하지 않는다.

각 모델은 `primary_key = True`를 갖기 위해서 정확히 하나의 field가 필요하다.

###Verbose field names
ForeignKey, ManyToManyField, OneToOneField를 제외한 각 field 타입은 첫 번째 argument에 verbose name(해당필드에 대한 설명)을 가질 수 있다. 만약 이름이 주어지지 않으면, django는 해당 field의 속성 이름을 이용해(underscores는 공백으로 변환한다) 자동적으로 생성한다. 

아래 예제에서 verbose name은 "persons's first name"이다.

```python
first_name = models.CharField("persons's first name", max_length=30)
```
아래의 예제에서 verbose name은 "first name"이 된다.

```python
first_name = models.CharField(max_length=30)
```

ForeignKey, ManyToManyField, OneToOneField는 첫 번째 argument로 모델 클래스를 갖는다. verbose name은 다른 위치에 정의해준다.

```python
poll = models.ForeignKey(
	Poll,
	ondelete=models.CASCADE,
	verbose_name="the related poll",
)
site = models.ManyToManyField(Site, verbose_name="list of sites")
place = models.OneToOneField(
	Place,
	on_delete=models.CASCADE,
	verbose_name="related place",
)
```
verbose_name의 첫 번째 문자를 대문자로 사용하지는 않는것이 컨벤션이다. Django가 필요할 때 첫 번째 문자를 자동으로 대문자로 만든다.

###Relationships
관계형데이터베이스의 힘은 테이블을 서로 연결하는것이다. Django는 다-대-일, 다-대-다, 일-대-일 세 가지 데이터베이스 관계의 가장 일반적인 유형을 정의하는 방법을 제공한다.

####Many-to-one relationships (다 - 대- 일)

**`django.db.models.ForeignKey`**를 이용해 다-대-일 관계를 정의한다. 다른 field 유형과 마찬가지로 모델의 클래스 속성으로 포함해 사용한다.

ForeignKey는 위치 argument가 필요하다. 모델에 연결할 클래스가 argument가 된다. 

예를 들어, 하나의 `Car`모델이 하나의 `Manufacturer`을 갖는다고 했을 때, `Manufacturer`는 수많은 car를 만들어내지만 각각의 car는 하나의 Manufacurer를 갖게된다. 

```python
from django.db import models

class Manufacturer(models.Model):
	#...
	pass
	
class Car(models.Model):
	manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
	#...
```

재귀적 관계(자기 자신과 다-대-일 관계가 있는 객체)와 아직 정의되지 않은 모델과의 관계도 만들 수 있다.

**ForeignKey의 재귀적 관계**  
재귀적 관계(Many-to-one 관계를 자기 자신과 맺는 객체)를 생성하려면 **`models.ForeignKey('self', on_delete=modles.CASCADE)`**를 사용한다.

아직 정의되지 않은 모델에서 관계를 정의하려면, 모델 오브젝트 자체가 아닌 모델 이름을 사용할 수 있다.

```python
from django.db import models

class Car(models.Model):
	manufacturer = models.ForeignKey(
		'Manufacturer',
		on_delete=models.CASCADE,
	)
	#...
class Manufacturer(models.Model):
	#...
	pass
```
>질문: 다른 클래스를 참조하려면 지정하는 순서의 영향을 받을텐데 예제는 Car를 먼저 작성했다. 괜찮은가?

**on_delete=models.CASCADE**   
계단식 삭제. Django는 SQL의 ON DELETE CASCADE 제약 조건을 에뮬레이트하고, ForeignKey가 포함된 객체도 삭제한다.  
 
**on_delete=models.SETNULL**   
일-대-다 의 관계가 정의 되었을 때, `일` 의 정보가 삭제되면, `다`는 관계된 정보값을 찾을 수 없게된다. `on_delete=models.SETNULL` 속성을 지정하면, `다`는 `NULL`값에 관계된 것으로 처리되어 에러가 발생하지 않는다.

**related_name**  
자기 자신을 ForeignKey로 참조하는 경우가 2번 이상일 때, 역참조 이름(_set)이 겹치게 된다. 따라서 각 객체에 `related_name`을 따로 설정해서 역참조에 사용한다.
  
자세한 내용은 [모델 필드](https://docs.djangoproject.com/en/1.10/ref/models/fields/#ref-foreignkey) 참조 

>ForeignKey field의 이름(예제에서는 manufacturer)은 소문자인 모델의 이름을 사용할 것을 권장한다. 필수는 아님.

####Many-to-many relationships
**`ManyToManyField`**를 사용하여 다-대-다 관계를 정의할 수 있다. 다른 field 유형과 마찬가지로 모델 클래스 속성으로 포함하여 사용한다.

ManyToManyFeild는 위치 argument가 필요하다. 모델에 연결할 클래스가 argument가 된다. 

예를 들면, Pizza가 다양한 Topping객체가 있을 경우, Topping이 여러 pizza에 있을 수 있고, 각 Pizza에 여러 Topping이 있는 경우이다.

```python
from django.db import models

class Topping(models.Model):
	#...
	pass
	
class Pizza(models.Model):
	#...
	toppings = models.ManyToManyField(Topping)
```
ForeignKey와 마찬가지로 재귀 관계와 아직 정의되지 않은 모델과의 관계를 만들 수도 있다.

ManyToManyField의 이름은 관련 모델 객체 집합을 설명하는 복수형으로 제안되지만 필수는 아니다.

어떤 모델에 ManyToMany가 있는지는 중요하지 않지만, 두 모델 중 하나에만 적용해야한다.

일반적으로, ManyToMany 인스턴스는 form에서 편집할 객체에 있어야한다. 위의 예제에서 topping이 여러개의 pizza를 갖는 것 보다, pizza가 여러개의 topping을 갖는 것이 자연스럽기 때문에 Pizza에 존재한다. 위의 방식대로라면, Pizza form을 사용하면 사용자가 topping을 선택할 수 있다.

[ManyToMany 예시](https://docs.djangoproject.com/en/1.10/topics/db/examples/many_to_many/)

ManyToManyField는 추가적인 arguement들을 가질 수 있다. [the model field reference](https://docs.djangoproject.com/en/1.10/ref/models/fields/#manytomany-arguments)에서 설명함.

**ManyToManyField.symmetrical**

ManyToManyField가 재귀적 관계일 때만 사용된다. 

```python
from django.db import models

class Person(models.Model):
	friends = models.ManyToManyField('self')
```  
이 모델은 ManyToManyField를 자기 자신에게 갖는다. 이 경우, Person 클래스에는 `person_set`속성이 주어지지 않는다. 즉 역 참조에 다른 값이 있지 않고 ManyToManyField는 대칭이라고 가정하는 것이다. A가 B의 친구라고 정의하면, 자동적으로 B가 A의 친구라고 여기는 것이다.

만약 두 관계를 대칭적으로 설정하고 싶지 않으면 `symmetrical=False`라고 설정해 비대칭 관계를 만들어 줄 수 있다. 

예를 들어 followship을 정의할 때, ManyToMany관계를 사용하는데 followship은 대칭관계가 아니기 때문에(A가 B를 팔로우한다고해서 B가 A를 팔로우하는 것은 아님) `symmetrical=False`를 적용해 비대칭적인 관계를 설정한다. 

```python
from django.db import models

class User(models.Model):
	following = models.ManyToManyField(
	        'self',
	        related_name='follower_set',
	        symmetrical=False,
	    )
```
   
>A.following.add(B): A가 B를 follow한다.   
>(정의된 related_name은 `follower_set`임)  
>B.follwer\_set.all(): B를 follow하는 사람들을 역참조하여 불러온다.

####Extra fields on many-to-many relationships
피자와 토핑을 믹스하고 매치하는 것 처럼 단순한 다-대-다 관계만 처리할 때는, 표준 ManyToManyField만 있으면 된다. 그러나 때로는 두 모델 간의 관계에 데이터를 연결할 수 도 있다. 

이런 경우 django는 many-to-many관계를 관리에 사용하는 모델을 지정할 수 있다. 중간자 모델을 생성하여 추가적인 필드를 생성하면 된다. ManyToManyField에 through 매개변수를 사용하여 중간자 모델을 연결한다. `through='중간자 모델 이름' ` 

```python
class Idol(models.Model):
	name = models.CharField(max_length=100)
	
	def __str__(self):
		return self.name
		

class Group(models.Model):
	name = models.CharField(max_length=100)
	members = moderls.ManyToManyField(
	Idol,
	through='MemberShip',
	through_fields=('group', 'idol'),
	)
	
	def __str__(self):
		return self.name


class MemberShip(modles.Model):	
	idol = modles.ForeignKey(idol)
	group = models.ForeignKey(Group)
	date_joined = models.DateTimeField()
	recommender = models.ForeginKey(
		Idol,
		null=True,
		blank=True,
		related_name=''
	)	

```
중간자 모델(intermediary model)을 설정할 때, 다-대-다 관계에 관련된 모델에 대해서 ForeignKey를 명시적으로 지정한다.

**중간자모델 제약사항**  

- 중간자 모델은 소스 모델(위의 예제에서는 Group)을 가리키는 단 하나의 ForeignKey를 가져야한다. 또는 Django가 사용해야하는 ForeignKey를 **`ManyToManyField.through_fields`**를 사용하여 명시적으로 지정한다. 만약 하나를 초과하는 ForeignKey를 갖고있음에도 `through_fields`를 지정해주지 않으면 유효성검증(validation)에서 에러가 발생한다. 
- 중간자 모델을 통해 ManyToMany관계가 재귀적인 경우, 동일한 모델에 대한 두 개의 ForeignKey를 사용할 수 있다. 하지만 다-대-다 관계의 두 가지 다른 측면으로 받아들여진다. 두 개 이상의 ForeignKey가 있는 경우에는 **`through_fields`**를 사용하지 않으면 validation error가 일어날 것이다. 
- 중간자 모델을 통해 ManyToMany 관계를 재귀적으로 정의할 경우에는  **`symmetrical=False`**를 지정해줘야한다. 


일반적인 many-to-many field와는 다르게  add(), create(), set()과 같은 것을 사용할 수 없다. remove()도 안됨.

clear()메소드를 사용하면 인스턴스의 many-to-many관계를 삭제할 수 있다. 
####One to One relationships
일대일 관계를 지정하려면 `OneToOneField`를 사용하면 된다. 모델의 클래스 속성으로 추가하여 

예를들면 한 사용자에 대한 추가 프로필 같은 것.



###Models across files
모델은 다른 앱의 모델과 연결할 수 있다. 이를 위해 모델이 정의 된 파일의 맨 위에 연결할 모델을 import해준다. 그리고 필요한 곳에서 다른 모델 클래스를 참조하면 된다.

```python
from django.db import models
from geography.models import ZipCode

class Restaurant(models.Model):
	#...
	zip_code = models.ForeignKey(
		ZipCode,
		on_delete=models.SET_NULL,
		blank=True,
		null=True,
	)
```
###Field name restrictions
django는 Model field name에 두 가지 제약을 갖고있다. 

- field name에 Python 예약어를 사용할 수 없다. 아래 예제처럼 Python 예약어를 사용하면 Python syntax error가 난다. 

```python 
class Example(models.Model):
	pass = models.IntegerField()
	# pass 는 예약어이다.
```

- field name에 underscore`_`를 하나를 초과해서 사용할 수 없다. Django가 query를 조회할 때 작동하는 구문이기 때문이다. 

```python
class Example(models.Model):
	foo__bar = models.IntegerField()
	# 'foo__bar'에 underscore가 두 개 사용되었다. 
```
이러한 제약들은 field name은 데이터베이스의 column name과 일치해야할 필요가 없기 때문에 예외가 있을 수 있다.

Django는 기본 SQL query에서 모든 데이터베이스 테이블의 이름과 컬럼이름을 escape하기 때문에 join, where, select와 같은 SQL 예약어는 model field name으로 허용한다. Django는 특정한 데이터베이스 엔진의 인용구문을 사용한다.

>[db column option](https://docs.djangoproject.com/en/1.10/ref/models/fields/#django.db.models.Field.db_column)


###Custom field types
존재하는 모델 field 중 하나가 목적에 맞게 사용되지 않거나, 일반적이지 않은 유형의 데이터베이스 컬럼타입을 사용하려면 자체적인 field class를 생성해야한다.  
[Writing custom model fields](https://docs.djangoproject.com/en/1.10/howto/custom-model-fields/)

##Meta options
내부의 `Meta`클래스를 사용해서 모델 메타데이터를 제공한다. 

```python
from django.db import models

class Ox(models.Model):
	horn_length = models.IntegerField()
	
	class Meta:
		ordering=["horn_length"]
		verbose_name_plural = "oxen"
```

모델의 메타데이터는 ordering 옵션, 데이터베이스 테이블 이름, 사용자가 읽을 수있는 단수 및 복수 이름(verbose\_name 및 verbose\_name\_plural)과 같이 **"필드가 아닌 모든"** 정보가 들어간다. 클래스 메타를 모델에 추가하는 것은 선택 사항이다.(반드시 필요한 것은 아님.)

[메타 옵션의 전체 목록: model option reference](https://docs.djangoproject.com/en/1.10/ref/models/options/)

##Model attributes
**objects**  
모델의 가장 중요한 속성 중 하나는 **Manager**이다. 이것은 Django 모델에 데이터베이스 쿼리 작업이 제공되고, 데이터베이스 인스턴스를 검색하는데 사용되는 인터페이스이다. (매니저를 통해 쿼리를 만들 수 있음) 
커스텀 Manager가 지정되지 않은 경우, 기본 이름은 **`objects`**이다. Manager는 모델 인스턴스가 아닌 모델 클래스만을 통해서 접근할 수 있다. 

##Model methods
모델에 커스텀 method를 정의하여 커스텀 'row-level' 기능을 객체에 추가한다. Manager method는 '테이블 전체'작업을 수행하지만, Model method는 특정 model 인스턴스(각 row값)에서 작동한다. self를 인자로 받음

```python
from django.db import models

class Person(models.Model):
	first_name = models.CharField(max_length=50)
	last_name = models.CharField(max_length=50)
	birth_date = models.DateField()
	
	def baby_boomer_status(self):
		"Return the person's baby-boomer status."
		import datetime
		if self.birth_date < datetime.date(1945, 8, 1):
			return "Pre-boomer"
		elif self.birth_date < datetime.date(1965, 1, 1):
			return "Baby boomer"
		else:
			return "Post-boomer"
			
	get _get_full_name(self):
		"Returns the person's full name."
		return '%s %s' % (self.first_name, self.last_name)
	full_name = property(_get_full_name)
```
[property](https://docs.djangoproject.com/en/1.10/glossary/#term-property)

**__str__()**  
모든 객체의 유니코드 표현을 리턴하는 Python의 "masic method"이다. 모델 인스턴스를 강제로 문자로 표현해야할 경우 Python과 Django가 사용한다. 대부분 console이나 admin에서 개체를 표시할 때 사용한다.

**get\_absolute\_url()**  
Django에게 객체의 URL을 어떻게 계산해야하는지 알려준다. Django는 admin 인터페이스에서 이것을 사용하고, 언제든지 객체의 URL을 찾는다. 

객체가 고유하게 식별하는 URL을 갖는 경우 이 메소드를 사용해야한다. 

###Overriding predefined model methods
커스터마이징 할 데이터베이스 동작을 캡슐화하는 또 다른 모델 메서드 집합이 있다. 특히 save()와 delete()의 작업을 변경하려는 경우가 많다. 

이러한 메서드들은 다른 메서드들로 재정의 할 수 있다. 

내장 method를 재정의하는 기본적인 사례는, 객체를 저장할 때마다 어떠한 동작을 일으키고 싶을 때이다. 

```python
from django.db import models

class Blog(models.Model):
	name = models.CharField(max_length=100)
	tagline = models.TextField()
	
	def save(self, *args, **kwargs):
		do_something()
		super(Blog, self).save(*args, **kwargs) # 진짜 save()메소드를 호출한다.
		
		do_somthing_else()
```
저장을 막을 수도 있다. 

```python
from django.db import models

class Blog(models.Model):
	name = models.CharField(max_length=100)
	tagline = models.TextField()
	
	def save(self, *args, **kwargs):
		if self.name == "Yoko Ono's blog":
		return 
	else:
		super(Blog, self).save(*args, **kwargs)
```
위의 예제처럼 superclass method `super(Blog, self).save(*args, **kwargs)`를 호출해서 객체가 데이터베이스에 저장되도록 하는 것이 중요하다. 
superclass method를 호출하는 것을 잊으면, 기본 동작은 발생하지 않고 데이터베이스에서도 아무것도 하지 않을 것이다. 

>python3 에서는`super().save(*args, **kwargs)`

model method에 전달할 argument를 전달하는 것 또한 중요하다.(\*args, \*\*kwargs)  

Django는 내장 model method 기능을 확장하여 새로운 argument를 추가한다. method 정의에 \*args, \*\*kwargs를 사용하면 코드가 추가될 때 코드가 자동으로 해당 argument들을 지원한다.

###Executing custom SQL
또다른 공통 패턴은 모델 메서드와 모듈 레벨 메서드에서 커스텀 SQL문을 작성하는 것이다.  
[using raw SQL](https://docs.djangoproject.com/en/1.10/topics/db/sql/)

##Model inheritance
Django의 모델 상속은 Python에서 일반적인 클래스 상속과 대부분 유사하게 작동한다. 하지만 아래의 기본사항은 준수해야한다. 기본 클래스는 `django.db.models.Model`의 서브클래스로 사용해야한다. 

**결정해야 할 사항**  
부모 모델이 자신만의 데이터베이스테이블이 될 지, 부모 모델이 공통적인 정보를 갖기만하고 자식모델을 통해서 보여줄지 결정해야한다.

Django에서는 세 가지 스타일의 상속이 가능하다.

1. 추상 기본 클래스. 부모 클래스가 정보를 가지고 있는 것 만 원한다면,  각 하위 모델에 대해 입력하지 않아도 된다. 이 클래스는 단독으로 사용할 수 없다. 
2. 이미 존재하는 모델을 서브클래스로 만들고 각 모델들이 자체 데이터베이스 테이블을 갖기 원한다면, 다중 테이블 상속이 필요하다.
3. 모델 필드를 수정하지 않고 모델의 Python 레벨의 동작만 수정하길 원하면, Proxy모델을 사용할 수 있다.

###Abstract base classes
몇가지 공통적인 정보를 여러 다른 모델에  넣으려면 추상 기본 클래스가 유용하다. 기본 클래스의 메타 클래스로 **`abstract=True`**를 입력하면 된다. 이 모델은 어떤 데이터베이스테이블도 생성하지 않을 것이다. 대신, 다른 모델들의 기본 클래스로 사용된다. 추상 기본클래스의 필드는 자식 클래스의 필드에 추가된다. 추상 기본클래스에 자식 클래스의 이름과 같은 필드가 생성되면 에러가 나고 django는 exception을 발생시킬 것이다.

추상클래스의 속성을 상속받은 클래스들만 데이터베이스가 만들어지고 추상 클래스는 데이터베이스를 생성하지않는다. 

```python
from django.db import models

class CommonInfo(models.Model):
	name = models.CharField(max_length=1000)
	age = modles.PositiveIntegerField()
	
	class Meta:
		abstract = True

class Student(CommonInfo):
	home_group = models.CharField(max_length=5)
```
Student 모델은 name, age, home_group이라는 세가지 필드를 갖게된다. CommonInfo모델은 추상 기본 클래스이기 때문에 일반적인 Django모델로 사용할 수 없다.(CommonInfo.objects. ~~ 이렇게 쓸수 없다는 것) 추상 기본 클래스는 데이터베이스 테이블이나 manager를 생성하지 않고, 인스턴스로 만들거나 직접 저장할 수도 없다.

abc로 줄여서 씀.

####Meta inheritance
추상 기본 클래스가 생성되면, django는 기본 클래스에서 선언한 내부 Meta 클래스를 속성으로 사용할 수 있게 만든다. 만약 자식 클래스가 자신만의 Meta 클래스를 선언하지 않았으면, 부모의 Meta 클래스를 상속받는다. 자식 클래스가 부모 클래스의 Meta 클래스를 확장하고 싶으면 아래의 예제와 같이 하면된다.

```python
from django.db import models

class CommonInfo(models.Model):
	#...
	class Meta:
		abstract = True
		ordering = ['name']
		
class Student(CommonInfo):
	#...
	class Meta(CommonInfo):
		db_table = 'student_info'
```
django는 추상 기본클래스의 Meta클래스에 맞춰 조절한다. Meta 속성을 지정하기 전에는 `abstract=False`로 지정되어있다. 이 말은 추상 기본 클래스의 자식 클래스들은 자동적으로 추상클래스가 되지 않는다는 것이다. 물론 다른 추상 기본클래스에서 상속받아 추상 기본 클래스를 만들 수 있다. 매번 `abstract=True`이라고 명시적으로 써주는 것만 기억하면 된다. 

추상 기본 클래스의 Meta 클래스에 포함하는 것이 적합하지 않은 속성들도 있다. 예를 들어, Meta 클래스가 `db_table`을 포함하게되면 모든 자식클래스들이 같은 데이터베이스를 사용하게된다. 

####Be careful with related\_name and related\_query\_name
ForeignKey와 ManyToManyField에 `related_name`이나 `related_query_name`을 사용한다면, 필드에 항상 고유한 reverse name과 query name을 지정해줘야한다. 이 클래스의 필드는 매번 정확히 동일한 속성값(related_name과 related\_query\_name을 포함)으로 각 자식 클래스에 포함되기 때문에 추상 기본 클래스에서 문제가 발생할 수 있다. (?)

문제 해결 : 추상 기본 클래스에서 related\_name 또는 related\_query\_name을 사용할 때, 값의 일부에 `%(app_label)s`and `'%(class)s'`를 포함하고 있어야한다.


- `%(class)s` 필드가 사용되는 자식 클래스의 소문자 이름으로 대체한다.
- `%(app_label)s` 자식 클래스가 포함된 앱의 소문자 이름으로 대체한다. 

설치된 각 app의 이름은 고유해야하고, 각 app 내의 모델 클래스 이름도 고유해야하므로 결과 이름이 달라진다.


related name 역방향 참조  
default: \_set
related query name 역방향 필터
default: 클래스명(소문자) 
related name이 지정되면 , related query name은 related name을 사용하게된다. 이 때, query name을 기본값으로 사용하고 싶으면 related query name = classname으로 다시 지정해줌 

**common/models.py**

```python
from django.db import models

class Base(models.Model):
	m2m = modles.ManyToManyField(
		OtherModel,
		related_name="%(app_label)s_%(class)s_related",
		related_query_name="%(app_label)s_%(class)ss",
	)

	class Meta:
		abstract = True
		
class ChildA(Base):
	pass
	
class ChildB(Base):
	pass
```

**rare/models.py**

```python
from common.models import Base

class ChildB(Base):
	pass
```
common.ChildA.m2m의 reverse name은 `common_childa_related`이고, reverse query name은 `common_childas`가 된다. common.ChildB.m2m필드의 reverse name은 `common_childb_related`가 되고, reverse query name은 `common_childbs`가 된다. 결과적으로 rare.ChildB.m2m필드의 reverse name은 `rare_childb_related`가 되고, reverse query name은 `rare_childbs`가 된다.   
related name과 related query name을 구성하는 것은 '%(class)s'와 '%(app_label)s'를 어떻게 사용하느냐에 따라 달려있다. 하지만 이것을 사용하는것을 잊는다면 django는 시스템 체크를 할 때(혹은 migrate할 때) 에러를 발생시킬 것이다. 

기본 추상 클래스의 필드에 related_name 속성을 지정하지 않으면, 기본 reverse name은 자식 클래스의 이름에 '_set'이 붙은 것이 된다. 자식 클래스에서 필드를 직접 선언했으면 정상적으로 처리된다. 예를 들어 위의 코드에서, related_name의 속성이 생략됬다면, m2m필드의 reverse name은 ChildA 필드의 경우 `childa_set`이고, ChildB 필드 의 경우 `childb_set`이다.

###Multi-table inheritance
테이블 간 연산때문에 성능이 좋지 않아서 잘 안쓴다. 
abc로 대체해서 사용함. 데이터베이스의 관점에서 볼 때 abc가 성능이 더 좋다. abc의 단점은 그 자체를 사용할 수 없다는 것.

####Meta and multi-table inheritance
####Inheritance and reverse relations
####Specifying the parent link field






###Proxy models
다중 테이블 상속을 사용할 때, 모델의 각 하위 클래스에 대해 새 데이터베이스 테이블이 만들어진다. 프록시의 경우 반대임. 부모 클래스의 경우에 만든다. 

```python
from django.db import models

class Person(models.Model):
	first_name = models.CharField(max_length=30)
	last_name = models.CharField(max_length=30)
	
class MyPerson(Person):
	class Meta:
		proxy = True
		
	def do_something(self):
		#...
		pass
```
같은 데이터에 따라 다른 객체를 갖고, 그 객체에 따라 다른 동작을 부여할 수 있다. 실제 존재하는 테이블은 하나고 여러 형태로 사용할 수 있는 것임.

```python
>>> p = Person.objects.create(first_name="foobar")
>>> MyPerson.object.get(first_name="foobar")
```



####QuerySets still return the model that was requested
####Base class restrictions
####Proxy model managers
####Differences between proxy inheritance and unmanaged models

###Multiple inheritance
다중 상속에서 메타 클래스를 포함하면 첫 번 째 부모만 사용할 수 있다.

두 개 클래스를 상속받으면 autofield가 충돌이 나기 떄문에 primary_key를 따로 지정해줘야한다.