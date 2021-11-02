---
title : "TIL37. Paginations"
excerpt : "Pagination이란, LimitoffsetPagination"
---

## Pagination
상품 리스트 페이지에서 데이터를 렌더링할 때, 데이터가 많다면 무한 스크롤로 이어질 수 있다. 
이를 위해 Pagination이 필요하다. 
Pagination은 이름에서도 유추할 수 있듯이, 데이터를 페이지별로 보여주게 하는 기능이다. (ex. 상품 30개/1페이지)
DRF에서도 Pagination 클래스라는 특정 기준을 설정하여 데이터를 페이지 별로 나누어 관리하는 기능을 제공한다. 
다만 pagination은 `generic view`나 `viewsets`에서만 사용할 수 있다. 만약 일반 `APIView`를 사용한다면, pagination API를 따로 만들어야 한다.
<br>

`DEFAULT_PAGINATION_CLASS`, `PAGE_SIZE` 키를 사용하여 `settings.py`에서 전역으로 설정할 수 있다. 
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 3
}
```

아니면 각각의 view별로 pagination을 설정할 수도 있다.
`pagination.py` 파일을 만들어서 `PageNumberPagination` 클래스를 상속받는다. 
page_size 속성에 몇개의 인스턴스까지 1페이지에 허용할지를 적어주면 된다.
이 클래스는 쿼리 파라미터로 현재 페이지 넘버를 받고, 이전과 다음 페이지 url도 같이 response한다.
```python
# api/pagination.py
from rest_framework.pagination import PageNumberPagination

class SmallSetPagination(PageNumberPagination):
    page_size = 3
```

이후 `views.py`에서 pagination_class 속성에 좀 전에 만든 `SmallSetPagination` 클래스를 대입한다. 
아래 클래스는 `generics`클래스를 상속받았기 때문에, 자동으로 pagination 기능을 사용할 수 있다.
pagination을 하려면 정렬 기준이 필요하기 때문에, 이 클래스에서는 id 최신순으로 정렬을 해 주었다.
```python
# api/views.py
from ebooks.api.pagination  import SmallSetPagination

class EbookListCreateAPIView(generics.ListCreateAPIView):
    queryset           = Ebook.objects.all().order_by("-id")
    serializer_class   = EbookSerializer
    permission_classes = [IsAdminUserOrReadOnly]
    pagination_class   = SmallSetPagination
```

## LimitOffsetPagination
LimitOffsetPagination은 쿼리 파라미터로 limit과 offset을 포함한다. limit은 1페이지에 최대로 받을 인스턴스의 개수를 나타내고, offset은 시작점을 나타낸다.
<br>

아래와 같이 1페이지에 나타낼 인스턴스는 100개이고, 정렬기준이 id라고 가정할때 시작점은 id 400부터이다.
```
# Request
GET https://api.example.org/accounts/?limit=100&offset=400
```

이에 대한 응답으로 이전 페이지의 offset은 id 300번이고, 다음 페이지의 offset은 id 500번이 된다. 
```python
# Response
HTTP 200 OK
{
    "count": 1023
    "next": "https://api.example.org/accounts/?limit=100&offset=500",
    "previous": "https://api.example.org/accounts/?limit=100&offset=300",
    "results": [
       …
    ]
}
```

