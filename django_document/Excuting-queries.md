#Making queries
[data models](https://docs.djangoproject.com/en/1.10/topics/db/models/)을 만들면 Django는 데이터베이스 추상 API를 이용해 객체를 생성, 검색, 수정, 삭제를 할 수 있다. 이 문서에는 API의 사용법을 설명하고있다. 다양한 모델 조회 옵션에 대한 것은 [data model reference](https://docs.djangoproject.com/en/1.10/ref/models/)를 참조하면 된다.

이 가이드 전체에서는 웹블로그 어플리케이션을 구성하는 모델을 아래와 같은 모델을 참고한다.

```python
from django.db import models

class Blog(models.Model):
	name = models.CharField(max_length=100)
	tagline = models.TextField90
	
	def __str__(self):
		return self.name
		
class Author(models.Model):
	name = models.CharField(max_length=200)
	email = models.EmailField()
	
	def __str__(self):
		return self.name
		
class Entry(models.Model):
	blog = models.ForeignKey(Blog)
	headline = models.CharField(max_length=255)
	body_text = models.TextField()
	pub_date = models.DateField()
	mod_date = models.DateField()
	authors = models.ManyToManyField(Author)
	n_comments = models.IntegerField()
	n_pingbacks = models.IntegerField()
	rating = models.IntegerField()

	def __str__(self):
		return self.headline
```

##Creating Objects
모델 클래스는 데이터베이스 테이블을 나타내고, 클래스의 인스턴스는 데이터베이스 테이블의 특정 레코드를 나타낸다. 

객체를 생성하는 방법은 모델 클래스에 keyword arguments를 사용해 인스턴스를 만들고, save()를 호출해 객체를 데이터베이스에 저장한다.

mysite/blog/models.py 파일이 있다고 가정하고 아래의 예제를 보자.

```python
>>>from blog.models import Blog
>>>b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>>b.save()
```
이 코드는 뒤에서 INSERT SQL문을 수행한다. Django는 save()를 호출하기 전 까지는 데이터베이스에 접근하지 않는다. 

save()메서드는 리턴하는 값이 없다. 

**create(\*\*kwargs)**  
객체를 생성하고 저장하는 것을 한번에 할 수 있는 컨벤션이다. 

```python
b = Blog.obejcts.create(name=;Beatles Blog', tagline='All the latest Beatles news.')
```
이 코드는 객체 `b`를 생성하고 `save()`를 호출해 객체를 저장한 방법과 동일하지만 한번에 할 수 있다. 

##Saving changes to objects
데이터베이스에 존재하는 객체를 수정하여 저장하려고 하면, save()를 사용한다.

데이터베이스에 이미 저장된 블로그 인스턴스 b5가 주어졌을 때, 아래의 코드는 b5의 이름을 변경하고 데이터베이스에서 해당 레코드를 업데이트한다. 

```python
>>>b5.name = 'New name'
>>>b5.save()
```
이 코드는 뒤에서 UPDATE SQL문을 수행한다. save()메서드가 호출되기 전 까지는 데이터베이스에는 접근하지 않는다. 

###Saving ForeignKey and ManyToManyField fields
ForeignKey 필드를 업데이트하는 것은 해당 필드에 올바른 타입의 객체를 지정하기만 한다면 일반 필드를 저장하는 것과 같은 방식이다. 

아래의 예제는 Blog와 Entry의 인스턴스가 적절히 설정되어있다는 가정하에 Entry 인스턴스의 블로그 속성을 수정한다.

```python
>>>from blog.models import Entry
>>>entry = Entry.objects.get(pk=1)
>>>cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>>entry.blog = cheese_blog
>>>entry.save()
```
ManyToManyField를 업데이트하는 것은 약간 다르다. 필드에 **add()**메서드를 사용해 관련된 레코드를 추가한다. 아래의 예제는 Author 인스턴스 **joe**를 entry객체에 추가한다.

```python
>>>from blog.models import Author
>>>joe = Author.objects.create(name="Joe")
>>>entry.authors.add(joe)
```
ManyToManyfield에 여러개의 레코드를 추가하려면 아래와 같이 add()메서드를 호출할 때, argument를 여러개 지정하면 된다.

```python
>>>john = Author.objects.create(name='John')
>>>paul = Author.objects.create(name='paul')
>>>george = Author.objects.create(name='george')
>>>ringo = Author.objects.create(name='Ringo')
>>>entry.authors.add(john, paul, george, ringo)
```
##Retrieving objects
데이터베이스에서 객체를 찾으려면, 모델 클래스의 Manager를 통해 QuerySet을 생성해야한다.

QuerySet은 데이터베이스의 객체 집합이다. QuerySet은 0개, 하나 혹은 여러개의 filter를 가질 수 있다. filter는 주어진 매개 변수를 기반으로 쿼리 결과의 범위를 좁힌다. SQL용어에서 QuerySet은 SELECT문과 같고 filter는 WHERE 또는 LIMIT과 같다. 

모델의 Manger를 사용하여 QuerySet을 얻는다. 각 모델에는 최소한 하나 이상의 Manager가 있고 기본적으로 Objects라고 한다. 다음과 같이 모델 클래스를 통해서 직접 접근할 수 있다. 

```python
>>>Blog.objects
>>>
>>>b = Blog(name='Foo', tagline='Bar')
>>>b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```
Manager는 모델에 대한 QuerySets의 주요한 소스이다. 예를 들어 `Blog.objects.all()`은 데이터베이스의 모든 Blog 객체를 포함한 QuerySet을 리턴한다. 

###Retrieving all objects
테이블에서 객체를 검색하는 방법 중 가장 간단한 것은  **all()**메서드를 사용해 모든 객체를 가져오는것이다. 

```python
>>>all_entries = Entry.objects.all()
```
all()메서드는 데이터베이스에서 모든 객체의 QuerySet을 리턴한다.

###Retrieving specific objects with filters
특정 객체를 찾으려면 필터 조건을 추가해 QuerySet을 구체화해야한다. filter와 exclude를 사용한다.

__filter(**kwargs)__  
주어진 검색 매개변수와 일치하는 객체를 포함한 새로운 QuerySet을 리턴한다.

__exclude(**kwargs)__
주어진 검색 매개변수와 일치하지 않는 객체를 포함한 새로운 QuerySet을 리턴한다. (매개변수와 일치하는 것을 제외하고 리턴)

filter와 exclude 함수 정의에서 검색 매개변수(__**kwargs__)는 다음 문서에서 설명하는 형식이어야 한다. [Field lookups](https://docs.djangoproject.com/en/1.10/topics/db/queries/#field-lookups)

예를 들어 2006년 부터의 블로그 항목의 QuerySet을 구하려면, 다음과 같이 filter()를 사용할 수 있다.

```python
Entry.objects.filter(pub_date__year=2006)
```
filter는 아래와 같이 objects.all()을 붙인 것과 붙지 않은 것이 동일하다. 어떻게 작성하든 전체에서 찾아온다.

```python
Entry.objects.all().filter(pub_date__year=2006)
```

>질문: __year 는 from의 의미를 갖고 있는건가?

####Chaining filters
QuerySet의 필터 결과는 QuerySet그 자체이기 때문에 세부적인 사항들을 연결해 필터할 수 있다.

```python
>>>Entry.objects.filter(
>>>		headline__startswith='What'
>>>).exclude(
>>>		pub_date__gte=datetime.date.today()
>>>).filter(
>>>		pub_date__gte=datetime(2005, 1, 30)
>>>)
```
데이터베이스의 모든 항목의 초기 QuerySet을 가져와 필터를 추가한 후, exclude를 추가하고 다시 다른 필터를 추가했다.  
결과는 2005년 1월 30일부터 지금까지 발행 된 'What'으로 시작하는 제목이 있는 모든 entry를 포함하는 QuerySet이 된다. 

####Filtered QuerySets are unique
QuerySet을 refine할 때마다, 새로운 QuerySet을 얻게된다. (이전의 QuerySet과는 묶이지 않음-영향을 주고 받지 않는다는 말) 각각의 refinement는 저장할 수 있고 재사용할 수 있는 별개의 고유한 QuerySet을 생성한다.

```python
>>> q1 = Entry.objects.filter(headline__startswith='What')
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```
이러한 세 QuerySet은 분리되어있다. 첫 번째는 'What'으로 시작하는 headline을 가진 모든 entry를 포함하는 QuerySet이다. 두번째는 pub_date가 현재이거나 미래인 record를 제외한 첫번째 QuerySet의 subset 이다. 세번째는 pub_date가 오늘이거나 미래인 record만 선택하는 첫번째 QuerySet의 subset이다. 이러한 과정을 모두 거쳐도 첫 번째의 QuerySet은 어떠한 영향도 받지 않으며, q1이 수정된다 하더라도, q2와 q3은 영향을 받지않는다. 

####QuerySets are lazy
QuerySet을 생성하는 것만으로는 데이터베이스에서 어떤 행동도 하지 않는다. 하루종일 필터를 쌓아도, QuerySet이 실행되기 전 까지는 Django는 실제로 query를 실행하지 않는다.

###Retrieving a single object with get()
filter()는 query에 매치되는 객체가 하나밖에 없다고 하더라도 항상 QuerySet을 만든다. (이 경우 단일 요소를 포함하는 QuerySet을 만든다)

query에 매치되는 요소가 하나뿐이라면, 객체를 직접 리턴하는 Manager에 get()메서드를 사용할 수 있다. 

```python
>>> one_entry = Entry.objects.get(pk=1)
```
get()은 filter()와 같은 표현을 사용하며 아래의  [Field lookups](https://docs.djangoproject.com/en/1.10/topics/db/queries/#field-lookups)를 참고하면 된다.

get()을 사용하는 것과 filter()에서 [0]슬라이스를 사용하는 것에는 차이가 있다. 
get()은 일치하는 쿼리가 없으면 DoesNotExist exception을 발생시킨다. (filter()는 결과가 없으면 None을 리턴함)  
또한 get()은 둘 이상의 결과를 찾으면 MultipleObjectsReturned를 발생시킨다. 

###Other QuerySet methods
데이터베이스에 객체를 찾으려고 할 때, all(), get(), filter(), exclude()와 같은 메서드들을 사용할 수 있다. [QuerySet API Reference](https://docs.djangoproject.com/en/1.10/ref/models/querysets/#queryset-api)에서 QuerySet을 리턴하는 메서드 항목을 참고.

###Limiting QuerySets
Python의 array-slicing syntax의 subset을 사용해 QuerySet을 특정한 수의 결과로 제한할 수 있다.   
SQL 문에서 LIMIT, OFFSET과 동일하다. 

```python
# return first 5 objects(LIMIT 5)
>>> Entry.objects.all()[:5]

# return the sixth through tenth objects(OFFSET 5 LIMIT 5)
>>> Entry.objects.all()[5:10]
```
역방향으로 인덱싱하는 것은 지원하지 않는다.
(i.e. Entry.objects.all()[-1])

QuerySet을 슬라이싱하는 것은 query를 실행하지 않고 새로운 QuerySet을 리턴한다. 예외는 아래의 예제와 같이 'step' parameter를 사용하는 경우이다. 

```python
# return a list of every second object of the first 10
>>> Entry.objects.all()[:10:2]
```

리스트가 아닌 단일 객체를 찾는 것이라면(e.g. SELECT foo FROM bar LIMIT 1), 슬라이스 대신 간단한 인덱스를 사용한다.

```python
>>> Entry.objects.order_by('headline')[0]
```
위의 코드는 대략 아래와 같다. 

```python
>>> Entry.objects.order_by('headline')[0:1].get()
```

하지만 일치하는 결과가 없을 경우, 첫 번째 코드는 IndexError를, 두번째 코드는 DoesNotExist를 발생시킨다.

<!--###Field lookups
###Lookups that span relationships
####Spanning multi-vlaued relationships
###Filters can reference fields on the model
###The pk lookup shortcut
###Escaping percent signs and underscores in LIKE statement
###Caching and QuerySets
####When QuerySets are not cached

##Complex lookups with Q objects
##Comparing objects
##Deleting objects
##Copying model instances
##Upadting multiple objects at once
##Related objects
###One-to-many relationships
####Forward
####Follosing relationships "backward"
####Using a custom reverse manager
####Additional methods to handle related objects
###Many-to-many relationships
###One-to-one relationships
###How are the backward relationships possible?
###Queries over related objects
##Falling back to raw SQL-->