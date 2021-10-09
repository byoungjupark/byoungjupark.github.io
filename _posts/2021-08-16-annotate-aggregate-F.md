---
title: "TIL30. annotate & aggregate & F"
excerpt: "장바구 GET API에 사용된 Django 쿼리 메소드"
---

장바구니 GET API 작성 시 총 주문 금액을 annotate와 aggregate, F 객체를 사용하여 나타낼 수 있다.


``` python
# carts.models.py
class Cart(models.Model):
    user    = models.ForeignKey('users.User', on_delete=models.CASCADE)
    product = models.ForeignKey('products.Product', on_delete=models.CASCADE)
    size    = models.ForeignKey('products.Size', on_delete=models.CASCADE)
    count   = models.PositiveIntegerField(default=0)

    class Meta:
        db_table = 'carts'
```

## 1. annotate
장바구니 객체는 가격 필드가 없지만, annotate를 사용하면 각각의 row에 대한 주문금액은 계산이 가능하다. 아래 표는 장바구니 객체를 표현한 것이고, annotate 메소드를 이용하면 `상품 수량 x 상품 가격` 이라는 가상의 새로운 필드를 추가할 수 있다.

<img src="https://images.velog.io/images/byoungju1012/post/b6c0c267-fcd9-47cc-8ae2-d68d2664babe/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-08-09%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%209.08.11.png">


> **annotate
'주석을 달다' 라는 뜻이지만, 장고 ORM 에서는 새로운 가상의 필드를 추가한다는 의미로 받아들여진다.**
이 메소드가 필요한 이유는?
'장바구니 객체에 주문금액 필드를 추가하면 되지 않을까' 라는 1차원적인 생각을 할 수도 있지만, 주문금액은 상품의 수량과 가격에 따라 변화하는 특징이 있다. POST 요청과 응답이 이루어진 후 상품의 금액이 바뀌었을 경우, GET 응답시 업데이트된 금액이 반영되지 않는다. 
이 외에도 특정 조건에 따른 필드의 집계값을 구하고 싶을 때, 필드명을 바꿔서 응답할 때 사용할 수 있다.

유저 51번의 장바구니 총 주문금액을 구하는 방법 2가지가 있다. (접근하는 본질은 같다)
1) annotate로 각 row별 주문금액을 계산한 뒤, aggregate로 총 합계를 구하는 방법
2) aggregate 메소드 안에서 row별 주문금액을 바로 넣어 계산하는 방법
```python
# 상품별 주문금액 구하는 방법 1
>>> carts = Cart.objects.filter(user=51).annotate(price = F("count") * F("product__price"))
carts.aggregate(total_price = Sum("price"))
{'total_price': Decimal('1990186.00')}

# 상품별 주문금액 구하는 방법 2
>>> Cart.objects.filter(user=51).aggregate(total_price = Sum(F("count")*F("product__price")))
{'total_price': Decimal('1990186.00')}
```

#### *annotate 응용

```python
# products.models.py
class Category(models.Model):
    category = models.CharField(max_length=45)

class SubCategory(models.Model):
    category     = models.ForeignKey('Category', on_delete=models.SET_NULL, null=True)
    sub_category = models.CharField(max_length=45)

class Product(models.Model):
    sub_category     = models.ForeignKey('SubCategory', on_delete=models.SET_NULL, null=True)
    name             = models.CharField(max_length=45)

```
Category - SubCategory - Product 순의 1:N 관계 모델링이다.
서브 카테고리별 상품 개수와 카테고리별 상품 개수를 annotate를 사용하여 구할 수 있다.

```python
# 서브 카테고리별 상품 개수
>>> Product.objects.values("sub_category_id").annotate(count = Count("sub_category"))
<QuerySet [{'sub_category_id': 1, 'count': 12}, {'sub_category_id': 2, 'count': 12}, {'sub_category_id': 3, 'count': 10}, {'sub_category_id': 4, 'count': 10}, {'sub_category_id': 5, 'count': 64}, {'sub_category_id': 6, 'count': 22}]>

# 카테고리별 상품 개수
>>> Product.objects.values("sub_category__category_id").annotate(count = Count("sub_category__category"))
<QuerySet [{'sub_category__category_id': 1, 'count': 24}, {'sub_category__category_id': 2, 'count': 20}, {'sub_category__category_id': 3, 'count': 64}, {'sub_category__category_id': 4, 'count': 22}]>
```
## 2. F 객체
여기서 F객체의 개념을 알아보자
왜 F객체를 써야하는가?

유저 51번의 장바구니를 조회할 때를 살펴보자. 
F객체를 사용하지 않고 바로 필드에 접근할 경우, `TypeError`가 발생하며 각 주문금액도 마지막 인스턴스의 집계값으로 반환된다.
aggregate 메소드로 총 합계를 구할 때에도 `FieldError` 가 발생한다. 

**F객체는 쿼리셋 또는 인스턴스에 접근하는 것이 아닌 해당하는 Field 값에 바로 접근할 수 있다. 참조하는 객체의 값도 바로 불러올 수 있다. **

```python
# F객체 미사용
>>> carts = Cart.objects.filter(user=51).annotate(price = cart.count * cart.product.price)
TypeError: QuerySet.annotate() received non-expression(s)

for cart in carts:
	print(price)
    
1462670.00
1462670.00
1462670.00
1462670.00

>>> carts.aggregate(total_price = Sum("price"))
FieldError: Cannot resolve keyword 'price' into field

# F객체 사용
>>> carts = Cart.objects.filter(user=51).annotate(price = F("count") * F("product__price"))
carts.aggregate(total_price = Sum("price"))
{'total_price': Decimal('1990186.00')}


