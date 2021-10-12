---
title : "TIL31. REST API와 DRF 개념"
excerpt : "API와 REST의 개념, Django와 DRF의 차이점"
---
# 1. API, REST
#### API (Application Programming Interface)
API란 서로 다른 소프트웨어 간의 커뮤니케이션 방법을 말한다.

#### Architecture without API
<div><img width="437" alt="without_API" src="https://user-images.githubusercontent.com/63541271/136952240-eb365708-a37c-4d70-887b-dd2cbffeec37.png"></div>
스마트폰 이전에는 웹서버만 구축해서 웹페이지를 보여줄 수 있었다. 하지만 스마트폰의 출현으로 다른 기기들(스마트폰, 노트북), 다른 운영체제(iOS, Android) 간에도 똑같은 컨텐츠를 제공할 필요가 생겼다.
<br>
<br>
위 그림과 같이 API가 없는 구조로는 웹클라이언트-웹서버의 상호작용만 이루어지고, 다른 클라이언트들은 DB에 각각 연결이 된다. 클라이언트들의 로직과 프로그래밍 언어도 다를 수 있다. 여기서 웹서비스에 기능을 추가하거나 버전을 업그레이드해야 한다면 클라이언트들은 각각 수정을 해줘야 한다. 시간이 많이 들고 버그 발생 가능성도 높아진다.


#### Architecture with API
<div><img width="596" alt="with_API" src="https://user-images.githubusercontent.com/63541271/136952247-a1d1fee7-57a6-4f1a-94b2-6c7c8b9f9bc2.png"></div>
이에 Json이나 XML과 같은 포맷으로 데이터를 다루는 별도의 API 서버가 필요했다. API는 DB와 각각의 클라이언트의 매개체로써 클라이언트 각각과 상호작용을 하며, 수정사항이 있다면 API 한 곳에서만 반영하면 된다. 

#### REST API
Web API의 가장 일반적인 방법이 REST이다. REST는 HTTP 기반으로 구현되었다. HTTP URI를 통해 자원을 명시한다. 하나의 URL로 4가지 HTTP 메소드(POST, GET, PUT, DELETE)를 전송할 수 있으며, 자원에 대한 CRUD 의미를 내포한다.


# 2. DRF (Django REST Framework)
DRF는 Django 안에서 RESTful API를 쉽게 구축할 수 있도록 도와주는 파이썬 오픈소스 라이브러리이다. 

## 2-1 Django vs DRF
웹 어플리케이션을 만들고 싶다면 Django를 사용하는 것이 좋다. 
API 를 구축하는 것이 목적이라면 DRF가 훨씬 유용하다. 
물론 Django만으로도 API를 만들 수 있지만, DRF가 API 작업에 최적화되어 있다. 
Serializers, OAuth, Markdown 등과 같이 관련 기능을 더 많이 제공하고, 이는 코드를 더욱 간결하게 만들어준다.

아래 예시로 Django와 DRF로 만든 API의 차이를 살펴보자.

Django만을 사용한 GET-view
```python
from django.core.serializers import serialize
from django.http import HttpResponse

class SerializedListView(View):
    def get(self, request, *args, **kwargs):
        qs = MyObj.objects.all()
        json_data = serialize("json", qs, fields=('my_field', 'my_other_field'))
        return HttpResponse(json_data, content_type='application/json')
```

DRF를 사용한 GET-view
```python
from rest_framework import generics

class MyObjListCreateAPIView(generics.ListCreateAPIView):
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    serializer_class = MyObjSerializer
```

DRF로 구축한 API가 훨씬 간결하고 인증 기능이 추가되었다.
<br>
<a href="https://stackoverflow.com/questions/49109791/django-or-django-rest-framework">참조 링크</a>

## 2-2 DRF를 사용하는 이유
- Serialization은 ORM과 non-ORM 데이터 소스를 모두 지원한다
- OAuth1a와 OAuth2용 패키지를 포함한 인증 정책을 가지고 있다
- Mozilla, Red Hat, Heroku와 같은 글로벌 기업에서도 사용한다


