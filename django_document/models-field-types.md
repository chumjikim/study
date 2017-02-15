#Field types

index

- Field types
	- AutoField
	- BigAutoField
	- BigIntegerField
	- BinaryField
	- BooleanField
	- CharField
	- CommaSeparatedIntegerField
	- DateField
	- DateTimeField
	- DecimalField
	- DurationField
	- EmailField
	- FileField
		- FileFiled and Filedfile
	- FilePathField
	- FloatField
	- ImageField
	- IntegerField
	- GenericIPAddressField
	- NullBooleanField
	- PositiveIntegerField
	- PositiveSmallIntegerField
	- SlugField
	- SmallIntegerField
	- TextField
	- TimeField
	- URLField
	- UUIDField

>[Model field reference 문서](https://docs.djangoproject.com/en/1.10/ref/models/fields/)



##AutoField
`class AutoField(options)`  

사용가능한 ID에 따라 자동으로 증가하는 IntegerField이다. 보통은 직접적으로 사용하지 않는다. 별다른 지정을 하지 않으면 primary key field가 모델에 자동으로 추가된다. 

```python
# Automatic primary key fields
# 기본적으로 django각 모델에 다음과 같은 field를 구성한다.
id = models.AutoField(primary_key=True)
```

##BigAutoField
`class BigAutoField(**options)`

64비트 정수형 필드.   

1에서  9223372036854775807까지의 숫자와 맞도록 보장된다는 것을 제외하고 AutoField와 유사하다.

##BinaryField
`class BinaryField(**options)`

바이너리 데이터를 저장하는 필드이다. bytes 할당만 지원한다.   

이 필드는 기능제한이 있는데, BinaryField 값에 queryset을 필터할 수 없고, Model Form에 BianryField 를 포함시킬 수 없다. 

>Abusing BinaryField  
>데이터베이스에 파일을 저장하는 것에 대해 생각할 수 있지만, 그 경우는 거의 잘못 설계된 것이라고 생각하면 된다. 이 필드는 적절한 정적파일 처리를 대체하지 않는다. (?)

##BooleanField
`class BooleanField(**options)`

True/False 필드.  

이 필드의 기본 위젯은 CheckboxInput이다.     

null값을 허용하려면 NullBooleanField를 사용한다.  

Field.default가 정의되지 않았다면 BooleanField의 기본값은 None이다. 

##NullBooleanField
`class NullBooleanField(**options)`

BooleanField와 유사하지만, Null값을 허용한다. BooleanField에서 null=True를 사용하는 대신 NullBooleanField를 사용한다.   

이 필드의 form 위젯의 기본 형태는 NullBooleanSelect이다. 


##CharField
`class CharField(max_length=None, **options)`

작은것 부터 큰 사이즈의 문자열을 나타내는 문자열 field이다. 이 field의 기본 위젯 형식은 TextInput이다.  
많은 양의 텍스트라면 TextField를 사용해라.

CharField는 추가적인 인자를 받는다.   

__CharField.max_length__  
field의 문자의 최대 길이다. max_length는 데이터베이스 레벨과 django의 validation에 강제된다. 


##CommaSeparatedIntegerField
`class CommaSeperatedIntegerField(max_length=None, **options)`

comma로 분리된 정수형 필드이다. CharField처럼 max_length argument가 요구된다. 

```
DEPRECATED SINCE VERSION 1.9
  
이 필드는 1.9 버전 이후로는 사용하지 않고, 대신 CharField를 사용한다.  
```

##DateField
`class Datefield(auto\_now\_add=False, **options)`

###Datefield.auto\_now
object가 __저장될 때 마다__, field를 자동으로 설정한다.  
`'last-modified'`같은 timestamps에 유용하다.  
항상 현재 날짜를 사용한다.  
재정의할 수 있는 값이 아니다. 

###DateField.auto\_now\_add
object가 __처음 생성될 때__, field를 자동으로 설정한다.  
timestamp 생성에 유용하다.  
항상 현재의 날짜를 사용한다.  
재정의할 수 있는 값이 아니다.  
그렇기 때문에 object를 생성할 때 이 field값을 설정하려고 해도 무시된다. 이 field를 수정하고 싶다면 `auto_now_add=True`대신 다음의 설정을 따라야한다. 

- DateField: default=date.today - `datetime.date.today()`
- DateTimeField: default=timezone.now - `django.utils.timezone.now()` 

> __Note__  
> 
>auto_now 혹은 auto\_now\_add를 True로 설정하면 해당 field는 **editable=False**, **blank=True**로 설정된다.
> 
>auto\_now 혹은 auto\_now\_add 옵션은 생성 또는 업데이트 할 때의 기본시간대 날짜를 사용한다. 다른 값을 원한다면 auto_now 혹은 auto\_now\_add를 사용하는 것 대신, 자신만의 호출 가능한 기본값을 사용하거나 save()를 무시할 수 있다. 또는 DateField 대신 DateTimefield를 사용하여 표시 시간에 datetime에서 date로 변환해 처리할 수 있다. 

##DateTimeField
`class DateTimeField(auto\_now=False, auto\_now\_add=False, **options)`

python에서 datetime.datetime 인스턴스로 표현되는 날짜와 시간이다. DateField와 동일한 arguments를 갖는다. 

이 필드의 form 위젯의 기본 형태는 단일 TextInput이다. admin은 JavaScript shortcuts으로 두개의 분리된 TextInput을 사용한다. 

##DecimalField
`class Decimal Field(max_digits=None, decimal_places=None, **options)`

고정 소수점 이하의 십진수로, python에서 Decimal 인스턴스로 표현한다. 두 가지의 필수적인 argument가 있다. 

**DecimalField.max_digits**
숫자에 허용되는 최대 자릿수이다. 이 수는 decimal_palces보다 크거나 같아야한다.

**DecimalField.decimal_places**
숫자와 함께 저장할 소수 자리수이다.

```python
models.DecimalField(..., max_digits=5, dicimal_places=2)
```

```python
models.DecimalField(..., max_digits=19, decimal_places=10)
```

이 필드의 form 위젯의 기본 형태는 localize가 False일 경우 NumberInput이고, 그렇지 않으면 TextInput이다.

> [FloatField vs. DecimalField](https://docs.djangoproject.com/en/1.10/ref/models/fields/#floatfield-vs-decimalfield)

##FloatField
`class FloatField(**options)`

부동소수점 숫자로, python에서 float인스턴스로 표현한다. 

이 필드의 form 위젯의 기본 형태는 localize가 False인 경우 NumberInput이고, 그렇지 않으면 TextInput이다. 

##DurationField
`class DurationField(**options)`

python으로 모델링한  timedelta를 이용해 시간 간격을 저장하는 필드이다. PostgreSQL에서 사용하면 데이터타입은 `interval`이고, Oracle에서는 `INTERVAL DAY(9) TO SECOND(6)`이 된다. 그 이외에는 microseconds를 사용해 `bigint`가 된다. 

>PostgreSQL 이외의 모든 데이터베이스에서 DurationField값을 DateTimeField의 인스턴스와 비교하는 것은 예상대로 동작하지 않을 것이다.

##EmailField
`class EmailField(max_length=254, **options)`

값이 유효한 이메일 주소인지 체크하는 CharField이다. `EmailValidator`를 이용해 입력받은 값이 유효한 것인지 확인한다.


##FileField
`class FileField(upload_to=None, max_length=100, **options)`

파일 업로드 필드이다. 

> primary_key와 unique argument가 지원되지 않기 때문에 이를 사용한다면 TypeError가 난다.

두 가지 선택적 argument가 있다.

####FileField.upload_to
이 속성은 업로드 디렉토리와 파일이름을 설정하는 두 가지 방법을 제공한다. 두 가지 방법 모두 값이 Storage.save()메서드로 값이 전달된다.

만약 문자열 값을 지정하면 [strftime()](https://docs.python.org/3/library/time.html#time.strftime)포맷이 포함될 수 있다. 이 경우 파일 업로드의 날짜와 시간으로 대체될 것이다. (업로드 된 파일이 주어진 디렉토리를 채우지 않도록 한다.)

```python
class MyModel(models.Model):
	#file will be uploaded to MEDIA_ROOT/uploads
	upload = models.FileField(upload_to='uploads/')
	# or ...
	# file will be saved to MEDIA_ROOT/uploads/2015/01/30
	upload = models.FileField(upload_to='uploads/%Y/%m/%d/')
```
기본 FileSystemStorage를 사용한다면, 업로드된 파일이 저장될 로컬 파일 시스템의 위치를 구성하기 위해 문자열 값이 MEDIA_ROOT경로에 추가될 것이다. 만약 다른 스토리지를 사용한다면 해당 스토리지의 upload_to를 핸들링하는 문서를 확인해봐야한다. 

upload_to는 함수처럼 호출이 가능할 수도 있다. 파일이름을 포함한 업로드 경로를 얻기위해 호출한다. 이 호출은 두 가지 필수 argument를 받고, 스토리지 시스템에 전달되는 Unix-style(슬래시로 표현) path로 리턴한다. 

| Argument | Description |
| --- | --- |
**instance** | FileField가 정의된 모델의 인스턴스이다. 구체적으로는, 현재 파일이 첨부되는 특정한 인스턴스이다. 대부분의 경우, 이 객체는 아직 데이터베이스에 저장되지 않은 상태이기 때문에 default AutoField를 사용하면 primary key필드에 아직 값이 없을 수도 있다. 
**filename** | 원래 파일에 주어진 파일이름이다. 최종 목적지 경로를 결정할 때, 고려될 수도 있고 아닐 수도 있다. 


```python
def user_directory_path(instance, filename):
	#file will be uploaded to MEDIA_ROOT/user_<id>/<filename>
	return 'user_{0}/{1}'.format(instance.user.id, filename)
	
class MyModel(models.Modle):
	upload = models.FileField(upload_to=user_directory_path)
```

####FileField.storage
파일 저장 및 검색을 처리하는 storage 객체이다. 이 객체를 제공하는 자세한 방법은 [Managing files](https://docs.djangoproject.com/en/1.10/topics/files/)를 참고.

이 필드의 form 위젯의 기본 형태는 ClearableFileInput이다. 

모델에 FileField나 ImageField를 사용한다면 여기에는 몇가지 단계가 있다. 

1. 파일 설정에서, Django가 업로드된 파일을 저장할 디렉토리의 전체 경로로  MEDIA_ROOT를 정의해야한다. (성능을 위해 이 파일들은 데이터베이스에 저장되지 않는다.) 해당 디렉토리의 base public URL로 MEDIA_URL을 정의한다. 해당 디렉토리가 웹 서버 유저의 계정으로 쓸 수 있는지 확인해야한다.
2. 모델에 FileField나 ImageField를 추가한다. 업로드된 파일을 사용하기 위해  MEDIA_ROOT의 서브디렉토리를 `upload_to`옵션에 지정한다.
3. 데이터베이스에 저장되는 모든것은 파일의 경로(MEDIA_ROOT를 기준으로 함)이다. Django의 `url`속성을 사용할 수 도 있다. 예를들어, ImageField가 mug_shot이라고 할 때, `{{ object.mug_shot.url }}`템플릿을 이용해 이미지의 절대경로를 얻을 수 있다.

예를 들어, MEDIA_ROOT가 `'/home/media`이고 upload_to가 `phots/%Y/%m/%d`로 설정되었다고  가정해보자.  
만약 Jan.15.2007에 파일을 업로드했다면, `/home/media/photos/2007/01/15`디렉토리에 저장될 것이다.
>upload_to의 %Y/%m/%d는 strftime()포맷이다. (%Y: 네 자리로 표현되는 `년`도, %m: 두 자리로 표현되는 `월`, %d: 두 자리로 표현되는 `일`)  

업로드된 파일의 디스크상의 파일이름이나 파일 사이즈를 검색하고 싶다면, 각각 `name`과 `size` 속성을 사용할 수 있다. 더 많은 속성과 메서드에 대한 정보는 [file](https://docs.djangoproject.com/en/1.10/ref/files/file/#django.core.files.File)클래스 레퍼런스와 [Managing files](https://docs.djangoproject.com/en/1.10/topics/files/)가이드를 참고.

> 파일은 모델을 저장하는 과정의 일부로 데이터베이스에 저장되기 때문에, 모델이 저장되기 전에는 디스크에 사용된 실제 파일이름을 신뢰할 수 없다.

`url`속성을 이용해 업로드된 파일의 상대 URL을 얻을 수 있다. 내부적으로 이것은 Storage클래스의 url()메서드를 호출한다.

업로드된 파일을 다룰 때는, 보안의 허점을 피하기 위해 파일을 업로드하는 위치와 파일 타입에 주의를 기울여야한다. 업로드된 모든 파일은 유효성 검증을 거쳐야한다.   

예를들어, 웹서버의 document root의 디렉토리에 누군가 유효성 검증을 거치지 않은 파일을 업로드하면, 누군가가 CGI나 PHP 스크립트를 업로드하고 사이트의 URL에 방문해 해당 스크립트를 실행할 수 있다. 이런 것을 허용하면 안된다. 

또한 업로드된 HTML파일은 브라우저(서버가 아니더라도)가 실행할 수 있기 때문에 XSS또는 CSRF공격과 유사한 보안상의 위협이 있다 .

FileField 인스턴스는 최대 길이가 100인 varchar 컬럼으로 데이터베이스에 생성된다. 다른 필드처럼 max_length argument를 이용해 변경할 수 있다. 

###FileField and FieldFile
`class FieldFile`

모델의 FileField에 접근할 때, FieldFile의 인스턴스는 파일에 접근하기 위한 proxy로 제공된다.

FieldFile의 API는 하나의 중요한 차이를 제외하면 File과 같다. : 클래스에 의해 래핑된 객체가 반드시 python 내장파일 객체에 대한 래퍼일 필요는 없다. 대신 Storage.open()메서드의 결과에 대한 래퍼이다. 이 메서드는 File객체 일 수도 있고, File API의 커스텀 storage의 구현일 수도 있다.
>?ㅎㅎ 무슨말이지..

File에서 상속받은 read(), write()와 같은 API외에도 FieldFile은 파일과의 상호작용에 사용할 수 있는 몇가지 메서드를 포함한다.

__WARNING__  
이 클래스의 save()와 delete()메서드는 
데이터베이스에 관련된 FieldFile의 모델 객체를 저장하는 기본값이다.

**FieldFile.name**  
관련된 FileField의 Storage의 root에서 상대경로를 포함한 파일이름

**FieldFile.size**  
Storage.size()메서드의 결과

**FieldFile.url**  
Storage 클래스의 url()메서드를 호출해서 파일의 상대경로로 접근하기 위한 읽기 전용 속성이다.

**FieldFile.open(mode='rb')**  
지정된 모드로 인스턴스에 관련된 파일을 열거나, 다시 연다. python의 표준 open()메서드와는 달리, 파일 descriptor를 리턴하지 않는다.

기본 파일은 접근시 열리기 때문에, 모드 변경을 위한 것이 아니라면 이 메서드를 호출할 필요가 없다. 

**FieldFile.close()**  
Python의 표준 file.close()메서드와 유사하게 동작하고 해당 인스턴스에 관련된 파일을 닫는다.

**FieldFile.save(name, content, save=True)**
이 메서드는 파일이름과 파일내용을 가져와 필드의 storage클래스에 전달한다. 그리고 저장된 파일을 모델 필드와 연결한다. 모델의 FileField 인스턴스에 파일 데이터를 수동으로 연결하려면 save()메서드를 사용해 해당 파일 데이터를 유지한다.

두 가지 argument가 필요하다.   
**name**: 파일 이름   
**content**: 파일의 내용을 담고있는 객체

save argument(optional)는 해당 필드와 연관된 파일이 변경된 후, 모델 인스턴스의 저장 여부를 제어한다. 기본은 True임.

content argument는 python의 내장 파일 객체가 아닌 `django.core.files.File`의 인스턴스여야한다. 아래와 같이 기존 python 파일 객체에서 File을 구성할 수 있다.

```python
from django.core.files import File
#Open an existing file using Python's built-in open()
f = open('/path/to/hello.world)
myfile = File(f)
``` 

```python
from django.core.files.base import ContentFile
myfile = ContentFile("hello world")
```

더 많은 정보는 [Managing files](https://docs.djangoproject.com/en/1.10/topics/files/)로...

**FieldFile.delete(save=True)**  
해당 인스턴스에 관련된 파일을 지우고 해당 필드의 모든 속성을 clear한다. 참고: 이 메서드는 delete()를 호출했을 때, 파일이 열려있으면 파일을 닫는다.

save argument(optional)는 해당 필드와 연관된 파일이 삭제된 후, 모델 인스턴스의 저장 여부를 제어한다. 기본은 True임.

모델이 삭제될 때, 관련 파일은 삭제되지 않는다. 어디에도 관계가 없는 파일을 지우려면, 직접 지워야한다. 


##FilePathField
`class FilePathField(path=None, match=None, recursive=False, max_length=100, **options)`

A CharField whose choices are limited to the filenames in a certain directory on the filesystem. Has three special arguments, of which the first is required:
 
CharField는 파일 시스템의 특정 디렉토리에 있는 파일 이름으로 제한된다.
세 가지 특수한 argument를 갖는다. 첫 번째는 필수사항임.

**FilePathField.path**  

**FilePathField.match**  

**FilePathField.recursive**  

**FilePathField.allow_files**  

**FilePathField.allow_folders**  

##ImageField
`class ImageField(upload_to=None, height_field=None, width_field=None, max_length=100, **options)`

FileField의 모든 속성과 메서드를 상속받지만, 업로드된 객체가 유효한 이미지인지 유효성검증을 한다. 

FileField에 사용할 수 있는 특수한 속성 이외에도 ImageField는 height와 width속성을 갖는다. 

해당 속성들의 query를 위해 ImageField는 두 가지 선택적인 argument를 갖는다.

**ImageField.height_field**  
모델 인스턴스가 저장될 때 마다 이미지의 height로 자동적으로 채워지는 모델 필드의 이름이다. 

**ImageField.width_field**  
모델 인스턴스가 저장될 때 마다 이미지의 width로 자동적으로 채워지는 모델 필드의 이름이다. 

[Pillow](https://pillow.readthedocs.io/en/latest/)라이브러리가 필요함

ImageField 인스턴스는 최대 길이가 100인 varchar 컬럼으로 데이터베이스에 생성된다. 다른 필드처럼 max_length argument를 이용해 변경할 수 있다. 

이 필드의 form 위젯의 기본 형태는 ClearableFileInput이다.

##IntegerField
`class IntegerField(**options)`

32비트 정수형 필드.  

django가 지원하는 모든 데이터베이스에서 -2147483648 부터 2147483647까지의 값을 허용한다. 

이 필드의 form 위젯의 기본 형태는 localize가 False이면 NumberInput이고, 그렇지 않으면 TextInput이다. 

>정수 사이즈에 따라 BigIntegerField, SmallIntegerField를 사용할 수 있다. 

##BigIntegerField
`class BigIntegerField(**options)`

64bit 정수형 필드.   

-9223372036854775808 에서 9223372036854775807까지의 숫자와 맞도록 보장된다는 것을 제외하고 IntegerField와 유사하다.  

이 필드의 form 위젯의 기본형태는 TextInput이다. 

##SmallIntegerField
`class SmallIntegerField(**options)`

IntegerField와 유사하지만, 특정 지점(데이터베이스에 종속적이다)에서만 값을 허용한다. 

django가 지원하는 모든 데이터베이스에서 -32768 에서 32767 까지의 값이 허용된다. 

##PositiveIntegerField
`calss PositiveIntegerField(**options)`

IntergerField와 유사하지만, 양수이거나 0이어야한다. django가 지원하는 모든 데이터베이스에서 0 부터 2147483647 까지의 값이 허용된다. 이전 버전과의 호환성을 위해 0이 허용된다. 

##PositiveSamllIntegerField
`class PositiveSmallIntegerField(**options)`

PositiveIntegerField와 유사하지만, 특정 지점(데이터베이스에 종속적임)에서만 값을 허용한다. django가 지원하는 모든 데이터베이스에서 0 부터 32767까지의 값이 허용된다.


##GenericIPAddressField
`class GenericIPAddressField(protocol='both', unpack_ipv4=False, **options)`

문자열 포맷의 IPv4 또는 IPv6 주소이다. (e.g. 192.0.2.30 or 2a02:42fe::4)  

이 필드의 form 위젯의 기본 형태는 TextInput이다.  

IPv6 주소 정규화는 [RFC 4291#section-2.2](https://tools.ietf.org/html/rfc4291.html#section-2.2)를 따르고, 이 섹션의 3번 단락에서 제안한 IPv4 포맷(예시로 ::ffff:192.0.2.0와 같은 포맷)을 포함한다.  

예를 들어,   
2001:0::0:01 을 정규화하면  `2001::1`    
::ffff:0a0a:0a0a 을  정규화하면 `::ffff:10.10.10.10`


##SlugField
`class SlugField(max_length=50, **options)`

`Slug`는 신문 용어이다. 슬러그는 문자, 숫자, 밑줄(_), 하이픈(-)만 포함하는 짧은 라벨이다. 일반적으로 URL에 사용된다. 

CharField와 마찬가지로, max_length를 지정할 수 있다. 만약 max_length를 지정하지 않았다면 django는 default 길이인 50을 사용할 것이다. 

Field.db_index를 True로 설정한다. 

다른 값을 기반으로 SlugField를 자동으로 채우는 것은 유용한 방법이다. `prepopulated_fields`를 사용하면 admin에서 자동적으로 처리할 수 있다. 

__SlugField.allow_unicode__  
True라면 해당 필드는 ASCII 문자 외의 Unicode 문자도 사용할 수 있다. default는  False임.

##TextField
`class TextField(**options)`

큰 text field이다.

이 필드의 form 위젯의 기본 형태는 Textarea이다.

만약 max_legth속성을 지정하면, 자동으로 생성된 form인 Textarea 위젯에 반영된다. 그러나 데이터베이스 레벨이나 모델에는 강제되지않는다. 이를 원한다면 CharField를 사용하면 된다. 


##TimeField
`class TimeField(auto_now=False, auto_now_add=False, **options)`

Python에서 datetime.time 인스턴스로 표현되는 시간이다. DateField와 마찬가지로 auto-population(자동 채우기) 옵션을 적용한다. 

이 필드의 form 위젯의 기본형태는 TextInput이다.    
admin은 몇 가지의 JavaScript shortcuts을 추가한다.

##URLField
`class URLField(max_length=200, **options)`

URL을 위한 CharField이다.

이 필드의 form 위젯의 기본형태는 TextInput이다.

모든 CharField의 서브클래스처럼, URLField는 max_length argument를 선택적으로 갖는다. 만약 max_length를 지정하지 않으면 default인  `max_length=200`이 된다. 

##UUIDField
`class UUIDField(**options)`

범용적으로 고유한 식별자를 저장하기 위한 필드이다. python의 UUID클래스를 사용한다.(코드 위에서 import 하고 사용함) PostgreSQL에서 사용하면 uuid데이터 타입에 저장되고, 그렇지 않으면 Char(32)에 저장된다. 

범용적으로 고유한 식별자는 primary_key에 대한AutoField를 대신할 수 있는 좋은 방법이다. 데이터베이스에서 UUID를 생성하지 않기때문에 dafualt를 사용하는 것이 좋다.(?: 이걸 의미하는 건가.. default = uuid.uuid4)

```python
import uuid
from django.db import models

class Person(models.Model):
	id = modles.UUIDField(primary_key=True,
	default = uuid.uuid4, editable=False)
	# other fields
```
UUID 인스턴스가 아닌, 호출가능한 객체가 기본값으로 전달된다.

