---
title: "TIL12. Python - sys 모듈과 path"
excerpt: "sys.modules와 sys.path의 차이점, sys모듈 위치 확인하는 방법, Absolute path Relative path, __init__.py 파일의 역할"
---

프로그래밍을 할 때 우리가 직접 모든 변수, 함수, 클래스를 만들지 않아도 된다. 이미 만들어져 있는 모듈과 패키지를 적절히 이용할 수 있다. 디렉토리와 파일의 개수가 많아진다면 한 모듈을 찾기 위해 일일히 클릭해서 들어가야 할까? 비효율적일 것이다. 모듈과 패키지의 위치를 좀 더 쉽게 파악하는 방법이 `sys` 모듈을 사용하는 것이다.

> **sys 모듈 (System-specific parameters and functions)**
파이썬의 내장 모듈(built-in-module)이다. 인터프리터와 상호작용하는 변수와 함수를 직접 제어하고 경로도 제공해준다. 

아래와 같이 `sys` 모듈을 불러보았다. `sys.version`의 결과로 파이썬 인터프리터의 버전 정보가 출력되었다. 이와 같이 `sys`모듈이 인터프리터와 상호작용한다는 것을 알 수 있다.
<img src="https://images.velog.io/images/byoungju1012/post/f092405b-ad77-4e71-9a23-e663e05d53d4/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-10%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.35.55.png" width="600">

