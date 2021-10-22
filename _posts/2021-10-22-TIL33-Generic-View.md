---
title : "TIL33. Generic View"
excerpt : "Generic View와 Mixin 클래스"
---

## GenericAPIView
GenericAPIView 클래스는 `queryset`이나 `serializer_class`와 같은 속성을 제공하는데, CRUD API를 작성하는데 공통으로 필요한 속성들을 제공한다.
Mixin클래스는 CRUD에 필요한 메서드들을 제공한다.
GenericAPIView는 하나 이상의 믹스인 클래스와 결합하여 사용할 수 있다.
GenericAPIView를 상속받은 클래스는 제네릭뷰의 위 속성들이나 `get_queryset()` `get_serializer_class()`메소드를 선언해야 한다. 


#### Mixin 클래스의 종류
- ListModelMixin : `.list` 메서드를 가지고 있고, 쿼리셋을 리스팅하는 기능을 한다.
- CreateModelMixin : `.create` 메서드를 호출하여 모델 인스턴스를 생성하여 생성한 객체를 응답한다.
- RetrieveModelMixin : `.retrieve` 메서드는 기존 모델 인스턴스를 응답한다.
- UpdateModelMixin : `.update` 메서드로 기존 모델 인스턴스를 업데이트한다.
- DestroyModelMixin : `.destroy` 메서드로 기존 모델 인스턴스를 삭제한다.

GenericAPIView와 Mixin클래스를 결합하여 Ebook 리스트의 get과 post 요청을 아래와 같이 작성할 수 있다.

```python
class EbookListCreateAPIView(mixins.ListModelMixin, 
                            mixins.CreateModelMixin,
                            generics.GenericAPIView):

    queryset         = Ebook.objects.all() 
    serializer_class = EbookSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

```

ListModelMixin 클래스를 보면 GenericAPIView의 메서드 중 기능에 필요한 메서드들만 사용하여 list라는 새로운 메서드들을 만들었다.
```python
class ListModelMixin:
    """
    List a queryset.
    """
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())

        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)
```



그냥 APIView에서는 각 request 메소드마다 직접 serialize 처리를 해주어야 하고 중복이 발생되었다.
GenericAPIView를 사용하면 클래스를 상속할 때 한번 queryset과 serialize를 정의하기만 하면 된다. 나머지는 Mixin과 연결된다. 

Mixin을 상속함으로써 반복되는 내용을 많이 줄일 수 있다. 하지만 여러개의 Mixin을 상속해야 하다보니 가독성이 떨어진다. 
이에 rest_framework에서는 Mixin과 GenericAPIView를 합친 또다른 클래스들을 정의해놓았다.

- generics.CreateAPIView : 생성
- generics.ListAPIView : 목록
- generics.RetrieveAPIView : 조회
- generics.DestroyAPIView : 삭제
- generics.UpdateAPIView : 수정
- generics.RetrieveUpdateAPIView : 조회/수정
- generics.RetrieveDestroyAPIView : 조회/삭제
- generics.ListCreateAPIView : 목록/생성
- generics.RetrieveUpdateDestroyAPIView : 조회/수정/삭제

Ebook 리스트 조회와 생성 API를 `ListCreateAPIView` 클래스를 상속받으면 아래와 같이 queryset과 serialize만 정의하면 된다. 
get과 post를 위한 메서드는 따로 정의를 해주지 않아도 된다.

```python
class EbookListCreateAPIView(generics.ListCreateAPIView):
    queryset         = Ebook.objects.all()
    serializer_class = EbookSerializer
```
