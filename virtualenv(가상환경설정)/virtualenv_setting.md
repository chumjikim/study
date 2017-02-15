#pip freeze requirements.txt 사용하기
__reauirements.txt__에 현재 패키지들의 버전을 담아놓고,
레파지토리에 올릴 때, 이걸 같이 올려주면 어떤 환경에서 해당 코드가 돌아가는지 알려줄 수 있음.


##reauirements.txt를 작성해서 github에 올리기

새로운 가상환경을 만든다

```python
pyenv virtualenv 3.4.3 pip_text_env # pip_text_env = 가상환경이름
```

현재 디렉토리에 pip_text_env가상환경 지정해줌 

```python
# 가상환경을 사용할 디렉토리로 이동 후 가상환경을 로컬에 지정해줌
pyenv local pip_test_env
```
해당 디렉토리에 git 생성

```python
git init

vi .gitignore # 필요한 ignore 정보 쓰고 저장
```

`l` : .git폴더, .gitignore파일, .python-version 파일이 생성된걸 확인할 수 있음


>`.python-version`파일이 생성된 곳에 가상환경이 적용된다.

패키지 두 개 추가해준다.

```python
pip install requests
pip install beautifulsoup4
```

Pycharm에서 해당 디렉토리를 열고 pip_test.py python파일 하나 만들어준다.

```python
py . # aliase로 py 는 pycharm을 여는 커맨드로 설정이 되어있는 상태임
```
Pycharm에서   
preference > project:pip_test > project interpreter 들어가서 
project interpreter를 바꿔준다.

show all > +  새로 추가해줌 

`/usr/local/var/pyenv/versions/3.4.3/envs/pip_test_env/bin/python`
경로로 지정해준다.

>우분투  
>/home/<유저명>/.pyenv/versions/3.4.3/env/<가상환경이름>/vin/python    
>
>macOS
>/usr/loca/var/pyenv/versions/3.4.3/env/<가상환경이름>/vin/python


`git status` 해보면 다음과 같이 나온다.

```
.gitignore
.idea/
pip_test.py
```

.gitignore 와 .idea/를 add하고, 커밋메세지 작성하여 commit

```python
git add .gitignore .idea/

git commit -m 'project setting'
```

pip_test.py 도 add하고 commit

```python
git add .pip_test.py

git commit -m 'add pip test'
```

깃헙에서 새로운 레파지토리를 생성하고, 주소를 가져와 연결한다.

```python
git remote add origin https://github.com/anohk/pip_test.git  

git push -u origin master
```

####pip freeze로 requirements텍스트 파일을 생성한다.

```python
pip freeze > requirements.txt
```
freeze 가 txt파일로 생김

>`>` 덮어쓰기  
`>>` 붙여넣기

해당파일을 add하고 커밋

```python
git add requirements.txt

git commit -m 'add requirements'
```
push하면 레파지토리에 파일들이 올라온걸 볼 수 있음.
 
##가상환경이 적용되지 않은 상태에서 파일을 받아 써보기

project디렉토리에서 새 폴더를 만들고, pip_test레파지토리를 클론해본다.

```python
git clone https://github.com/anohk/pip_test.git
```
pwd: /Users/<사용자명>/project/sample  
pip test로 들어간다.  
pwd: /Users/<사용자명>/project/sample/pip_test 로 이동

```python
cd pip_test 
```

```python
pyenv virtualenv 3.4.3 cloned_project

pyenv local cloned_project
```
python pip_test.py하면 원본 파일의 가상환경 적용이 안되있기 때문에 작동하지 않는다.

```python
pip install -r requirements.txt
```
를 하면 requirements에 있는 가상환경이 적용된다.

그리고 다시 python pip_test.py해보면 작동함!


>python 버전 정보는 README.md에 써준다.


##가상환경 지우기 

가상환경이 설정된 디렉토리에 들어가서

```python
rm .python-version

pyenv uninstall <가상환경이름>
```
가상환경 날아감

 ---
 .gitignore를 추가하지 않고 사용하던 레파지토리는, 커밋을 모두 지우고 다시 추가해야한다.
이미 커밋한 것은 .gitignore의 영향을 받지 않기 때문.

rm -rf .git
으로 깃을 날려버리고, (파일은 존재함)

git init  으로 깃 폴더를 다시 만든다.
 
그리고 커밋을 다시 한도 ㅏ ...
리모트도 다시추가.

커밋 두개를 하나로 수정하고 싶다면  
git reset HEAD~1

git push origin master -force
강제적으로 덮어쓰기함.

__*push 후 커밋 내용을 바꾸면 안된다.*__

---