## 1. sys.modules 와 sys.path의 차이점
두 속성의 차이점은 import의 유무이다. 어떤 모듈을 import할 때, 인터프리터는 **`1.sys.modules` `2.built-in-modules` `3.sys.path`** 순으로 모듈의 저장 위치를 찾아본다.
`sys.modules` 는 이미 import한 모듈과 패키지를 저장한다. dictionary 형태로 결과값을 돌려준며 key에 모듈 이름이 들어간다. dictionary는 key의 중복이 불가능하기 때문에 동일한 이름의 모듈은 존재하지 않는다. 
![](https://images.velog.io/images/byoungju1012/post/df2358fe-7ee2-4377-8b31-58fc32b35b20/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-10%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.00.43.png)

`sys.path` 는 아직 import 하지 않은 모듈과 패키지들의 경로들을 저장한다. 경로를 list 형태의 결과값으로 돌려준다. 아래 이미지는 `sys.path`의 값을 출력한 것이다. 모듈과 패키지의 절대경로를 보여준다. _(참고. 원래 List로 출력되지만 가독성을 위해 값이 한줄씩 출력되도록 한 것)_ 
`sys.path`는 `append`를 이용해 새로운 모듈과 패키지를 추가할 수 있다. 내가 만든 모듈을 다른 디렉토리에도 사용하고 싶다면 우선 그 경로를 path에 추가한 후 import로 가져올 수 있다.
![](https://images.velog.io/images/byoungju1012/post/85a8fe1e-9856-45fd-af6b-516539ce81d7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-10%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2010.21.56.png)

## 2. sys 모듈 위치 확인하는 방법
`sys` 모듈은 파이썬에 내장되어 모듈이다. `sys.builtin_module_names`를 하면 내장모듈에 `sys` 모듈이 나오는 것을 확인할 수 있다.
![](https://images.velog.io/images/byoungju1012/post/10850abc-78aa-4a33-a450-c81ef4de6e79/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-10%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%209.21.22.png)

## 3. Absolute path vs. Relative path
import하는 모듈과 패키지가 다른 디렉토리에 있다면 경로를 표시해줘야 한다. 경로 표현 방식으로 Absolute path 절대경로와 Relative path 상대경로가 있다. 

절대경로는 디렉토리의 경로를 최상위부터 모두 표시해준다. 사용자의 현재 디렉토리 위치와 무관하다. 경로 파악을 빠르게 할 수 있다. 하지만 여러 개의 하위 디렉토리를 가진다면 경로의 길이가 길어진다는 단점이 있다.
```python
from package2.subpackage1.module5 import function2
# package2 디렉토리 -> subpackage1 디렉토리 -> module5 모듈의 function2 함수를 import함
```

상대경로는 현재 디렉토리를 기준으로 나타낸다. `.`dot을 이용하는데, `.` dot 1개는 현재 디렉토리의 위치를 의미한다. `..` dot 2개는 상위 디렉토리를 의미한다. 절대경로에 비해 깔끔하지만 경로를 헷갈리기 쉽다.
```python
# package2/module3.py 현재 package2의 module3 모듈에 위치함

from .subpackage1.module5 import function2 
# 현재 디렉토리의 하위 디렉토리 subpackage1 -> module5 모듈의 function2 함수를 import 함
```

## 4. calculator 패키지 만들기

### 4-1. main.py에서 상대경로로 add_and_mutiply 를 임포트 했을 때 발생하는 에러는? main module 에서 패키지의 모듈 임포트하는 법
`ImportError`가 발생한다. 파이썬에서 메인 모듈은 항상 절대경로를 이용하여 import 해야 한다. 메인 모듈은 말 그대로 패키지의 메인 파일이며 프로그램의 시작점을 말한다. 메인 모듈에는 통상 `if __name__ == "__main__"` 코드를 넣어준다. 이 if문은 메인 모듈이 메인 프로그램으로 실행될 때와 다른 모듈에서 import한 단순 모듈일 때를 구분하기 위해 사용할 수 있다.

![](https://images.velog.io/images/byoungju1012/post/e36d0ad3-3aab-4140-b940-dea18d52af1b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-10%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2011.14.08.png)

main.py는 이 디렉토리의 메인 모듈인데 아래와 같이 상대경로로 import 하였기 때문에 import error가 발생한 것이다. 절대경로로 모듈을 불러와야 한다.
```python
from calculator.add_and_multiply import add_and_multiply
```

### 4-2. add_and_multiply.py에서 multiply함수를 절대경로와 상대경로도 각각 임포트 해보고 main 모듈과 차이점을 생각해보고 결과를 출력해보기
절대경로와 상대경로 모두 정상적으로 값이 출력될 것이라 예상했는데, 둘 다 에러가 발생했다. 정상 출력이라 생각했던 이유는 add_and_multiply.py가 메인 모듈이 아니기 때문이었다.

절대경로는 `ModuleNotFoundError`가 발생했다.
<img src="https://images.velog.io/images/byoungju1012/post/59114d39-a84a-438b-aa61-4b5719a6c221/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-11%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.12.47.png">

상대경로는 `ImportError`가 발생했다.
<img src="https://images.velog.io/images/byoungju1012/post/49e18c64-d29f-450d-866a-6bc3c7058c61/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-11%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.13.18.png">

에러의 종류는 다르지만,calculator 패키지가 없다는 것으로 원인은 같다. 
1. 오타 확인 : 오타 없음
2. 패키지로 되어있는지 확인 : `sys.path` 안에 calculator 패키지는 잘 들어가 있다. 

`add_and_multiply.py`는 `multiply` 모듈과 같은 경로에 있으므로 부모 경로인 `calculator` 패키지를 경로에 넣어주면 안된다. `from multiplication import multiply`로 하면 정상 출력된다.

## 5. `__init__.py` 파일의 역할
`__init__.py`는 이 파일이 포함하고 있는 디렉토리가 패키지로 인식되도록 알려주는 역할이다. 내용을 비워둘 수 있지만, 이 파일을 사용하여 import할 경로를 짧게 나타낼 수 있다. 지금까지는 패키지의 모듈을 불러오려면 절대경로나 상대경로를 이용해서 `import 패키지.모듈`형태로 사용했다. `__init__.py`파일에 넣고 싶은 패키지와 모듈을 넣는다면 `import 패키지` 형태로 간단하게 가져올 수 있다. 
