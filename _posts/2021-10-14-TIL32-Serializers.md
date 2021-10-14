---
title : "TIL32. Serializers"
excerpt : "Serializes의 개념, render와 parse"
---

Django Rest Framework 실습을 하며 개념을 익혀보자

# 1. Serializers란?
직렬화라는 뜻
<br>
Serializers는 쿼리셋이나 모델 인스턴스와 같은 복잡한 데이터들을 파이썬 데이터 타입으로 쉽게 변환하여 JSON이나 다른 content type에 쉽게 렌더링할 수 있게 해준다. 
DRF에서 중요한 기능 중 하나이고, `Serializer`나 `ModelSerializer` 클래스를 사용한다.
<br>
deserialization 기능도 제공하는데, parsing된 데이터를 validate 한 후 복잡한 데이터 타입(쿼리셋, 모델 인스턴스)으로 다시 변환하는 작업을 말한다.

### 1-1 Serializer 클래스 만들기
장고에서는 기본적으로 serializers 파일을 제공하지 않기 때문에 DRF을 사용한다면 수동으로 serializers.py을 만들어주어야 한다. 
이 때, app 디렉토리에 하위 디렉토리 하나를 만들어 준후, 그 파일을 생성하도록 한다. 
DRF는 REST API를 구축하는데 특화된 라이브러리이기 때문에 api 디렉토리를 만들어 이 안에서 관리해주도록 한다.

순수 장고에서 작성한 모델과 rest_framework의 serializers 모듈을 import 해온다.
상속받은 Serializer 클래스에는 이미 만들어진 create와 update 메소드가 있는데, 이를 이용해 객체에 접근하여 생성하거나 수정할 수 있다.

```python
# news/api/serializers.py
from rest_framework import serializers
from news.models    import Article

class ArticleSerializer(serializers.Serializer):
    id               = serializers.IntegerField(read_only=True)
    author           = serializers.CharField()
    title            = serializers.CharField()
    description      = serializers.CharField()
    body             = serializers.CharField()
    location         = serializers.CharField()
    publication_date = serializers.DateField()
    active           = serializers.BooleanField()
    created_at       = serializers.DateTimeField(read_only=True)
    updated_at       = serializers.DateTimeField(read_only=True)

    def create(self, validated_data):
        return Article.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.author            = validated_data.get('author', instance.author)
        instance.title             = validated_data.get('title', instance.title)
        instance.description       = validated_data.get('description', instance.description)
        instance.body              = validated_data.get('body', instance.body)
        instance.location          = validated_data.get('location', instance.location)
        instance.publication_date  = validated_data.get('publication_date', instance.publication_date)
        instance.descripactivetion = validated_data.get('active', instance.active)
        instance.save()
        return instance 

```

- create : validated_data 파라미터에는 keyword argument가 들어가고, `ArticleSerializer` 클래스를 통해 `Article` object의 instance를 생성할 수 있다.

- update : instance와 validated_data 2개의 파라미터를 받는다. 
instance는 기존 객체의 instance를 가져오고, valideated_data는 request한 객체를 가져온다. 
dictionary get 메소드로 validated_data의 각 key가 있다면 그 value를, 없다면 기존 instance의 데이터를 instance의 각 필드에 대입한다. 마지막으로 저장을 해준다.
<br>
serializer를 구축해놓으면 View에서의 로직이 훨씬 간단해진다.

### 1-2 serialize, render, parse
shell에서 serialize와 rendering, parsing을 해보자.

#### serialize 하기
`Article` 객체를 직렬화한 후 데이터를 출력해 보았다. 
'active'의 value는 Boolean 타입으로 되어있다. JSON 형태로 응답하려면 문자열로 전환해줘야한다. 
아래 `JSONRenderer`를 사용해 타입을 변경할 수 있다.

```python
>>> from news.api.serializers import ArticleSerializer
>>> from news.models import Article

#Article 객체를 직렬화하기
>>> article_instance = Article.objects.first()
>>> serializer = ArticleSerializer(article_instance)
>>> serializer.data
{
    'id': 1, 
    'author': 'John Doe', 
    'title': 'Happy Birthday ISS: 20 Years of The International Space Station',
    'description': 'some fancy desce', 
    'body': 'some content', 
    'location': 'Earth', 
    'publication_date': '2021-10-11', 
    'active': True, # Boolean 타입
    'created_at': '2021-10-11T07:50:04.682553Z', 
    'updated_at': '2021-10-11T07:50:04.682589Z'
}
```

#### rendering과 parsing 하기
`JSONRenderer`로 직렬화한 객체를 JSON 형태로 변경해주었다.

```python
>>> from rest_framework.renderers import JSONRenderer

# rendering 하기
>>> json = JSONRenderer().render(serializer.data)
>>> json
b'{
    "id":1,
    "author":"John Doe",
    "title":"Happy Birthday ISS: 20 Years of The International Space Station",
    "description":"some fancy desce",
    "body":"some content",
    "location":"Earth",
    "publication_date":"2021-10-11",
    "active":true,
    "created_at":"2021-10-11T07:50:04.682553Z",
    "updated_at":"2021-10-11T07:50:04.682589Z"
}
```

파이썬 내장 모듈 io를 사용해서 스트림작업을 할 수 있다. 
스트림은 데이터 입출력(input, output)의 흐름을 의미하고, 입출력의 중간자 역할을 한다.
<br>
JSON 형태를 파이썬 byte 타입으로 변환한 뒤, JSONParser로 parsing 해준다.
이제 parsing된 객체를 ArticleSerializer 클래스의 파라미터로 넣어준다.
`is_valid()` 메소드로 유효성 검사를 한 후 저장하면 Article 객체에 새로운 인스턴스가 생성된다.
참고로 `.validated_data`, `.errors`, `.data`는 `is_valid()`를 호출한 다음 사용할 수 있다.

```python
>>> import io
>>> from rest_framework.parsers import JSONParser

# stream과 parsing 하기
>>> stream = io.BytesIO(json)
>>> data = JSONParser().parse(stream)
>>> data
{
    'id': 1, 
    'author': 'John Doe', 
    'title': 'Happy Birthday ISS: 20 Years of The International Space Station',
    'description': 'some fancy desce', 
    'body': 'some content', 
    'location': 'Earth', 
    'publication_date': '2021-10-11', 
    'active': True, 
    'created_at': '2021-10-11T07:50:04.682553Z', 
    'updated_at': '2021-10-11T07:50:04.682589Z'
}

# parsing된 객체 validation 검사 후 새로운 인스턴스 생성하기 
>>> serializer = ArticleSerializer(data=data)
>>> serializer.is_valid()
True
>>> serializer.validated_data
OrderedDict([
  ('author', 'John Doe'), 
  ('title', 'Happy Birthday ISS: 20 Years of The International Space Station'), 
  ('description', 'some fancy desce'), 
  ('body', 'some content'), 
  ('location', 'Earth'), 
  ('publication_date', datetime.date(2021, 10, 11)), 
  ('active', True)
])


>>> serializer.save()
{
    'author': 'John Doe', 
    'title': 'Happy Birthday ISS: 20 Years of The International Space Station',
    'description': 'some fancy desce', 
    'body': 'some content', 
    'location': 'Earth', 
    'publication_date': datetime.date(2021, 10, 11), 
    'active': True
}
<Article: John Doe Happy Birthday ISS: 20 Years of The International Space Station>

```

