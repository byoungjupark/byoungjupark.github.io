---
title: "TIL27. RESTful API"
excerpt: "REST, REST API, RESTful API 각각의 개념"
---

### 1. REST 란
Representational State Transfer
자원을 표현하고 자원의 상태를 전달하는 것을 의미한다.
* 자원 Resource : 자원에는 문서, 이미지, 데이터 등이 있으며, 서버에 존재한다.
* 표현 Representation : 자원을 표현하는 것
ex) DB의 유저 정보가 자원일 때 'users'로 자원을 표현할 수 있다.

자원을 표현하는 방식으로 URI(Uniform Resource Identifier)를 사용한다. 
클라이언트가 자원의 상태(정보)에 대한 조작을 요청하면 서버는 적절한 응답을 보낸다.


### 2. REST API
* API Application Programming Interface
인간과 기계 사이의 소통을 이어주는 역할
ex) 인간-TV 는 리모컨으로 상호작용할 수 있다
인간-컴퓨터는 모니터, 키보드, 마우스로 상호작용할 수 있다

* 웹에서의 API
웹 클라이언트와 웹 서버는 API를 사용하여 교류할 수 있고, 그 방식 중 하나가 REST API이며, REST는 HTTP를 기반으로 구현되었다.

* REST API 작성 규칙
	1. 자원은 명사, 소문자를 사용한다
	2. 디렉토리는 복수 명사를 사용한다
	3. URI 마지막 부분은 슬래시(/)를 포함하지 않는다
	4. 하이픈(-)은 ok, 밑줄(_)은 no

### 3. RESTful API
REST를 REST 답게 사용하는 것을 RESTful이라고 표현한다.
이해하기 쉽고 사용하기 쉬운 REST API를 만들어 제공하는 웹서비스를 'RESTful' 하다고 표현할 수 있다.
