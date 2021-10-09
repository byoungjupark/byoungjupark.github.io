
---
title: "TIL13. iterator와 generator"
excerpt: "return과 yield의 차이점, generator expression, list comprehension, lazy evaluation"

---

## 1. iterator 이터레이터
이터레이터는 iterable한(반복가능한) 데이터를 순차적으로 꺼내올 수 있는 객체를 말한다. 이터레이터는 next() 또는 `__next__` 메소드를 사용해서 하나씩 꺼내올 수 있다. 반복가능한 데이터로는 list, string, dictionary 등이 있다.

이 자료형들은 반복가능한 특성을 지녔지만 이터레이터라고 할 수는 없다. 따라서 next()메소드를 사용할 수 없다. iter() 또는 `__iter__` 메소드를 통해 이터레이터로 만들어줘야 한다. 이터레이터 출력이 종료된 뒤에는 StopIteration이 발생한다. 더 이상 반복할 것이 없다는 뜻이다.
```python
lst = [1,2,3] 
print(type(lst)) # list 출력
next(lst) # TypeError 발생

I = iter(lst) # 이터레이터 객체로 만들어준다
print(type(I)) # listiterator 출력
next(I) # 1 출력
next(I) # 2 출력
next(I) # 3 출력
next(I) # StopIteration 발생
```

## 2. generator 제너레이터
제너레이터는 이터레이터를 생성하는 객체이다. 
yield 키워드는 generator 함수에서만 사용할 수 있다. 

### 2-1 return 과 yield 의 차이점
return은 함수 실행을 종료한 후 결과값을 반환한다.
yield는 함수 내에서 yield를 만나면 우선 일시정지 후 yield 값을 next()메소드에 전달한다. iteration이 소진될 때까지 yield -> next() 과정을 반복한 후 StopIteration이 발생하면 함수가 종료된다.

```python
import time
L = [1,2,3]

def print_iter(iter):
    for element in iter:
        print(element)

def lazy_return(num):
    print("sleep 1s")
    time.sleep(1)
    return num

print("comprehension_list=")
comprehension_list = [lazy_return(i) for i in L]
print_iter(comprehension_list)

print("generator_exp=")
generator_exp = (lazy_return(i) for i in L)
print_iter(generator_exp)
```
<함수 실행 결과>
![](https://images.velog.io/images/byoungju1012/post/b1b4a3a1-7f3a-452f-baf9-dc4d45d074ae/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-12%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%202.17.42.png)

### 2-2 generator expression 제너레이터 표현식 vs list comprehension 리스트 컴프리헨션

두 방식은 반환값을 보내는 시점이 다르다. 리스트 컴프리헨션 방식은 함수 종료후 반환하는 반면, 제너레이터 표현식은 함수를 진행하면서 반환한다.

리스트 컴프리헨션 방식은 함수 실행을 종료한 뒤 return 값을 돌려준다. 
즉, [lazy_return(i) for i in L] 의 lazy_return 함수를 실행하면서 sleep 1s가 3번 출력된다. 
함수 종료 후 return 값인 [1,2,3] 이comprehension_list 객체로 할당된다.
이후 print_iter 함수를 실행하면서 comprehension_list 값들이 차례대로 출력된다.

제너레이터 표현식은 함수를 실행하면서 첫 yield를 만날 때 그 값을 밖으로 전달한다. 전달 후 다시 함수로 돌아와 다음 yiedl를 만나고 두번째 값을 밖으로 전달한다. 이 과정을 반복이 종료될 때까지 진행한다. 
(lazy_return(i) for i in L)의 lazy_return 함수 실행 시  "sleep 1s"를 출력 후 1초 뒤에 L의 첫번째 값 1을 print_iter 함수에게 반환한다. 
print_iter 는 이 값을 저장 후 출력한다. 이후 lazy_return 함수로 돌아와 위 과정을 반복한다.

### 2-3 lazy evaluation
이와 같이 제너레이터는 함수 실행을 지연시킬 수 있고, 이를 lazy evaluation 이라고 한다. 메모리를 절약하는데 활용할 수 있고, 코드 실행을 지연시키고 싶을 때 사용할 수 있다.
