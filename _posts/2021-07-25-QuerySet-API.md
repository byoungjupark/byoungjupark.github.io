---
title: "TIL22.  QuerySet API"
excerpt: "get_or_create, update_or_create, aggregate, get과 filter, filter lookups"
---

장고 crud를 진행하면서 쿼리셋 메소드에 대해 좀 더 배워야겠다는 생각이 들었다. 지금까지는 `all`, `create`, `get`, `filter` 메소드 정도만 많이 사용했는데, 다른 메소드들도 장고쉘에서 사용해보았다. 지난번 주인과 강아지 과제로 쿼리셋 메소드를 사용해보았다.


## 1. get_or_create
> Owner.objects.get_or_create(**kwargs)
return (object, Boolean Tag)
객체가 데이터베이스에 존재하지 않다면 새로운 객체를 생성하고 그 객체와 True를 반환한다. 존재하면 존재하는 객체와 False를 반환한다. 
튜플 형태로 반환한다.

지금까지는 객체를 생성할 때 `create`를 사용해왔다. `get_or_create`는 중복되는 객체가 있는지 먼저 체크한 후 생성한다. 

![](https://images.velog.io/images/byoungju1012/post/9280f9ab-8c34-4377-af48-33f46ada49c1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.49.41.png)

* `Owner.objects.get_or_create(name="키", ~)`
이미 id 1번에 존재하는 데이터이므로 False 를 반환한다.

* `Owner.objects.get_or_create(name="태연", email="ty@email.com", age=20)` 
데이터베이스에 없기 때문에 `(<Owner: Owner object (8)>, True)` 을 반환한다. 데이터베이스에 접속하면 id 8번으로 새로운 객체를 생성한 것을 확인할 수 있다.

<img src="https://images.velog.io/images/byoungju1012/post/821a5b1f-784e-47f1-a724-76cf2dc8a66d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.49.50.png" width=400>

## 2. update_or_create
>Owner.objects.update_or_create(**kwargs, default={})
return (object, Boolean Tag)
`keyword arguments`에 해당하는 객체가 없다면, 새로운 객체를 생성하고 True를 반환한다. 객체가 있다면 default의 인자를 업데이트하고  False를 반환한다.

좀 전에 생성한 Owner id가 8번인 객체를 예로 들어 알아보자.
* `Owner.objects.update_or_create(name="태연", email="ty@email.com", age=10, default={"name" : "지연"})`
메소드 안의 keyword arguments가 이미 존재한므로 default 의 name만 '지연'으로 업데이트되었다.

<img src="https://images.velog.io/images/byoungju1012/post/a68ced8e-6477-419b-a469-10363f00b1d5/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.52.24.png" width=400>

* `Owner.objects.update_or_create(name="태연", email="ty@email.com", age=10, default={"name" : "지연"})` 
한번 더 같은 메소드를 작성해보자. 
이번에는 keyword arguments가 데이터베이스에 없다. 이름이 '지연'으로 업데이트되었기 때문에 '태연'이 없다. 따라서 새로운 객체가 생성되면서 이름은 default의 값을 따른다. (아래 id 11번)

<img src="https://images.velog.io/images/byoungju1012/post/288c2a90-3983-4454-828a-ecc9a1ea1bda/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%209.18.13.png" width=400>

## 3. aggregate

>from django.db.models import Avg, Max, Min
Owner.objects.all().aggregate(Avg("age"))
필드 전체를 통합하여 합계, 평균, 최대값, 최소값 등을 구할 때 사용한다. 
계산은 장고 모듈을 불러와서 사용할 수 있다.
딕셔너리 형태로 반환한다.

![](https://images.velog.io/images/byoungju1012/post/8defeffa-b956-4cee-a76d-7b59524b5bb3/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-25%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.05.10.png)

`Owner`의 평균 나이를 계산할 수 있다. 반환된 키 값은 장고에서 자동으로 생성해준다. 다른 이름으로 바꾸고 싶다면 `(average=Avg("age"))`와 같이 원하는 이름을 작성하면 된다.

## 4. get과 filter 

### 4-1 관계를 통한 조회
<img src="https://images.velog.io/images/byoungju1012/post/d8d5981d-955d-487c-a19e-844ac32897c4/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2010.55.19.png" width=400>

이전에는 관계가 있는 두 모델에 대해 조회를 할 때 정참조와 역참조를 생각하며 메소드를 작성했다. 정참조인 `Dog`은 위와 같이 한 줄로 작성하여 `Owner`의 이름을 출력할 수 있다. 역참조의 경우는 자신을 참조하는 객체를 모르기 때문에 attribute error 가 발생한다. 대신 `_set`과 for 문을 사용해 `Dog` 이름을 출력할 수 있다.

![](https://images.velog.io/images/byoungju1012/post/bc95e780-6c87-45a7-92d1-c4ab3e62cbc0/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2011.15.21.png)

이중밑줄 `__`을 사용하면 정참조와 역참조와 무관하게 작용한다.
`get/filter(관계되는 객체__필드명 = 필드값)`

![](https://images.velog.io/images/byoungju1012/post/01e46fa6-5883-4103-b968-2a1491640939/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2011.21.55.png)

`Actor`와 `Movie` 와 같은 M2M 관계에서도 가능하다. 

<img src="https://images.velog.io/images/byoungju1012/post/039a4513-adfc-440e-99b5-b12efdd42a9c/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.54.18.png" width=400>

`Actor.objects.filter(movie__title="돈")`
영화 '돈'에 출연한 배우를 조회해준다.

`Movie.objects.filter(actor__first_name="준열")`
배우 '준열'이 출현한 영화들을 조회해준다.

![](https://images.velog.io/images/byoungju1012/post/cfe88a49-feb8-4fca-9c7a-9e33c86db9c2/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.55.35.png)

중간테이블로 연결된 M2M 관계와 같이 여러 모델이 연결된 관계에 대해서도 필터링할 수 있다. 중간테이블을 먼저 쓴 다음 연결될 모델과 필드명을 적어준다.
`Actor.objects.filter(actormovie__movie__title="0title")`

![](https://images.velog.io/images/byoungju1012/post/4777d196-c2e8-4bdd-9923-87d407f7d8e9/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.58.34.png)

### 4-2 lookups
>`field__lookuptype = value`
lookup 옵션은 관계된 다른 모델의 쿼리셋을 가져올 수 있다. filter, exclude, get 메소드에서 사용할 수 있다.

* lt / gt / lte / gte : ~보다 작다 / ~보다 크다 / ~보다 작거나 같다 / ~보다 크거나 같다
예) model.objects.filter(id__lt=5) 
id가 5보다 작은 데이터를 검색한다.

* in : 반복가능한 타입을 넣어준다. (리스트, 튜플, 쿼리셋) 문자열도 가능하다. 
예) model.objects.filter(id__in[1,2,4]) 
id 1,2,4 중 존재하는 데이터만 반환한다. 아무것도 없으면 빈 쿼리셋을 반환한다.

* year, month, day : 연도, 월, 일 검색
예) model.objects.filter(published_date\__year=2019) 
published_date가 2019년인 데이터를 검색한다.
model.objects.filter(published_date__year\__gt=2019)
published_date가 2019년 이후의 데이터를 검색한다.

* isnull : null인 데이터 검색
예) model.objects.filter(name__isnull=True)

* contains / icontains : 지정값이 포함된 데이터를 검색 / 대소문자 구분 x
예) model.objects.filter(name__contains="김")
name에 "김"이 포함된 데이터를 검색한다.

* startswith / endswith / istartswith, iendswith : 지정값으로 시작하는 데이터를 검색 / 지정값으로 끝나는 데이터 검색 / 대소문자 구분 x

* exact / iexact : 지정값과 일치하는 데이터 검색 / 대소문자 구분 x

* range : 해당 범위 안의 데이터를 검색한다. 문자, 숫자, 날짜 등의 범위로 지정할 수 있다.

![](https://images.velog.io/images/byoungju1012/post/d2ade940-f2dc-4c5a-87cc-d84f254c7be4/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2010.59.35.png)

`Owner.objects.filter(age__lt=20)` 
`Owner`의 나이가 20살 미만인 데이터를 검색하여 쿼리셋으로 반환해준다.

`Owner.objects.filter(dog__age__gt=4)`
`Dog` 모델 자체에서 필터링할 수 있지만, `Dog`의 나이가 4살 초과인 데이터를 검색할 때 이런 식으로도 표현이 가능하다.
