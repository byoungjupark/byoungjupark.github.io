---
title: "TIL41. Pycharm 사용법과 SQLAlchemy"
excerpt: "환경변수 설정, black툴, SQLAlchemy의 orm과 session"
---
## GitHub repo

[https://github.com/byoungjupark/pure-python](https://github.com/byoungjupark/pure-python)

## 오늘 배운 것
### 1. Pycharm에서 환경변수 설정하기
vs code에서는  `.env` 파일에서 환경변수 설정을 했는데, pycharm에서는 파일 생성이 되지 않았다. 
대신 환경설정 부분에서 변수를 세팅할 수 있다.
run, debug 등을 실행하는 상태바에서  `Edit Configurations` 에 들어간다.
<div><img width="400" alt="환경변수1" src="https://user-images.githubusercontent.com/63541271/142877288-144a0ee3-9aa3-4ffe-9b9c-6b4181082e0b.png"></div>

`Environment variables` 에 들어가면 프로젝트와 관련된 환경변수를 설정할 수 있다.
<div><img width="696" alt="환경변수2" src="https://user-images.githubusercontent.com/63541271/142877330-47c16b74-252e-45fa-af23-3e61d5fa1f0b.png"></div>

데이터베이스와 관련된 정보, 토큰과 관련된 정보는 아래와 같이 key value를 설정하여 노출시키지 않도록 한다.
<div><img width="584" alt="환경변수3" src="https://user-images.githubusercontent.com/63541271/142877345-4259e4ed-2836-4c0e-950e-189faaffe73b.png"></div>


### 2. black으로 포맷팅하는 법

black은 파이썬 코드 자동 포맷팅 도구
pep8이라는 파이썬 코드 작성 스타일가이드 일종이 있는데, black을 사용하면 자동으로 이 스타일을 적용시켜준다.
black [파일이름] 명령어를 입력하면 포맷팅이 가능하지만, 매번 하기에는 번거롭다.
pycharm에서는 `Prefrences` - `Tools` - `External Tools` 에 black을 추가하면, 자동으로 코드스타일이 적용된다.
<img width="976" alt="black1" src="https://user-images.githubusercontent.com/63541271/142877577-2051e766-48ff-4e40-b8df-9f4343aa71d7.png">

- Program : 실행파일 (black이 설치된 경로를 적어준다)
- Argument : 현재 파일이름
- Working directory : 프로젝트 디렉토리

`File Watchers` 툴로 추가하게 되면 브랜치를 변경할 때 파일 변경으로 인식되어 불필요하게 자주 실행될 경우가 있다. 
`External Tools` 에 추가하여 단축키를 설정하면 원하는 파일이나 디렉토리를 눌러 코드를 포맷팅할 수 있다. 
`Preferences` - `Keymap` 에서 단축키로 설정해줄수도 있다.

<img width="556" alt="black2" src="https://user-images.githubusercontent.com/63541271/142877600-63daffcb-0282-4aa2-8eb5-82a989a3978b.png">


### 3. SQLAlchemy (db연결, orm, session)
1) 데이터베이스 생성 및 연결하기
파이썬과 데이터베이스를 연결시키려면 mysqlclient라는 데이터베이스 커넥터로 연결시킬 수 있다.

#### 데이터베이스 커넥터
- mysqlclient : C로 구현됨, 속도 더 빠름
- pymysql : 순수 파이썬으로 구현됨
    

우선 MySQL에서 데이터베이스를 만든다.
SQLAlchemy는 데이터베이스와 연결시켜 ORM을 해주는 파이썬 패키지이다. 
SQLAlchemy의 session객체는 ORM으로 매핑된 객체들을 지속적으로 관리해주는 역할을 한다. 
ORM 객체 자체는 세션 내에서 유지된다.
<br>

코드들이 데이터베이스에 바로 보내지는 것이 아닌 세션객체를 거치게 되는데, 커밋을 해야 세션에서 코드 작업을 인식하고 데이터베이스에 보낼 수 있다. 

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base

# 데이터베이스 주소로 연결하기
engine = create_engine(
            f"mysql+mysqldb://{DB['user']}:{DB['password']}@{DB['host']}:{DB['port']}/{DB['name']}?charset=utf8"
        )
# 세션객체와 인스턴스 만들기
Session = sessionmaker(engine)
session = Session()
```

2) 테이블 매핑하기

sqlalchemy declarative_base()로 테이블을 생성하고 연결시킬 수 있다. 
Base 객체를 상속받아 사용할 테이블들을 만들어준다. 
sqlalchemy의 Column으로 각 필드를 생성하고, 내장된 타입클래스를 불러와 설정하면 된다.
<br>

String은 최대길이를 지정해주도록 한다.

```python
# database.py
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```

```python
# models.py
from sqlalchemy import Column, Integer, String
from database import Base

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    email = Column(String(30))
    password = Column(String(200))
		
		# 실제 값이 들어가서 저장될 변수 선언하기
    def __init__(self, email, password):
        self.email = email
        self.password = password
```

### 4. GitHub계정 바꿔서 올리기 (feat.깃헙토큰 변경)

개인계정으로 push를 할 때 아래와 같은 에러가 발생했다.
깃헙이 회사계정으로 연결되어 있기 때문에, 회사계정 토큰과 개인계정 토큰이 일치하지 않는 것이 원인

```python
git push
remote: Permission to byoungjupark/byoungjupark.github.io.git denied to sihong12.
fatal: unable to access 'https://github.com/byoungjupark/pure-python.github.io.git/': The requested URL returned error: 403
```

다른 깃헙계정으로 커밋을 올리고 싶은 경우, 원래 계정으로 연결된 깃헙토큰을 변경하거나 삭제하고, 올리고 싶은 계정의 토큰을 추가하여야 한다.
git config —list 로 어떤 유저가 등록되어 있는지 확인할 수 있다.
mac은 키체인 시스템으로 인증정보를 저장한다.
git credential-osxkeychain erase로 토큰이 저장된 키체인을 삭제해준다.
이후 다시 push를 할 때 user.name과 password를 입력하라는 내용이 뜨고, 원하는 계정 정보를 입력하면 그 계정으로 커밋을 올릴 수 있다.

