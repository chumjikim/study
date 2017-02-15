#QuerySet API reference

##When QuerySets are evaluated
데이터베이스에 실제로 접근하지 않고도 내부적으로 QuerySet을 생성, 필터, 슬라이스, 전달할 수 있다. queryset을 실행하기 전 까지는 데이터베이스에는 어떠한 동작도 하지 않는다. 

다음과 같은 방법으로 QuerySet을 실행할 수 있다.

- **Iteration**  
QuerySet은 반복할 수 있고, 처음 반복을 시작할 때 데이터베이스 쿼리를 실행한다.   
다음 예제는 데이터베이스의 모든 entry의 headline을 출력한다.
	
	```python 
	for e in Entry.objects.all():
		print(e.headline)
	```
>만약 최소한 하나의 결과가 있는지 확인하고 싶은거라면 이 방법보다 `exits()`가 더 효율적이다. 

- **Slicing**  
[Limiting QuerySets](https://docs.djangoproject.com/en/1.10/topics/db/queries/#limiting-querysets)에서 설명한 것처럼, QuerySet은 python의 array-slicing syntax를 이용해 슬라이스 할 수 있다. 실행되지 않은 QuerySet을 슬라이스 하는 것은 실행되지 않은 또다른 QuerySet을 리턴한다. 그러나 슬라이스 구문에 'step' 매개변수를 사용하면  Django는 데이터베이스 쿼리를 실행하고 리스트를 리턴할 것이다. 실행된 QuerySet을 슬라이스해도 리스트가 리턴된다.  

	실행되지 않은 QuerySet을 슬라이스하면 또다른 실행되지 않은 QuerySet을 리턴하지만, 나중에 수정(필터를 추가하거나, 정렬을 수정하거나 등등)하는 것은 허용되지 않는다. 명확한 의미도 없고 SQL로 잘 변환되지 않기 때문이다. 

- **Pickling/Caching**  
자세한 내용은 아래에서..  
여기서 중요한 내용은 결과가 데이터베이스에서 읽혀진다는 것이다.

- **repr()**  
repr()를 호출하면 QuerySet이 실행된다. Python 대화형 인터프리터의 편의를 위한 것이라서 해당 API를 사용하면 결과를 즉시 볼 수 있다. 

- **len()**  
len()을 호출하면 QuerySet이 실행된다. 이건 결과 리스트의 길이를 리턴한다.

	레코드의 개수만 필요한 경우에는 데이터베이스 레벨에서 개수를 세는 것이 효율적이다. 이 때 Django에서 제공하는 count()메서드를 사용할 수 있다. SQL문에서는 SELECT COUNT(*)임.

- **list()**  
list()를 호출하면 강제적으로 QuerySet을 실행시킬 수 있다. 
	
	```python
	entry_list = list(Entry.objects.all())
	```

- **bool()**  
bool(), or, and 혹은 if문으로 QuerySet을 테스트하면 쿼리가 실행된다. 만약 결과가 하나 이상이면 QuerySet은 True이고, 그렇지 않으면 False이다.

	```python
	if Entry.objects.filter(headline="Test"):
		print("There is at least one Entry with the headline TEst")
	```

###Pickling QuerySets
[pickle](https://docs.python.org/3/library/pickle.html#module-pickle) = python 객체 구조를 직렬화(byte로 변환)하거나 비 직렬화하기 위해서 binary protocol을 구현하는 모듈

>객체의 직렬화는 객체의 내용을 바이트 단위로 변환하여 파일 또는 네트워크를 통해 스트림이 가능하게 하는 것을 의미한다.

QuerySet을 Pickle하면, Pickling을 하기 전에 메모리에 모든 결과가 로드된다(데이터베이스에서 가져옴). Pickling은 캐시할 때 precursor로 사용되고, 캐시된 queryset이 리로드 되었을 때 결과가 존재하고 사용할 수 있는 상태여야한다.(데이터베이스에서 읽어오는 건 캐싱의 목적에 맞지 않음)  
QuerySet을 unpickle했을 때, 현재 상태의 데이터베이스 결과가 아니라 pickle 한 시점의 결과를 갖고있다. 

>피클한 애는 캐시한 것이기 때문에, 언피클 할때 현재 데이터베이스에 있는 결과랑 다를 수 있음. 

데이터베이스에서 QuerySet을 다시 만들기 위해 필요한 정보만 pickle하고 싶다면 QuerySet의 속성 query를 pickle하면 된다. 그럼 원래의 QuerySet을 다시 만들 수 있다. 

```python
qs1 = Student.objects.all() 
# '모든 학생 객체를 가져와'라는 쿼리셋을 qs1에 저장함
# SELECT 학생정보 FROM Student 라는 SQL 쿼리 문이 자동으로 생성됨

a = pickle.dumps(qs1)
# qs1은 쿼리셋이기 때문에 피클하면 쿼리를 수행하고 결과를 serialize한다.

b = pickle.dumps(qs1.query)
# qs1.query는 qs1의 query attribute(SQL쿼리문)임 
# qs1.query는 쿼리셋이 아니라 string이기 때문에 쿼리를 수행하지 않고 
# string(SELECT 학생정보 FROM Student)을 serialize한다.

query = pickle.loads(b)
# b를 언피클해서 query에 저장함
# 언피클한 결과는 string이다.

qs2 = MyModel.objects.all()
# 'MyModel의 모든 객체를 가져와'라는 쿼리셋을 qs2에 저장함 
# SELECT 객체정보 FROM MyModel 이라는 SQL 쿼리문(qs2.query)이 자동으로 생성됨 

qs2.query = query
# sq2.query에 query를 저장한다
# SELECT 학생정보 FROM Student(query) 라는 SQL쿼리문이 저장된다. 
# 처음 qs1에 저장한 쿼리문을 restore한다.
# qs2의 쿼리셋이 실행될 때는, qs1을 생성할 때의 시점이 아니라 현재의 시점에서 실행된다.
# 만약 데이터베이스가 변경되었다면 변경 후의 데이터베이스가 적용된다.
```
qs2의 쿼리셋이 실행될 때는, qs1을 생성할 때의 시점이 아니라 현재의 시점에서 실행된다.
만약 데이터베이스가 변경되었다면 변경 후의 데이터베이스가 적용된다.
--> 이게 .query를 피클하는 이유임.

현재 데이터베이스를 기반으로 쿼리셋을 수행한 결과를 serialize하고 싶을 때는 쿼리셋을 피클한다.

##QuerySet API
`class QuerySet(model=None, query=None, using=None)`

일반적으로 QuerySet으로 상호작용 할 때, chaining filter를 사용한다. 이를 위해 대부분의 QuerySet 메서드는 새로운 queryset을 리턴한다. 

QuerySet 클래스는 두 개의 public attribute를 갖고 조절할수 있다. 

**ordered**  
QuerySet을 정렬할 수 있으면 (order_by()메서드를 갖고있거나 기본적으로 정렬기준이 있는 모델이면)True, 아니면 False임.

**db**  
해당 쿼리가 현재 실행될 때 사용할 데이터베이스



##Methods that return new QuerySets
django는 QuerySet에 의해 리턴된 결과의 타입이나 SQL쿼리가 실행되는 방법을 수정하는 다양한 QuerySet refinement method를 제공한다.

###filter()
`filter(**kwargs)`

주어진 lookup parameter와 매치되는 객체를 포함하는 새로운 Queryset을 리턴한다.

lookup parameters(\*\*kwargs)는 [Field lookups](https://docs.djangoproject.com/en/1.10/ref/models/querysets/#id4)에서 설명하고 있는 형식이어야 한다. 여러개의 parameter들은 SQL문에서 **AND**를 통해 join된다.

더 복잡한 쿼리를 실행해야하는 경우에는 Q Objects를 사용할 수 있다. 

###exclude()
`exclude(**kwargs)`

주어진 lookup parameter와 매치되지 않는 객체를 포함하는 새로운 Queryset을 리턴한다.

여러개의 parameter들은 SQL문에서 **AND**를 통해 join되고 전체는 **NOT()**으로 묶여있다. 

아래 예제는 pub_date가 2015년 1월 3일보다 늦고, headline이 'Hello'인 모든 entries를 제외한다.

```python
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3), headline='Hello)
```
SQL문으로는 다음과 같다.

```SQL
SELECT ...
WHERE NOT (pub_date > '2005-1-3' AND headline = 'Hello')
```

아래 예제는 pub_date가 2005년 1월 3일 보다 늦거나 headline이 'Hello'인 모든 entries를 제외한다.

```python
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3)).exclude(headline='Hello')
```
SQL문으로는 다음과 같다.

```SQL
SELECT ... 
WHERE NOT pub_date > '2005-1-3'
AND NOT headline = 'Hello'
```

###annotate()
`annotate(*args, **kwargs)`

제공된 [query expressions]()의 리스트로 QuerySet에서 각 객체에 주석을 붙인다. 표현은 QuerySet의 객체와 관련된 객체에 대해 계산된 간단한 값, 모델 필드에 대한 참조(혹은 관련된 모델), 또는 집합 표현(평균, 합계 등)일 수 있다. 

annotate()의 각 argument는 리턴된 QuerySet의 각 객체에 추가되는 주석이다. 

django가 제공하는 집계 함수는 [Aggregation Functions](https://docs.djangoproject.com/en/1.10/ref/models/querysets/#id5)에 설명되어있다.

Keyword argument를 사용해 지정된 annotation은 Keyword를 annotation의 alias로 사용한다.  
익명 argument는 집계 함수의 이름과 집계되는 모델 필드에 따라 alias를 생성한다.  
단일 필드를 참조하는 집계표현만 익명 argument가 될 수 있다. 다른 것들은 모두 Keyword argument여야한다.

예를 들어, 블로그의 리스트를 다루는 경우, 각 블로그에 entry가 몇개나 만들어졌는지 확인할 수 있다.

```python
>>>from django.db.models import Count
>>>q = Blog.objects.annotate(Count('entry'))
# The name of the first blog
>>>q[0].name
'Blogasaurus'
# The number of entries on the first blog
>>>q[0].entry__count
42
```
Blog 모델은 entry__count를 자체적으로 정의하지 않지만, keyword argument를 이용해 aggregate function을 지정하면, 해당 annotation의 이름을 제어할 수 있다.

```python
>>>q = Blog.objects.annotate(number_of_entries=Count('entry'))
>>>q[0].number_of_entries
42
```

###order_by()
`order_by(*fields)`

기본적으로 QuerySet에 의해 리턴된 결과는 모델 메타의 ordering 옵션에 의해 주어진 ordering tuple에 의해서 정렬된다.   
order_by메소드를 사용하면 QuerySet단위로 오버라이딩 할 수 있다.

```python
Entry.objects.filter(pub_date__year=2005).order_by('-pub_date', 'headline')
```
위의 예제의 결과는 pub_date를 내림차순으로 정렬한 다음 headline을 오름차순으로 정렬한다.

랜덤으로 정렬하고 싶으면 `?`를 사용한다.

```python
Entry.objects.order_by('?')
```
참고: **order_by('?')**는 데이베이스 백엔드에 따라서 느리거나 비쌀 수 있다. 

다른 모델의 필드로 정렬하려면, 모델의 관계를 쿼리할 때와 같은 구문을 사용한다. 필드의 이름에 두 개의 undersocre`__`를 붙이고, 새로운 모델의 필드 이름을 붙인다.

```python
Entry.objects.order_by('blog__name', headline)
```

다른 모델과 관련된 필드로 정렬하려고 할 때, Meta.ordering이 지정되어있지 않으면 django는 관련된 모델의 primary key를 사용해 정렬할 것이다. 

```python
Entry.objects.order_by('blog')
```
위의 예제는 아래와 동일하다. 

```python
Entry.objects.order_by('blog__id')
```

만약 Blog가 ordering = ['name']이라면, 첫 번째 queryset은 다음과 동일할 것이다.

```python
Entry.objects.order_by('blog__name')
```

관련된 필드의 **_id**를 참조해 JOIN을 사용하지 않고도 queryset을 정렬하는 것이 가능하다. (foreignkey로 연결되있으면 연결한 모델의 primarykey가 `model_id`라는 필드로 들어와있기 때문에, 연결한 모델을 거치지 않고 바로 가져올 수 있다.)

```python
# No join
Entry.objects.order_by('blog_id')

# Join
Entry.objects.order_by('blog__id')
```
**asc()**나 **desc()**를 호출해 [query expressions](https://docs.djangoproject.com/en/1.10/ref/models/expressions/)에 의해 정렬할 수도 있다.

```python
Entry.objects.order_by(Coalesce('Summery', 'headline').desc())
```

**distinct()**를 사용하는 경우, 관련 모델의 필드로 정렬 할 때는 주의해야한다. 다음에 나오는 distinct()에 대한 설명을 보면 관련 모델 ordering이 예상된 결과를 어떻게 바꿀 수 있는지 알 수 있다. 

대소문자를 구분해 정렬하는 방법은 없다. Django가 결과를 정렬해도 데이터베이스에서는 일반적인 정렬로 처리한다.

**Lower**를 사용해 소문자로 변환된 필드로 정렬할 수 있다.

```python
Entry.objects.order_by(Lower('headline').desc())
```

> 질문: case sensitive하지 않은데 Lower로 변환해서 정렬하는 이유가 있나?
>   
> `c, a, b, A, D, B, E`이걸 바로 정렬하면 `A,B,D,E, a, b, c`가 됨(대문자가 ASCII코드에서 앞에 있어서 이렇게 정렬이된다). 이걸 대소문자 구분없이 정렬하려면 Lower를 사용할 수 있다. 결과는 `a, A, b, B, c, D, E` 


만약 기본 정렬 값도 적용하고 싶지 않으면 parameter를 넣지 않고 **order_by()**라고 쓰면 된다.

각 order_by()호출은 이전의 정렬을 지우기 때문에 아래 예제와 같이 사용하면, pub_date로만 정렬이 된다.

```python
Entry.objects.order_by('headline').order_by('pub_date')
```

__WARNING__  
Odering은 free operation이 아니다. ordering 이 적용된 각 필드는 데이터베이스에 비용을 들이고 있는 것이다. 추가된 각각의 ForeignKey는 암묵적으로 모든 default ordering이 포함되어있다. 

만약 쿼리가 어떠한 ordering도 지정되지 않았다면, 데이터베이스에서 지정되지 않은 순서로 결과가 리턴된다.  특정 ordering은 결과에서 각 객체를 고유하게 식별하는 필드의 셋으로 정렬할 경우에만 정렬이 보장된다. 예를들면 **name** 필드가 고유하지 않은 경우에 이름 순서로 정렬을 하면, 같은 이름의 객체가 항상 같은 순서로 표시되는 것은 아니다.

###reverse()
`reverse()`

reverse()메서드를 사용해 qeuryset의 요소가 리턴되는 순서를 바꾼다. reverse()를 다시 호출하면 순서가 정방향으로 복원된다.

queryset에서 마지막 5개의 아이템을 찾고 싶다면, 다음과 같이 작성한다. 

```python
my_queryset.reverse()[:5]
```
이 방법은 Python에서 시퀀스를 마지막부터 슬라이스하는 것과 완전히 같지않다. 위의 예제는 마지막 아이템을 리턴하고, 마지막에서 두번째 아이템을 리턴한다. Python 시퀀스에서는 seq[-5:]라고 하면 마지막에서 다섯번째 항목이 먼저 보인다. django는 마지막에서부터 슬라이스하는 것을 지원하지 않는다. SQL에서 효과적으로 수행하는 것이 불가능하기 때문이다. 

또한 reverse()는 ordering이 정의된 QuerySet에서만 호출해야한다. ordering이 지정되지 않은 QuerySet에서 reverse()를 호출하는 것은 아무 영향이 없다. 

###distinct()
`distinct(*fields)`

SQL query의 SELECT DISTINCT문을 사용해 새로운 QuerySet을 리턴한다. 이렇게 하면 query 결과에서 중복된 row가 제거된다.   

기본적으로 QuerySet은 중복 row를 삭제하지 않는다. 실제로 Blog.objects.all()과 같은 간단한 query는 중복된 결과가 발생할 가능성이 별로 없어서 문제가 되지 않는다. 하지만, query가 여러 테이블에 걸쳐져 있는 경우 QuerySet을 실행할 때, 중복된 결과를 얻을 수도 있다. 이런 상황에서 distinct()를 사용한다. 

PostgreSQL에서만 DISTINCT가 적용되야하는 필드의 이름을 지정하기 위해 위치 argument를 전달할 수 있다. 이것은 SELECT DISTINCT ON SQL query로 변환된다. 일반적인 distinct()호출을 위해서 데이터베이스는 고유한 row를 판별할 때,  각 row의 각 field를 비교한다. 지정된 field이름으로 distinct()가 호출되면 데이터베이스는 지정된 필드 이름만 비교한다. 

아래 예제는 첫 번째 이외에는 PostgreSQL에서만 작동한다. 

```python
>>> Author.objects.distinct()
[...]

>>> Entry.objects.order_by('pub_date').distinct('pub_date')
[...]

>>> Entry.objects.order_by('blog').distinct('blog')
[...]

>>> Entry.objects.order_by('author', 'pub_date').distinct('author', 'pub_date')
[...]

>>> Entry.objects.order_by('blog_name', 'mod_date').distinct('blog__name', mode_date)
[...]

>>> Entry.objects.order_by('author', 'pub_date').distinct('author')
[...]
```

###values()
`values(*fields)`

iterable로 사용될 경우에는 모델 인스턴스가 아닌, dictionariy를 리턴하는 QuerySet을 리턴한다.

각 dict는 모델 객체의 속성 이름에 해당하는 key를 사용해 객체를 나타낸다.

아래의 예제에서는 values()의 dict를 일반 모델 객체와 비교한다.

```python
# This list contains a Blog object.
>>> Blog.objects.filter(nmae__startswith='Beatles')
<QuerySet [<Blog: Beales Blog>]>

# This list contains a  dictionary.
>>> Blog.objects.filter(name__startswith='Beatles').values()
<QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>
```

values() 메서드는 선택적으로 위치 argument를 갖는다. `*field`는 SELECT 가 제한되어야하는 필드 이름을 지정한다. 필드를 지정했다면, 각 dict는 지정한 필드의 field keys와 values 만 포함된다. 만약 필드를 지정해주지 않으면, dict에는 데이터베이스 테이블의 모든 필드에 대한 키/값이 포함된다.

```python
>>> Blog.objects.values()
<QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>
>>> Blog.objects.values('id', 'name')
<QuerySet [{'id': 1, 'name': 'Beatles Blog'}]>
```

- ForeignKey인 foo라는 필드가 있을 경우, values()호출은 foo\_id라는 dict key를 리턴한다. 이것은 실제로 값을 저장하는 숨겨진 모델 속성의 이름이기 때문이다. (foo 속성은 관련된 모델을 참조한다) 
values()를 호출하고, 필드 이름을 전달 할 때, foo 또는 foo\_id를 전달하면 같은 결과를 얻게된다. (dict key는 전달한 필드 이름과 일치한다)

	```python
	>>> Entry.objects.values()
	<QuerySet [{'blog_id': 1, 'headline': 'First Entry', ...}, ...]>
	
	>>> Entry.objects.values('blog')
	<QuerySet [{'blog': 1}, ...]>
	
	>>> Entry.objects.vlaues('blog_id')
	<QuerySet [{'blog_id': 1}, ...]>
	```
- distinct()와 함께 values()를 사용하면, 결과의 ordering에 영향을 줄 수 있다. 자세한건 dict()의 메모를 참고.
- extra()를 호출한 후 values()를 사용하면, extra()의 select argument로 정의된 모든 필드가 명시적으로  values()의 호출에 포함되어야한다. values()호출 후 만들어진 extra()호출을 하면 extra selected 필드는 무시된다.
- values()뒤에 only()와 defer()를 호출하는 건 의미가 없기 때문에, NotInplementedError가 발생한다.

적은 수의 필드에서 값만 필요하고, 모델 인스턴스 객체의 기능이 필요하지 않을 때 유용하다. 이런 경우에는 사용할 필드만 선택하는 것이 더 효율적이다. 

values()를 호출 한 후, filter(), order_by()와 같은 메서드를 호출할 수 있다. 이말은 이러한 두 호출이 동일하다는 것을 의미한다. 

```python
Blog.objects.values().order_by('id')

Blog.objects.order_by('id').values()
```

Django를 만든 사람들은 values()와 같은 출력에 영향을 미치는 선택적인 메서드는 뒤에 따라오고, 모든 SQL에 영향을 주는 메서드를 먼저 넣는 것을 선호하지만 실질적으로는 별 상관이 없다. 

OneToOneField, ForeignKey 또는 ManyToManyField 속성을 통해 역방향 관계에 있는 관련 모델의 필드를 참조할 수 도 있다. 

```python
>>> Blog.objects.values('name', 'entry__headline')
<QuerySet [{'name': 'My blog', 'entry__headline': 'An entry'}, {'name': 'My blog', 'entry__headline': 'Another entry'}, ...]>
```

###values_list()
`values_list(*fields, flat=False)`

반복된 후 dict를 리턴하지 않고 tuple을 리턴한다는 점을 제외하고는 values()와 유사하다. 각 튜플은 values_list()호출로 전달된 각 필드의 값을 포함한다. 그렇기 때문에 첫 번 째 아이템은 첫 번째 필드가 된다. 

```python
>>> Entry.objects.values_list('id', 'headline')
[(1, 'First entry'), ...]
```

단일 필드만 전달하면 flat parameter를 사용할 수 있다. flat이 True면, tuple대신 단일 값을 리턴한다. 

```python
>>> Entry.obejcts.values_list('id').order_by('id')
[(1,),(2,),(3,), ...]

>>> Entru.objects.values_list('id', flat=True).order_by('id')
[1, 2, 3, ...]
```

둘 이상의 필드가 있을 때 flat으로 전달하면 에러가 난다. 

values_list()에 값을 전달하지 않으면, 모델의 모든 필드가 선언된 순서대로 리턴한다.

특정 모델의 인스턴스의 특정 필드값을 가져오는 것이 일반적인 경우이다. values_list()뒤에 get()을 호출하면 된다.

```python
>>> Entry.objects.values_list('headline', flat=True).get(pk=1)
'First entry'
```

values()와  values_list()는 둘 다 특정한 케이스에 최적화되어있다. :
모델 인스턴스를 생성하는 오버헤드 없이 데이터의 하위집합을 검색한다. 이것은 many-to-many그리고 다중 값 관계(역방향 foreignkey의 one-to-many관계)를 다룰때와는 거리가 멀다 . one row, one object 라는 가정이 성립하지 않기 때문임.

ManyToManyField를 통한 쿼리동작

```python
>>> Author.objects.values_list('name', 'entry__headline')
[('Noam Chomsky', 'Impressions of Gaza'),
('George Orwell', 'Why Socialists Do Not Believe in Fun'),
('George Orwell', 'In Defence of English Cooking'),
('Don Quixote', None)
]
```
여러 entry가 있는 author는 여러 번 나타나고, entry가 없는 author는 entry의 headline에 대해 none을 갖는다.

마찬가지로 역방향 foreignkey를 query할 때, author가 없는 entry에 대해서는 None이 나온다.

```python
>>> Entry.objects.values_list('authors')
[('Noam Chomsky',),('George Orwell',),(None,)]
```

###dates()
`dates(field, kind, order='ASC')`

QuerySet의 내용 안에서 특정 종류의 사용가능한 모든 날짜를 나타내는 datetime.date객체의 리스트로 QuerySet을 리턴한다. 

**field:** 모델의 DateField 이름이어야한다.    
 
**kind:** "year", "month", "day"중 하나이어야한다.  
결과 리스트에 있는 각 datetime.date객체는 주어진 `type`으로 잘린다.  

- "year" 해당 필드의 모든 고유한 연도 값 리스트를 리턴한다. 
- "month" 해당 필드의 모든 연도/월 값 리스트를 리턴한다.
- "day" 해당 필드의 모든 연도/월/일 값 리스트를 리턴한다.

**order:** 결과를 정렬하는 방법을 지정하는 것이고 기본값은 'ASC'이다. 'ASC'또는 'DESC'여야한다.

```python
>>> Entry.objects.dates('pub_date', 'year')

>>> Entry.objects.dates('pub_date', 'month')

>>> Entry.objects.dates('pub_date', 'day')

>>> Entry.objects.dates('pub_date', 'day', order='DESC')

>>> Entry.objects.filter(headline__contains='Lennon').dates('pub_date', 'day')
```
###datetimes()
`datetimes(field_name, kind, order='ASC', tzinfo=None)`

QuerySet의 내용 안에서 특정 종류의 사용가능한 모든 날짜를 나타내는 datetime.datetime객체의 리스트로 QuerySet을 리턴한다.

**field_name:** 모델의 DateTimeField 이름이어야한다.

**kind:** "year", "month", "day", "hour", "minute", "second" 중 하나이어야한다. 결과 리스트에 있는 각 datetime.datetime 객체는 주어진 `type`에 따라 잘린다. 

**order:** 결과를 정렬하는 방법을 지정하는 것이고 기본값은 'ASC'이다. 'ASC'또는 'DESC'여야한다.

**tzinfo:** dateimes가 잘리기 전에 변환되는 시간대를 정의한다. 주어진 datetime은 사용중인 시간대에 따라 다른 표현을 갖는다. 이 parameter는 datetime.tzinfo객체여야한다. None일 경우 Django는 현재 시간대를 사용한다. USE_TZ가 False일 경우에는 효과가 없다. 

###none()
`none()`

none()을 호출하면 객체를 리턴하지 않는 queryset이 만들어지고, 결과에 접근할 때 query가 실행되지 않는다. qs.non() qeuryset은 EmptyQuerySet의 인스턴스이다.

```python
>>> Entry.objects.none()
<QuerySet []>
>>> from django.db.models.query impory EmptyQuerySet
>>> isinstance(Entry.objects.non(), EmtyQuerySet)
True
```

###all()
`all()`

현재 QuerySet(혹은 QuerySet 서브클래스)의 복사본을 리턴한다. 모델 매니저나 QuerySet을 전달하고 그 결과에 대해 추가적인 필터링을 하는 걸 원할때 유용하다. 두 객체에서 모두 all()을 호출하면, 작업 할 QuerySet을 명확히 갖게된다. 

QuerySet이 실행되면, 일반적으로 결과를 캐시한다. QuerySet이 실행된 후 데이터베이스의 데이터가 변경된 경우, 이전 실행된 QuerySet에서 all()을 호출해 동일한 쿼리에 대해 업데이트된 결과를 얻을 수 있다. 

###select_related()
`select_related(*fields)`

ForeignKey 관계를 따르는 QuerySet을 리턴하고, 쿼리를 실행할 때 추가적인 관련 객체 데이터를 선택한다. 이것은 더 복잡한 쿼리의 결과로 성능을 향상하지만, 나중에 ForeignKey 관계를 사용하면 데이터베이스 쿼리가 필요하지 않게된다. 

아래의 예제에서는 일반적인 조회와 select_reltaed()조회를 비교한다.  
기본은 다음과 같다. 

```python
# Hits the database.
e = Entry.objects.get(id=5)

# Hits the database agin to get the related Blog object.
b = e.blog
```
다음은 select_related 조회이다.

```python
# Hit the database
e = Entry.objects.select_related('blog').get(id=5)

# Doesn't hit the database, 
# because e.blog has been prepopulated
# in the previous query.
b = e.blog
```

select_related()는 객체의 queryset과 함께 사용할 수 있다. 

```python
from django.utils import timezone

# Find all the blogs with entries scheduled to be published in the future.
blogs = set()

for e in Entry.objects.filter(pub_date__gt=timezone.now()).select_related('blog'):
	# Without select_related(), this would make a database query for each
	# loop iteration in order to fetch the related blog for each entry

	blogs.add(e.blog)
```

filter()와 select_related()의 chaining 순서는 별로 중요하지 않다. 다음의 queryset들은 동일하다.

```python
Entry.objects.filter(pub_date__gt=timezone.now()).select_related('blog')

Entry.objects.select_related('blog').filter(pub_date__gt=timezone.now())
```

ForeignKey를 쿼리하는 것과 유사한 방법으로 ForeignKey를 추적할 수도 있다.
다음과 같은 모델을 갖고 있다고 가정해보면,

```python
from django.db import models

class City(models.Model):
	# ...
	pass
	
class person(models.Model):
	# ...
	hometown = models.ForeignKey(
		City,
		on_delete=models.SET_NULL,
		blank=True,
		null=True,
	)
	
class Book(models.Model):
	# ...
	author = models.ForeignKey(Person, on_delete=models.CASCADE)
```
`Book.objects.select_related('author__hometown').get(id=4)`는 관련된 Person과 관련된 City를 캐시한다. 

```python
b = Book.objects.select_related('author__hometown').get(id=4)
p = b.author # Doesn't hit the databas
c = p.hometown # Doesn't hit the databas

b = Book.objects.get(id=4) # No selet_related() in this example.
p = b.author # Hit the databas
c = p.hometown # Hit the databas
```
select_related()에 전달된 필드 리스트에서 ForeignKey또는 OneToOneField를 참조할 수 있다. 

또한 select_related()에 전달된 필드 리스트에서 OneToOneField의 역참조도 가능하다. 즉, 필드가 정의된 객체로 다시 OneToOneField를 되돌릴 수 있다. 필드 이름을 지정하는 대신, 관련 객체의 필드에 related_name을 사용한다. 

QuerySet에서 이전에 호출된 select_related에 의해 추가된 관련 필드 리스트를 지우려면, None을 parameter로 전달하면된다.

```python
>>> without_relations = queryset.select_related(None)
```

select_reltaed의 chaining은 다른 메서드들과 비슷한다.  
select_related('foo', 'bar')는 select_related('foo').select_related('bar')와 동일하다. 

###prefetch_related()
`prefetch_related(*lookups)`

지정된 lookup마다 관련된 객체를 하나의 batch에서 자동으로 검색하는 QuerySet을 리턴한다.

select_related와 유사한 목적을 갖고있다. 둘 다 객체에 접근해 발생하는 데이터베이스의 쿼리가 넘치는 상황을 막을 수 있도록 설계되었지만 전략은 상당히 다르다. 

select_related 는 SQL join 을 생성하고 SELECT 문에 관련 객체의 필드를 포함해 작동한다. 때문에 select_related는 동일한 데이터베이스 쿼리에서 관련 객체를 가져온다. 그러나 많은 관계에 참여함으로써 생기는 더 큰 결과 집합을 피하기 위해서, select_related는 단일 값 관계ForeignKey와 One-To-One)로 제한된다.

반면, prefetch_related 각 관계에 대해 별도로 조회하고, python에서 joining을 한다. 이것은 select_related가 지원하는 ForeignKey와 OneToOne관계 이외에도 select_related에서 할 수 없었던, ManyToMany와 ManyToOne객체를 prefetch할 수 있다. 
또한 GenericRelation과 GenericForeignKey의 prefetch도 지원하지만, 동질적인 결과 집합으로 제한되어야한다. 예를 들어, GenericForeignKey가 참조하는 객체 prefetch는 쿼리가 하나의 ContentType으로 제한되는 경우에만 지원된다. 

다음과 같은 모델을 갖고있다고 가정해본다.

```python
from django.db import models

class Topping(models.Model):
	name = models.CharField(max_length=30)
	
class Pizza(models.Model):
	nmae = models.CharField(max_length=50)
	toppings = models.ManyToManyField(Topping)
	
	def __str__(self):
		return '%s (%s)' % (
			self.name,
			', '.join(topping.name for topping in self.toppings.all()),
		)
```
실행시켜보면, 다음과 같은 결과가 된다.

```python
>>> Pizza.objects.all()
["Hawaiian (ham, pineapple)", "Seafood(prawns, smoked salmon)" ...]
```
문제는 `Pizza.__str__()`가 `self.toppingsall()`요청할 때 마다 데이터베이스 쿼리를 해야하기 때문에, `Pizza.objects.all()`은 Pizza QuerySet의 모든 아이템에 대해 Toppings 테이블에 대한 쿼리를 실행한다.

prefetch_related를 사용하면 쿼리를 두개로 줄일 수 있다.

```python
>>> Pizza.objects.all().prefetch_related('toppings')
```
이것은 각 Pizza에 대한 self.toppigs.all()을 의미한다. 이제 self.toppings.all()이 호출될 때 마다 해당 아이템에 대한 데이터베이스로 이동하지 않고, 단일 쿼리로 채워진 prefetch된 QuerySet 캐시에서 해당 아이템을 찾는다.  
즉, 관련된 모든 토핑을 단일 쿼리에서 가져오고, 관련 결과의 미리 채워진 캐시가 있는 QuerySet을 만드는 것에 사용된다. 이 QuerySet이 self.toppings.all()호출에 사용된다. 

<!--prefetch_related()의 추가 쿼리는 QuerySet이 실행을 시작하고, 기본 쿼리가 실행 된 후에 실행된다. -->

###extra()
`extra(select=None, where=None, params=None, tables=None, order_by=None, select_params=None)`

때로는 Django 쿼리 문법만으로는 복잡한 WHERE절을 쉽게 표현할 수 없다. 이러한 경우에 Django는 QuerySet에 의해 생성된 SQL에 특정 절을 삽입하기 위한 hook인 extra() QuerySet 수정자를 제공한다.

**최후의 방법으로 사용해야한다**
이 API는 곧 deprecate 될 것임.
다른 QuerySet으로 표현할 수 없는 경우에만 사용해야 한다.

**WARNING**
extar()를 사용할 때 마다 매우 조심해야한다. 사용할 때 마다 SQL injection 공격으로 부터 보호하기 위해 params를 사용해 사용자가 제어할 수 있는 모든 parameter를 escape처리해야한다.

이러한 추가 조회는 다른 데이터베이스 엔진(SQL코드를 명시적으로 작성하고 있기 때문)으로 이식할 수 없고 DRY원칙을 위반할 수 있기 때문에 가능하면 피해야한다.

params, select, where, tables 중 하나 이상을 지정해야한다. argument는 필요하지 않지만 적어도 하나는 사용해야한다.

- select  
	select argument는 SELECT절에 추가적인 필드를 넣을 수 있다. 해당 속성을 계산하는데 사용할 속성이름을 SQL절로 맵핑하는 dictionary여야한다. 
	
- where / tables  
	where를 사용해 명시적으로 SQL WHERE절을 정의할 수 있다.(비 명시적인 join을 수행하기 위해서)  
	tables를 사용해 SQL FROM절에 테이블을 수동으로 추가할 수 있다.  
	where와 tables 모두 문자열 리스트를 가져온다. 모든 where parameter는 다른 검색 기준과 'AND'된다.
	 
- order_by  
	extra()를 통해 포함된 새로운 필드나 테이블 중 일부를 사용해 결과 QuerySet을 정렬해야하는 경우, order_by parameter를 extra()에 사용하고 문자열을 전달한다.
	
- params  
	위에서 설명한 where parameter는 표준 Python 데이터베이스 문자열 자리 표시자('%s')를 사용해 데이터베이스 엔진이 자동으로 인용해야하는 parameter를 나타낼 수 있다. params argument 는 대체될 추가 parameter의 리스트이다. 

###defer()
`defer(*fields)`

어떤 복잡한 데이터 모델링 상황에서는 모델에 많은 필드들이 포함될 수 있고 그중 어떤 필드는 많은 데이터를 가지고 있을 수 있다. 또는 Python 객체로 변환하는 것에 많은 비용이 드는 필드가 있을 수도 있다. 특정한 필드에 대해 필요성을 느끼지 못하는 상황에서 QuerySet의 결과를 사용하는 경우, 처음 데이터를 가져올 때 Django에 데이터베이스에서 검색하지 않도록 할 수 있다. 

로드되지 않도록 defer()에 필드의 이름을 전달한다.

```python
Entry.objects.defer('headline', 'body')
```
지연 필드가 있는 QuerySet은 여전히 모델 인스턴스를 리턴한다. 해당 필드에 접근하면 각 지연 필드는 데이터베이스에서 검색된다. (한번에 모든 지연필드가 검색되는 것이 아니라, 한번에 하나씩 검색됨)

defer()를 여러번 호출 할 수 있다. 각 호출은 지연필드에 새 필드를 추가한다. 

```python
# Defers both the body and headline fields
Entry.objects.defer('body').filter(rating=5).defer('headline')
```

필드가 defer set에 추가되는 순서는 중요하지 않다. 이미 지연된 필드 이름으로 defer()호출을 해도 괜찮다.(필드는 여전히 지연된다)

두 개의 underscore`__`를 사용해 관련 필드를 구분해서 사용하면, 관련 모델에서 필드 로딩을 지연할 수 있다.(관련 모델이 select_related()를 통해 로드되는 경우)

```python
Blog.objects.select_related().defer('entry__headline', 'entry__body')
```
만약 deferred field set을 지우려면, defer()에 parameter로 None을 전달하면 된다. 

```python
# Load all fields immediately
my_queryset.defer(None)
```

모델의 일부 필드는 defer()를 요청해도 안되는 경우가 있다. primary key는 로딩을 지연할 수 없다. select_related()를 사용해 관련 모델을 검색하는 경우, 기본 모델에서 관련 모델로 연결하는 필드 로딩을 defer하면 오류가 발생한다. 


###only()
`only(*fields)`

only()메서드는 defer()메서드의 반대이다. 
모델을 검색할 때, 지연되서는 안되는 필드를 호출한다.  
거의 모든 필드가 지연될 필요가 있는 모델을 가지고 있다면, 보완적인 필드 셋을 지정하기 위해서만 only()를 사용하면 코드가 더 간단해질 수 있다. 

모델에 name, age, biography라는 필드를 가지고 있다고 가정해보자. 아래의 두 queryset은 지연 필드의 관점에서 동일하다.

```python
Person.objects.defer('age', 'biography')
Person.objects.only('name')
```
only()를 호출 할 때 마다 즉시 로드할 필드 셋을 바꾼다. 메소드의 이름은 mnemonic: 해당 필드만 즉시 로드하고 나머지는 지연된다. 따라서 only()를 연속적으로 호출하면 최종 필드만 고려된다. 

```python
# This will defer all fields except the headline.
Entry.objects.only('body', 'rating').only('headline')
```

defer()는 점진적으로 작동하기 때문에(지연 목록에 필드를 추가함) only()와 defer()에 대한 호출을 결합해서 논리적으로 작동하게 할 수 있다. 

```python
# Final result is that everything except 'headline' is deferred
Entry.objects.only('headline', 'body').defer('body')

# Final result loads headline and body immediately # (only() replaces any existing set of fields)
Entry.objects.defer('body').only('headline', 'body')
```

only()를 사용하고 select_related()를 사용해 요청된 필드를 생략하면 에러가 발생한다. 

###using()
`using(alias)`

이 메서드는 둘 이상의 데이터베이스를 사용하는 경우, QuerySet을 실행할 데이터베이스를 제어할 때 사용한다. 이 메서드는 DATABASE에 정의된 데이터베이스의 alias를 유일한 argument로 받는다. 

```python
# queries the database with the 'default' alias.
>>> Entry.objects.all()

# queries the database with the 'backup' alias
>>> Entry.objects.using('backup')
```
###select\_for\_update()
`select_for_update(nowait=False)`



###raw()
`raw(raw_query, params=None, translations=None)`

-

##Methods that do not return QuerySets

다음 QuerySet 메서드는 QuerySet을 실행하고 QuerySet 이외의 것을 리턴한다.

이런 메서드는 캐시를 사용하지 않고, 호출할 때 마다 데이터베이스를 조회한다. 

###get()
`get(**kwargs)`

주어진 lookup parameter와 매치되는 객체를 리턴한다. parameter는 [Field lookups](https://docs.djangoproject.com/en/1.10/ref/models/querysets/#id4)에 설명된 포맷이어야한다. 

get()은 둘 이상의 객체가 발견되면 MultipleObjectsReturned를 발생시킨다. MultipleObjectsReturned 예외는 모델 클래스의 속성이다. 

get()은 지정된 parameter에 대한 객체를 찾을 수 없는 경우 DoesNotExist 예외를 발생시킨다. 이 예외는 모두 클래스의 속성이다. 

```python
Entry.objects.get(id='foo') # rases Entry.DoesNotExist
```

DoesNotExist 예외는 django.core.exceptions.ObjectDoesNotExit에서 상속된다. 따라서 여러개의 DoesNotExist 예외를 지정할 수 있다. 

```python
from django.core.exceptions import ObjectDoesnotExit

try: 
	e = Entry.objects.get(id=3)
	b = Blog.objects.get(id=1)
except ObjectDoesNotExist:
	print("Either the entry or blog doen't exit")
```
Queryset이 한 row를 리턴할 거라고 예상되면 argument 없이 get()을 사용하여 해당 row에 대한 객체를 리턴할 수 있다.

```python
entry = Entry.objects.filter(...).exclude(...).get()
```

###create()
`create(**kwargs)`

객체를 생성하고 저장하는 것을 한번에 한다.
 
```python
p = Person.objects.create(first_name='Bruce', last_name='Spingsteen'
```
위의 코드는 아래와 같다. 

```python
p = Person(first_name='Bruce', last_name='springsteen')
p.save(force_insert=True)
```

[force_insert](https://docs.djangoproject.com/en/1.10/ref/models/instances/#ref-models-force-insert) parameter는 새로운 객체가 항상 생성된다는 것을 의미한다. 일반적으로는 이것에 대해 걱정할 필요는 없다. 하지만 모델에 사용자가 설정한 primary key값이 포함되어있고, 해당 값이 이미 데이터베이스에 있는 경우에는, primary key는 고유해야하기 때문에 IntegrityError가 발생하고 create() 호출은 실패할 것이다. 이런 경우에는 예외처리를 준비해야한다. 

###get\_or\_create()
`get_or_create(default=None, **kwargs)`

주어진 kwargs(모델의 모든 필드가 기본값이면 비어있을 수도 있음)로 객체를 찾는 방법이다. 필요한 경우(찾았는데 없으면?)에는 생성한다. 

(object, created)의 튜플을 리턴한다. object는 검색되거나 생성된 객체이고, created는 새 객체가 만들어졌는지 여부를 지정하는 boolean이다. 

```python
try:
	obj = Person.objects.get(first_name='John', last_name='Lennon')
excepy Person.DoesNotExist:
	obj = Person(first_name='John', last_name='Lennon', birthday=date(1940, 10, 9))
	obj.save()
```
위의 코드는 모델 필드 수가 증가하면, 다루기 힘들어진다. 이 코드는 get\_or\_created()를 사용해 아래와 같이 작성할 수 있다. 

```python
obj, created = Person.objects.get_or_create(
	first_name='John',
	last_name='Lennon',
	defaults={'birthday': date(1940, 10, 9)},
)
```

get\_or\_created()로 전달된 keyword argument(선택사항인 defaults를 제외하고)는 get()호출에 사용된다. 만약 객체를 찾으면, get\_or\_create()는 해당 객체의 tuple을 리턴하고  False를 리턴한다. 만약 다중 객체를 찾으면, get\_or\_create는 MultipleObjectReturned를 발생시킨다.  
객체를 찾지 못한 경우에는, get\_or\_create()는 인스턴스화하고 새 객체를 저장한다. 새로 만들어진 객체의 tuple과 True를 리턴한다. 새로운 객체는 대략 아래의 알고리즘에 따라 생성된다.

```python
parmas = {k: v for k, v in kwargs.items() if '__' not in k}
params.update(defaults)
obj = self.model(**params)
obj.save()
```
두 개의 underscore를 포함하지 않는 `defaults`가 아닌 keyword argument로 시작한다.  defaults의 내용을 추가하고, 필요한 경우 key를 오버라이딩하고, 결과를 모델 클래스의 keyword argument로 사용한다.

위의 알고리즘은 단순화하는 것이지만 모든 세부사항을 모두 포함하고 있다. 내부적으로는 error-checking 을 좀 더 하고, 여타 컨디션을 조절한다. 

만약 defaults라는 이름의 필드를 갖고있고, get\_or\_create()로 정확한 조회를 하고 싶으면 **defaults__exact**를 사용하면 된다.

```python
Foo.objects.get_or_create(defaults__exact='bar', defaults={'defaults': 'baz'}
```

사용자가 지정한 primary key를 사용할 때, get\_or\_create() 메서드는 create()메서드와 유사한 에러 동작을 한다. 객체를 만들어야하는데 key가 이미 데이터베이스에 있으면, IntegrityError가 발생한다. 

###update\_or\_create()
`update_or_create(default=None, **kwargs)`

주어진 kwargs를 통해 객체를 업데이트한다. 필요하면 새로 생성한다. defaults는 객체 업데이트에 사용되는 (field, value) 쌍의 dict이다. 

객체가 생성되거나 업데이트되면, (object, created) tuple을 리턴한다. created는 새로운 객체가 생성되었는지 여부를 나타내는 boolean이다. 

update\_or\_create 메서드는 주어진 kwargs를 기반으로 데이터베이스에서 객체를 가져오려고 한다. 일치하는 것을 찾으면, defaults dict에 전달된 필드를 업데이트한다. 

```python
defaults = {'first_name': 'Bob'}
try:
	obj = Person.objects.get(first_name='john', last_name='Lennon')
	for key, value in defaults.items():
		setattr(obj, key, value)
	obj.save()
except Person.DoesNotExist:
	new_values = {'first_name': 'John', 'last_name': 'Lennon'}
	new_values.update(defaults)
	obj = Person(**new_values)
	obj.save()
```
위의 코드는 모델 필드 수가 증가하면, 다루기 힘들어진다. 이 코드는 update\_or\_create()를 사용해 아래와 같이 작성할 수 있다. 

```python
obj, created = Person.objects.update_or_create(
	first_name='John',  
	last_name='Lennon',
	defaults={'first_name': 'Bob'}.
)

```

###bulk_create()
`bulk_create(objs,batch_size=None)`

이 메서드는 제공된 객체의 리스트를 데이터베이스에 효율인 방법으로 삽입한다. (얼마나 많은 객체가 있던 하나의 query를 사용한다)

```python
>>> Entry.objects.bulk_create([
		Entry(headline='This is a test),
		Entry(headline='This is only a test'),
])
```
여기에는 몇 가지 주의사항이 있다.

- 모델의 save()메서드는 호출할 수 없고, pre_save와 post_save신호는 전달되지 않는다.
- 다중 테이블 상속 시나리오에서는 하위 모델에 적용되지 않는다.
- 모델의 primary key가 Autofield인 경우 데이터베이스 백엔드가 지원하지 않으면 primary key의 속성을 검색하거나 설정하지 않는다.
- ManyToMany 관계에서는 작동하지 않는다.

**batch_size** parameter는 한 쿼리에서  얼마나 많은 객체를 생성할 지 컨트롤한다. 기본은 하나의 batch에 모든 객체를 생성하는 것이다. (쿼리 당 최대 999개의 변수가 사용되는 것이 기본인 SQLite는 예외임)

###count()
`count()`

데이터베이스에서 QuerySet과 일치하는 객체의 개수를 정수로 리턴한다. count()는 예외가 발생하지 않는다. 

```python
# Returns the total number of entries in the database.
Entry.objects.count()

# Returns the number of entries whose headline contains 'Lennon'
Entry.obejcts.filter(headline__contains='Lennon').count()
``` 

count()호출은 뒤에서 SELECT COUNT(*)를 실행한다. 따라서 모든 레코드를 python 객체에 로드하고 결과에 대해 len()을 호출하는 대신, 항상 count()를 사용해야한다. 
(메모리에 객체를 로드해야하는 경우가 아니라면, len()이 더 빠르다)

QuerySet의 아이템 개수를 얻고, 이를 통해 모델 인스턴스를 검색하는 경우(예를 들면 반복같은걸로), len(queryset)을 사용하는것이 더 효율적이다.   

###in_bulk()
`in_bulk(id_list=None)`

primary-key 값의 리스트를 받아서 각 primary-key값을 주어진 ID의 객체의 인스턴스와 매핑하는 dict를 리턴한다. 만약 리스트가 주어지지않으면 쿼리셋의 모든 객체가 리턴된다.

```python
>>> Blog.objects.in_bulk([1])
{1: <Blog: Beatles Blog>}

>>> Blog.objects.in_bulk([1, 2])
{1: <Blog: Beatles Blog>, 2: <Blog: Cheddar Talk>}

>>> Blog.objects.in_bulk([])
{}

>>> Blog.objects.in_bulk()
{1: <Blog: Beatles Blog>, 2: <Blog: Cheddar Talk>, 3: <Blog: Django Weblog>}
```

in_bulk()에 빈 리스트를 넘기면, 빈 dict 를 얻게된다. 

###iterator()
`iterator()`

QuerySet을 실행하고 그 결과를 iterator해서 리턴한다. Queryset은 일반적으로 내부적으로 결과를 캐시하기 때문에 반복된 실행은 추가적인 쿼리는 발생하지 않지만 iterator()는 QuerySet 레벨에서 캐시하지 않고 데이터베이스에서 결과를 직접 읽는다. (내부적으로 기본 iterator는 iterator()를 호출하고 리턴한 값을 캐시한다.) 한 번만 접근하면 되는 많은 객체를 리턴한 QuerySet의 경우 성능이 향상되고 메모리가 크게 감소할 수 있다. 

이미 실행된 QuerySet에서 iterator()를 사용하는 것은 쿼리를 반복해서 다시 실행하는 것이 강제된다. 

또한 iterator()를 사용하면 이전의 prefetch_related()호출은 무시된다. 두 개의 최적화가 함께 할 수 없기 때문이다.

###latest()
`latest(field_name=None)`

date field로 제공된 field_name을 사용하면 테이블의 최신 객체를 날짜순으로 리턴한다.

아래의 예제는 pub_date 필드에 따라 테이블의 최신 entry를 리턴한다. 

```python
Entry.obejcts.latest('pub_date')
```

모델의 Meta가 get_latest_by를 지정한 경우, field_name argument를 earliest() 또는 latest()로 두면 된다. 

get()과 마찬가지로, 주어진 parameter의 객체가 없는 경우 earliest()와 latest()는 DoesNotExist를 발생시킨다. 

earliest()와 latest()는 편의와 가독성을 위해서 존재하는 것이다. 

###earliest()
`earliest(field_name=None)`

방향이 바뀌는 것을 제외하고 latest()와 동일하다.

###first()
`first()`

queryset에 의해 일치된 첫 번째 객체를 리턴한다. 일치하는 객체가 없는 경우 None을 리턴한다. QuerySet에 ordering이 지정되지 않았을 경우, queryset은 자동으로 primary key에 의해 정렬된다. 

```python
p = Article.objects.order_by('title', 'pub_date').first()
```

first()는 편리한 메서드이고, 위의 코드는 아래와 같다. 

```python
try:
	p = Article.objects.order_by('title', 'pub_date')[0]
except IndexError:
	p = None	
```
###last()
`last()`

마지막 객체를 리턴하는 것을 제외하고는 first()와 동일하다.

###aggregate()
`aggregate(*args, **kwargs)`

QuerySet으로 계산된 집계(평균, 총합 등)의 값을 dict로 리턴한다. aggregate()의 각 argument는 리턴되는 dict에 포함될 값을 지정한다. 

Django가 제공하는 집계함수는 [Aggregation Functions](https://docs.djangoproject.com/en/1.10/ref/models/querysets/#id5)에 설명되어있다.   
집계는 [query expressions](https://docs.djangoproject.com/en/1.10/ref/models/expressions/)이기 때문에 집계를 다른 집계 또는 값과 결합해 복잡한 집계를 만들 수 있다.

keyword argument를 이용해 지정된 aggregate는 해당 keyword를 annotation의 이름으로 사용한다. 익명 argument는 집계함수의 이름과 집계하고있는 모델 필드에 따라 생성된 이름을 갖는다. 복잡한 aggregate는 익명 argument를 사용할 수 없으므로 keyword argument를 alias로 지정해야한다. 


```python
>>> from django.db.models import Count
>>> q = Blog.objects.aggregate(Count('entry'))
{'entry__count': 16}
```
keyword argument를 이용해 집계함수를 지정하면, 리턴된 aggregation 값의 이름을 컨트롤 할 수 있다.

```python
>>> q = Blog.objects.aggregate(number_of_entries=Count('entry'))
{'number_of_entries': 16}
```

###exists()
`exist()`

QuerySet이 어떤 값이라도 가지고 있으면 True, 아니면 False를 리턴한다. 이것은 가장 간단하고 빠른 방법으로 쿼리를 수행하려고 하지만, 일반적인 QuerySet 쿼리와 거의 같은 쿼리를 실행한다.

exists()는 QuerySet의 객체 멤버쉽과 QuerySet의 객체의 존재와 관련된 검색에 유용한다. (특히 큰 QuerySet의 경우)

고유 필드(e.g. primary_key)가 있는 모델이 QuerySet의 멤버인지 여부를 찾는 가장 효율적인 방법은 아래와 같다. 

```python
entry = Entry.objects.get(pk=123)
if some_queryset.filter(pk=entry.pk).exits():
	print("Entry contained in queryset")
```
전체 queryset을 통해 실행하고 반복해야하는 아래의 코드보다 빠르다. 

```python
if entry in some_queryset:
	print("Entry contained in Queryset")
```

queryset이 아이템을 갖고있는지에 대해 찾으려면 다음과 같다.

```python
if some_queryset.exist():
	print("There is at least one object in some_queryset")
```
위의 코드는 아래보다 빠를 것이다. (하지만 큰 정도로 차이나는 것은 아님)

```python
if some_queryset:
	print("There is at least one object in some_queryset")
```

###update()
`update(**kwargs)`

지정된 필드의 SQL update query를 수행하고, 일치하는 row의 수를 리턴한다. (일부 row에 이미 새 값이 있는 경우, update된 row의 수가 다를 수 있다.)

2010년에 publish된 모든 blog entry의 comment 해제하려면 다음과 같은 방법을 사용할 수 있다. (예제에서는 Entry 모델에 pub_date와 comments_on필드가 있다고 가정한다.)

```python
>>> Entry.objects.filter(pub_date__year=2010).update(comments_on=False)
```
여러개의 필드(수에는 제한이 없음)를 update할 수도 있다. 아래의 예제에서는 comments_on과 headline필드를 update한다. 

```python
>>> Entry.objects.filter(pub_date__year=2010).update(comments_on=False, headline='This is old')
```
update()메서드는 즉시 적용되고, 업데이트되는 QuerySet에 대한 유일한 제약은 관련모델이 아닌, 모델의 메인 테이블에 있는 column만 업데이트 할 수 있다는 것이다. 아래의 예제와 같이 수행할 수는 없다.

```python 
>>> Entry.objects.update(blog__name='foo') # Won't work!
```

관련 필드를 기반으로 필터링하는 것은 가능하다.

```python
>>> Entry.objects.filter(blog__id=1).update(comments_on=True)
```

슬라이스를 가져오거나 더이상 필터링할 수 없는 QuerySet에 대해서는 update()를 호출할 수 없다.

update()메서드는 영향을 받은 row의 수를 리턴한다.

```python
>>> Entry.objects.filter(id=64).update(comments_on=True)
1

>>> Entry.objects.filter(slug='nonexistent-slug').update(comments_on=True)
0

>>> Entry.objects.filter(pub_date__year=2010).update(commnets_on=False)
132
```

모델 객체에 대해서는 아무것도 하지 않고, record만 업데이트 하고 싶은 경우에 가장 효율적인 방법은 메모리에 모델 객체를 로드하는 것이 아니라, update()를 호출해 접근하는 것이다. 

```python
e = Entry.objects.get(id=10)
e.comments_on = False
e.save()
```
위와 같이 수행하는 것보다 아래와 같이 수행하는 것이 효율적이다. 

```python
Entry.objects.filter(id=10).update(comments_on=False)
```

update()를 사용하면 경쟁조건을 방지할 수도 있다. (객체의 로드와 save()호출 사이의 짧은 기간 동안 데이터베이스에서 뭔가 변경될 수도 있음)

update()는 모델에서 save() 메서드를 호출하지 않고, SQL 레벨에서 업데이트를 한다. 따라서 pre_save 또는 post_save 신호(Model.save()를 호출한 결과)를 발생시키지 않는다. 커스텀 save()메서드를 가진 모델에 대해 record 집합을 업데이트하려면, 해당 record를 반복하고 save()를 호출하면된다.

```python
for e in Entry.objects.filter(pub_date__year=2010):
	e.comments_on = False
	e.save()
```

###delete()
`delete()`

Queryset의 모든 row에 대해 SQL delete query를 수행하고, 삭제된 객체의 수와 객체의 유형 당 삭제한 수가 있는 dict를 리턴한다. 

delete()는 즉시 적용된다. 슬라이스를 가져오거나 더이상 필터할 수 없는 QuerySet에 대해서는 delete()를 호출할 수 없다. 

```python
>>> b = Blog.objects.get(pk=1)

# Delete all the entries belonging to this Blog.
>>> Entry.objects.filter(blog=b).delte()
(4, {'weblog.Entry': 2, 'weblog.Entry_authors': 2})
```

기본적으로 Django의 ForeignKey는 SQL 제약 조건(ON DELETE CASCADE)을 실행한다. 이는 삭제할 객체를 가리키는 foreignkey가 있는 개체까지 함께 삭제한다는 것이다. 

```python
>>> blogs = Blog.objects.all()

# This will delete all blogs and all of ther Entry objects.
>>> blogs.delete()
(5, {'weblog.Blog': 1, 'weblog.Entry': 2, 'weblog.Entry_authors': 2})
```
이러한 cascade 동작은 ForeignKey의 on_delete argument에서 커스터마이징 할 수 있다.

delete()메서드는 대량으로 삭제를 수행하고 모델에서 delete()메서드를 호출하지 않는다. 하지만 모든 객체(cascaded 삭제 포함)에 대해 pre_delete및 post_delete 신호를 내보낸다.

Django는 객체를 메모리로 가져와 신호를 보내고, cascade를 처리해야한다. 그러나 cascade가 없고, 신호가 없다면 django는 짧은 경로를 취해 메모리로 가져오지 않고 객체를 삭제할 수 있다. 큰 삭제는 메모리의 사용량을 줄이고, 실행된 쿼리의 양도 줄일 수 있다. 

on_delete가 DO_NOTHING으로 설정된 ForeignKey는 짧은 경로로 삭제하는 것을 막지않는다. 


###as_manager()
`classmethod as_manager()`

QuerySet메서드의 복사본이 있는 Manager 인스턴스를 리턴하는 클래스 메서드이다. 자세한 내용은 [Creating a manger with QuerySet methods](https://docs.djangoproject.com/en/1.10/topics/db/managers/#create-manager-with-queryset-methods)를 참고.


-

##Field lookups
필드 조회는 SQL WHERE절의 meat를 지정하는 방법이다. QuerySet 메서드인 filter(), exclude(), get()에 대한 keyword argument로 지정된다. 

소개는 [models and database queries documentation](https://docs.djangoproject.com/en/1.10/topics/db/queries/#field-lookups-intro)를 참고.

Django에 내장된 lookup은 다음과 같다. 모델 필드에 대한 [커스텀 lookup](https://docs.djangoproject.com/en/1.10/howto/custom-lookups/)도 작성할 수 있다.

검색 유형이 제공되지 않은 경우, (Entry.objects.get(id=14)와 같은 경우) 조회 유형은 정확하다고 가정한다.

###exact
정확히 일치하는 것  
비교를 위해 제공된 값이 None이면 SQL NULL로 해석한다. (자세한 내용은 isnull참조)

```python
Entry.objects.get(id__exact=14)
Entry.objects.get(id__exact=None)
```
SQL로는 다음과 같다.

```SQL
SELECT ... WHERE id =14;
SELECT ... WHERE id IS NULL;
```

###iexact
대소문자를 구분하지 않는 정확한 일치.  
비교를 위해 제공된 값이 None 이면 SQL NULL로 해석한다. 

```python
Blog.objects.get(name__iexact='beatles blog')
Blog.objects.get(name__iexact=None)
```
SQL로는 다음과 같다.

```SQL
SELECT ... WHERE name ILIKE 'beatles blog';
SELECT ... WHERE name IS NULL;
```
첫 번째 쿼리는 'Beatles Blog', 'BeaTleS bLoG'등과 일치한다.

###contains
대소문자를 구분하고, 포함 여부를 테스트한다.

```python
Entry.objects.get(headline__contains='Lennon')
```
SQL로는 다음과 같다.

```SQL
SELECT ... WHERE headline LIKE '%Lennon%';
```
이것은 'Lennon honored today'와 일치하지만,  
'lennon honored today'와는 일치하지 않는다.

###iconains
대소문자를 구분하지 않고, 포함 여부를 테스트한다.

```python 
Entry.objects.get(headline__icontains='Lennon')
```
SQL로는 다음과 같다.

```SQL
SELECT ... WHERE headline ILIKE '%Lennon%'
```

<!--###in
###gt
###gte
###lt
###lte
###startswith
###istartswith
###endswith
###iendswith
###range
###date
###year
###month
###day
###week_day
###hour
###minute
###second
###isnull
###regex
###iregex

##Aggregation functions
###expression
###output_field
###**extra
###Avg
###Count
###Max
###Min
###StdDev
###Sum
###Variance

##Query-related tools
###Q() objects
###Prefetch() objects
###prefetch_related_objects()-->