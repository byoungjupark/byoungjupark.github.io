---
"title" : "TIL42. getter와 setter, property"
---

## getter와 setter
파이썬 클래스에는 getter와 setter라는 메서드가 있다.
getter는 값을 가져오는 메서드이고, setter는 값을 변경해주는 메서드이다.
함수명은 보통 get과 set을 붙여준다.

```python
class Natural:
    def __init__(self, n):
        self.n = n

    def get_n(self): 
        return self.__n

    def set_n(self, n):
        if n<1:
            self.__n = 1
        else:
            self.__n = n
            
>>> num = Natural(7)
>>> num.n
7
>>> num.n = 20
>>> num.n
20
>>> num.n=0
>>> num.n
1
```

Natural 클래스에 대해 `get_n` 과 `set_n` 이라는 메서드를 만들어준다.
Natural 클래스의 인스턴스 num을 생성했다. 
num.n을 출력할 때 getter 메서드가 실행되며 7을 반환해준다.
num.n에 20을 할당하였다. 이 때 setter 메서드가 실행되는데 값을 할당(대입)하는 시점에 함수가 동작한다.
정말 setter가 실행된 것이 맞는지 `set_n` 함수의 로직을 통해 걸러보자.
num.n에 0을 할당한 후 속성을 출력해보면 1이 나오는 것을 알 수 있다. 
n이 1미만일 경우 1을 반환하는 제어문이 작동한 것을 알 수 있다.

## property
getter와 setter가 아닌 property로 클래스의 속성을 표현할 수도 있다.
getter메서드 대신 `@property` 를  추가해주고, setter 메서드는 `@property의 함수명.setter` 를 함수 위에 추가하면 된다.

```python
class Natural:
    def __init__(self, n):
        self.n = n

    @property
    def n(self):
        return self.__n

    @n.setter
    def n(self, n):
        if n<1:
            self.__n = 1
        else:
            self.__n = n
```

property 와 setter메서드는 언제 사용하는 것이 좋을까?
우선 파이썬 뿐만 아니라 많은 언어들이 클래스 기반의 객체지향  프로그래밍을 사용한다. 
파이썬에서는 `@property` 를 통해 객체지향을 지원하는 데코레이터이다. 
`__` 언더바 2개를 붙인 클래스 속성은 클래스 내부에서만 사용된다. 
클래스의 인스턴스를 생성하여 출력하려고 하면 AttributeError를 발생시킨다. 
클래스 내부에서만 사용하여 변수의 변경에 제한을 두고 싶을 때 사용할 수 있다.
개인적으로 느낀 점은 한 클래스 내에서 속성의 값이 변화하는 경우에 사용하면 편리하겠다고 생각했다. 
주문금액 별로 할인율을 적용하고 싶을 때 setter메서드를 사용하여 할인금액을 바로 반환할 수 있다.

```python
class Order:
    def __init__(self, price):
        self.price = price
        self.__price = 0

    @property
    def discount_price(self):
        return self.__price

    @discount_price.setter
    def discount_price(self, order_price):
        if order_price < 10000:
            self.__price *= 0.9
        elif order_price < 50000:
            self.__price *= 0.8
        elif order_price < 100000:
            self.__price *= 0.6
```
