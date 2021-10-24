---
title : "TIL34. Serializer field 종류"
excerpt : "SerializerMethodField, 관계된 모델간 사용하는 field들"
---

# 1. SerializerMethodField

모델에 등록된 필드가 아니고, JSON 형태로 주고 받을 때 필요한 데이터들은 SerializerMethodField로 커스텀한 필드를 생성해 사용할 수 있다. 
또는 모델에 있는 값을 변형해서 새로운 필드의 값으로 넣고 싶을 때에도 메소드를 만들어 사용할 수 있다.
<br>

Article 객체에 대해 출간한 후 시간이 얼마나 지났는지에 대한 데이터가 필요하다. 
하지만 publication_date(출간일) 필드만 모델링이 되어있다. 
이 때 serializer에서 모델에 없는 time_since_publication 필드를 추가해준다. (SerializerMethodField를 통해) 
이 필드에 들어갈 데이터를 생성이나 변형할 메소드를 만들어준다. 
메소드 이름에는 `get_[추가할 필드명]` 와 같은 규칙이 있다. 

```python
class ArticleSerializer(serializers.ModelSerializer):
    
    # 새로운 필드를 추가한다.
    time_since_publication = serializers.SerializerMethodField()

    # ModelSerializer에서는 Meta 클래스에서 시리얼라이즈를 한다. 
    # Article 모델 중 id 필드를 제외하고 시리얼라이즈 한다.
    class Meta:
        model   = Article
        exclude = ("id", )

    # object에는 직렬화된 Article 객체가 인스턴스로 들어간다. (위 Meta클래스에서 지정한 모델 객체)
    def get_time_since_publication(self, object): 
        publication_date = object.publication_date
        now              = datetime.now()
        time_delta       = timesince(publication_date, now)
        return time_delta
```

# 2. 관계된 모델 간에 사용하는 field들
관계가 있는 모델들에 대해 ForeinKey나 ManyToManyField 등을 사용하여 정의했다.
ModelSerializer 클래스에서도 관계형을 정의하기 위한 필드들이 다양하다. 
<br>

앨범과 노래트랙의 관계(1:N 관계)를 예시로 하여 각 필드들의 역할과 차이점을 알아보자.

```python
class Album(models.Model):
    album_name = models.CharField(max_length=100)
    artist = models.CharField(max_length=100)

class Track(models.Model):
    album = models.ForeignKey(Album, related_name='tracks', on_delete=models.CASCADE)
    order = models.IntegerField()
    title = models.CharField(max_length=100)
    duration = models.IntegerField()

    class Meta:
        unique_together = ['album', 'order']
        ordering = ['order']

    def __str__(self):
        return '%d: %s' % (self.order, self.title)
```

## 2-1 StringRelatedField
StringRelatedField는 `__str__` 메소드를 사용하여 관계의 대상을 나타내는데 사용된다.
Track 모델을 만들 때 `__str__` 메소드도 같이 만들었는데, 리턴되는 값이 order필드값: title필드값 임을 알 수 있다.


```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.StringRelatedField(many=True)

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']
```
아래 album에 대해 시리얼라이즈한 결과를 보면, tracks의 value 구조가 Track모델의 `__str__` 리턴값으로 나타난 것이다.

```python
# 출력 결과
{
    'album_name': 'Things We Lost In The Fire',
    'artist': 'Low',
    'tracks': [
        '1: Sunflower',
        '2: Whitetail',
        '3: Dinosaur Act',
        ...
    ]
}
```

## 2-2 PrimaryKeyRelatedField
PrimaryKeyRelatedField는 primary 키를 사용하여 관계의 대상을 나타낸다.

```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.PrimaryKeyRelatedField(many=True, read_only=True)

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']

```

```python
# 출력 결과
{
    'album_name': 'Undun',
    'artist': 'The Roots',
    'tracks': [
        89,
        90,
        91,
        ...
    ]
}

```

## 2-3 HyperlinkedRelatedField
관계된 객체들에 대해 연결지을 때 사용할 수 있다. 이 필드를 사용하면 기본값으로 id 필드를 포함하지 않는다. url과 관련된 파라미터 `view_name`을 통해 해당 url들을 리턴한다.

```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.HyperlinkedRelatedField(
        many=True,
        read_only=True,
        view_name='track-detail'
    )

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']
```

```python
# 출력 결과
{
    'album_name': 'Graceland',
    'artist': 'Paul Simon',
    'tracks': [
        'http://www.example.com/api/tracks/45/',
        'http://www.example.com/api/tracks/46/',
        'http://www.example.com/api/tracks/47/',
        ...
    ]
}
```

[참고 자료 클릭](https://runebook.dev/ko/docs/django_rest_framework/api-guide/relations/index)
