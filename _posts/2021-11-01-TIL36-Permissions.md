---
title : "TIL36. Permissions"
excerpt : "Permission의 개념과 종류, 커스텀 permissions 만들기"
---

## Permissions란
Permissions란 서비스를 어느 정도로 이용할 수 있는지에 대한 권한을 말한다. 
인증(Authentication)할 때 알아낸 사용자의 정보(`request.user`나 `request.auth`를 기반으로 체크한다.
DRF에서 permissions는 항상 리스트로써 정의된다. view 본문을 실행하기 전에 list의 각 permission을 검사하는데, 확인에 실패하면 `exceptions.PermissionDenied`나 `exceptions.NotAuthenticated` 예외가 발생하며 view가 실행되지 않는다. 
이후 `403 Forbidden`이나 `401 Unauthorized` 응답을 반환시킨다.
<br>

DRF는 Object-level permissions를 지원한다. 
Object level permissions는 사용자가 특정 개체(일반적으로 모델 인스턴스)에 대한 작업을 허용해야 하는지 여부를 결정하는데 사용된다.
object level permission는 `.get_object()`가 호출될 때 DRF의 generic views에 의해 실행된다.
<br>

`settings.py`에서 전역으로 설정할 수 있다. 
하지만 부여할 권한이 view마다 다를 수 있으므로, api view에서 직접 권한을 지정할 수 도 있다.

```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ]
}
```
## Permissions의 종류
DRF에서 기본으로 제공하는 permissions는 다음과 같다.
- AllowAny (디폴트 전역 설정) : 인증 여부에 상관없이 뷰 호출을 허용
- IsAuthenticated : 인증된 요청에 한해서 뷰 호출 허용 (로그인이 되어있어야만 접근 허용)
- IsAdminUser : Staff 인증 요청에 한해서 뷰 호출 허용
- IsAuthenticatedOrReadOnly : 비인증 요청에게는 읽기 권한만 허용 (로그인이 되어 있지않아도 조회는 가능)
- DjangoModelPermissons : 인증된 요청에 한하여 뷰 호출 허용, 추가로 장고 모델단위 Permissions 체크
- DjangoModelPermissionsOrAnonReadOnly : DjangoModelPermissions와 유사, 비인증 요청에게는 읽기만 허용
- DjangoObjectPermissons : 비인증 요청은 거부, 인증된 요청은 Object에 대한 권한 체크 수행
<br>

[참고 링크](https://donis-note.medium.com/django-rest-framework-authentication-permission-%EC%9D%B8%EC%A6%9D%EA%B3%BC-%EA%B6%8C%ED%95%9C-cc9b183fd901)


## 커스텀한 Permission 만들기 (1)
어드민 유저에게만 생성, 수정, 삭제 권한을 주려고 하는데 기존 permissions에는 딱 맞는 것이 없다. 
이 때 DRF에서 제공하는 permissions를 상속받아 커스텀하게 사용할 수도 있다.
<br>

커스텀 permissions를 만드려면 `BasePermission`을 오버라이딩하고 이 클래스가 가진 메소드 `has_permission`, `has_object_permission` 2가지 중 최소 1개 이상은 사용하여야 한다. 
DRF의 permissions들은 `BasePermission`의 자녀클래스로 상황에 맞게 적절한 permissions를 상속해도 된다.

```python
SAFE_METHODS = ('GET', 'HEAD', 'OPTIONS')

# 기본 제공하는 IsAdminUser 클래스
# 유저가 존재하고 어드민 유저일 경우에만 True를 반환한다.
class IsAdminUser(BasePermission):
   def has_permission(self, request, view):
       return bool(request.user and request.user.is_staff)
```

우선 permissions.py를 만들어주고 `IsAdminUser`클래스를 상속받아온다.
`is_admin`은 True나 False로 반환된다.
SAFE_METHODS에는 POST, PUT, DELETE가 없기 때문에 읽기 작업과 쓰기 작업을 구분할 때 사용할 수 있다.
아래 커스텀 permissions는 에러처리를 제외한 4가지 경우로 나눠볼 수 있고, 4번을 제외하고 모두 True를 반환한다. 

1. request.method가 SAFE_METHODS에 있거나 is_admin이 True일 경우 -> True 반환 (어드민 유저이고 조회가능)
2. request.method가 SAFE_METHODS에 있거나 is_admin이 False일 경우 -> True 반환 (일반 유저이고 조회가능)
3. request.method가 SAFE_METHODS에 없거나 is_admin이 True일 경우 -> True 반환 (어드민 유저이고 생성,수정,삭제 가능)
4. request.method가 SAFE_METHODS에 없거나 is_admin이 False일 경우 -> False 반환 (어드민 유저아니여서 생성,수정,삭제 불가능)

```python
# api/permissions.py
from rest_framework import permissions

class IsAdminUserOrReadOnly(permissions.IsAdminUser):
    def has_permission(self, request, view):
        # super는 기존 상속한 클래스의 메소드를 그대로 사용하겠다는 의미
        is_admin = super().has_permission(request, view) 
        return request.method in permissions.SAFE_METHODS or is_admin 
```

커스텀하게 작성한 `IsAdminUserOrReadOnly` 클래스를 임포트한다. 
Admin일 경우에만 POST를 할 수 있도록 권한을 주었다. 
다른 유저들은 권한이 없기 때문에 읽기 작업만 가능하다.

```python
# api/views.py
from ebooks.api.permissions import IsAdminUserOrReadOnly

class EbookListCreateAPIView(generics.ListCreateAPIView):
    queryset           = Ebook.objects.all().order_by("-id")
    serializer_class   = EbookSerializer
    permission_classes = [IsAdminUserOrReadOnly]
```

## 커스텀한 Permission 만들기 (2)
이번에는 리뷰와 관련된 커스텀 permission을 만들어보자.
회원만 리뷰를 작성할 수 있고, 리뷰는 1번만 작성할 수 있도록 한다.
<br>

`has_object_permission` 메소드를 사용하면 APIView의 get_object 메소드를 통해 object를 가져올 수 있다. 
여기서 `has_permission` 검사가 이미 통과된 경우에만 인스턴스 수준의 `has_object_permission` 메소드가 호출된다.

```python
# api/permissions.py
class IsReviewAuthorOrReadOnly(permissions.BasePermission):
    
    def has_object_permission(self, request, view, obj):
        # 읽기 권한도 허용한다
        if request.method in permissions.SAFE_METHODS:
            return True
        # 인스턴스에 review_author라는 속성이 있어야 한다
        return obj.review_author == request.user
```


```python
# api/views.py
class ReviewCreateAPIView(generics.CreateAPIView):
    queryset           = Review.objects.all()
    serializer_class   = ReviewSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

    def perform_create(self, serializer):
        ebook_pk = self.kwargs.get("ebook_pk")
        ebook    = get_object_or_404(Ebook, pk=ebook_pk)
        
        review_author   = self.request.user
        review_queryset = Review.objects.filter(ebook=ebook, review_author=review_author)

        if review_queryset.exists():
            raise ValidationError("You Have Already Reiviewed this Ebook!")

        serializer.save(ebook=ebook, review_author=review_author)

class ReviewDetailAPIView(generics.RetrieveUpdateDestroyAPIView):
    queryset            = Review.objects.all()
    serializer_class    = ReviewSerializer
    permissions_classes = [IsReviewAuthorOrReadOnly]
```
