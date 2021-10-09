---
title: "DashInsert 함수 만들기"
excerpt: "파이썬 enumerate를 사용한 알고리즘 "
---

### 1. 문제
DashInsert 함수는 숫자로 구성된 문자열을 입력받은 뒤 문자열 안에서 홀수가 연속되면 두 수 사이에 -를 추가하고, 짝수가 연속되면 *를 추가하는 기능을 갖고 있다. DashInsert 함수를 완성하시오.

>입력 예시: 4546793
> 출력 예시: 454*67-9-3

### 2. 나의 풀이
```python
def DashInsert():
    num = input("숫자를 입력해주세요: ")
    lst = list(num)
    for i in lst:
        if int(i) % 2 == 1 and int(lst[lst.index(i)+1]) % 2 == 1:
            lst.insert(lst.index(i)+1, "-")
        elif int(i) % 2 == 0 and int(lst[lst.index(i)+1]) % 2 == 0:
            lst.insert(lst.index(i)+1, "*")
        else: 
            pass
    for j in lst:
        print(j,end="")
    
DashInsert()
```
#### 고려한 점
1. input 함수 출력 결과는 문자형이기 때문에 수식에 포함할 때에는 int로 정수 변환을 해야 한다.
2. list 함수로 리스트 형태로 변환시킨 다음, for문으로 리스트 값 하나하나를 돌린다.
3. for문 안에 연속 홀수인지, 연속 짝수인지를 판단하는 if문을 작성한다.
4. 업데이트된 리스트를 다른 for문을 사용해 공백없는 일렬 형태로 출력한다.

#### 코드 해석
* def DashInsert():
		num = input("숫자를 입력해주세요: ")
        lst = list(num)
먼저 DashInsert 함수를 만들었다.
사용자의 입력을 받는 input 함수를 넣어 새로운 변수 num으로 선언하였다.
num은 list 함수로 리스트 형태로 변환시켰다. 
이제 처음 입력한 문자형태의 숫자들은 각각 리스트의 값으로 나누어진 상태이다.

* for i in lst:
리스트의 값들을 순서대로 불러와서,

* if int(i) % 2 == 1 and int(lst[lst.index(i)+1]) % 2 == 1: 
현재 값(i)과 다음 값에 각각 2로 나누었을 때 나머지가 1이면, (= 값이 홀수이면,)
	- 다음 값을 어떻게 인식할까? 
    리스트의 인덱싱를 사용했다. 현재 인덱스 lst.index(i)에 +1을 하면 다음 인덱스가 된다. 다음 인덱스의 값은 lst[lst.index(i)+1] 로 나타냈다.
    - 리스트의 값들은 문자 형태이기 때문에, 현재 값과 다음 값을 각각 int로 감싸주어 정수형으로 나타냈다.

* lst.insert(lst.index(i)+1, "-")
다음 인덱스 값에 -를 삽입해준다.

* elif int(i) % 2 == 0 and int(lst[lst.index(i)+1]) % 2 == 0:
lst.insert(lst.index(i)+1, "*")
나머지 값을 0으로 바꾸면 값이 짝수일 때를 나타낸다.

* else : pass
연속 홀수이거나 연속 짝수가 아니면(홀수와 짝수일 때) pass하여 다음 단계를 진행시켰다.

* for j in lst: print(j,end="")
다른 for 문을 사용하여, 추가된 값들로 재정의된 리스트를 공백없이 한줄로 출력시켰다.

#### 출력 결과 및 간과한 점
하지만 출력 결과는 다음과 같이 ValueError로 떴다.
![](https://images.velog.io/images/byoungju1012/post/abdbe4a1-716d-4392-a4ff-201f614f6070/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-06-18%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%203.00.56.png)

for문이 반복되는 동안, 현재 인덱스의 다음 번호에 -이나 \*를 추가하면 다음 인덱스는 기존 숫자가 아닌 -나 \*이 된다. if문의 조건에 리스트의 값을 int함수를 사용해 정수화하는 것으로 나타냈다. 하지만 -나 \*이 들어가는 순간 형타입오류(ValueError)가 발생하는 것이다.

### 3. 모범 답안
```python
data = "4546793"

numbers = list(map(int,data)) #숫자 문자열을 숫자 리스트로 변경
result = []

for i, num in enumerate(numbers):
	result.append(str(num))
    if i < len(numbers)-1:			#다음 수가 있다면
    	is_odd = num % 2 == 1 			#현재 수가 홀수
        is_next_odd = numbers[i+1] % 2 == 1	#다음 수가 홀수
        if is_odd and is_next_odd:		#연속 홀수
        	result.append("-")
        elif not is_odd and not is_next_odd:	#연속 짝수
        	result.append("*")
            
print("".join(result))
```
* data = "4546793" 
입력받을 숫자를 data 변수로 만들어준다.

* numbers = list(map(int,data)) 
map함수에 int함수를 넣어 문자열을 각각의 숫자형으로 전환시킨다. 그리고 리스트로 만들어준다.

* for i, num in enumerate(numbers): result.append(str(num))
> enumerate 
인덱스와 값을 돌려주는 파이썬 내장함수이다. 인덱스를 출력하기 때문에, 이 함수의 입력값은 순서가 있는 리스트나 튜플 등이 들어간다.

 i는 인덱스, num은 리스트의 값을 받을 것이다.
result 리스트에 numbers 리스트값을 문자형으로 추가시킨다.

* if i < len(numbers)-1:
    	is_odd = num % 2 == 1
        is_next_odd = numbers[i+1] % 2 == 1
        인덱스 번호를 나타내는 i는 0부터 시작하고, len함수는 1을 기준으로 문자길이를 세기 때문에 len에 1을 뺀 값을 넣어준다.
        is_odd로 현재 값이 홀수인지를,
        is_next_odd로 다음 값이 홀수인지를 나타내는 변수 2개를 만든다.

* if is_odd and is next_odd: result.append("-")
elif not is_odd and not is_next_odd: result.append("*")
is_odd와 is_next_odd 둘다 참(홀수)이면 result리스트에 -를 추가한다.
is_odd와 is_next_odd 둘다 거짓(짝수)이면 result리스트에 *를 추가한다.

* print("".join(result))
result 리스트를 공백없이 한줄로 출력시킨다.
