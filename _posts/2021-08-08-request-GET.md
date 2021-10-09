---
title: "TIL28. request.GET"
excerpt: "Python의 get 메소드, Django의 GET 객체"
---

### 1. request.get
request 객체에 대해 get 메소드를 실행하는 것을 말한다.
클라이언트의 요청을 받고 장고는 HttpRequest 객체를 만들어 해당 View 메소드의 첫번째 인자로 전달해준다. request객체는 딕셔너리 형태로 되어있다.

### 2. request.GET
HTTP의 GET 메소드로 보내온 request 일 경우, `request.GET`을 통해 url의 파라미터 정보를 얻을 수 있다.
request.POST 라면, HTTP POST 메소드의 request일 때의 요청 정보를 받아올 수 있을 것이다.
(POST 메소드는 보통 body에 요청한 내용이 담겨져 있는데, 이 부분을 보여주는지 url 파라미터 정보를 보여주는지는 추후 확인해봐야겠다.)

HttpRequest 객체에서 GET과 POST 속성은 장고의 QueryDict 인스턴스이다.  

```python
QueryDict('a=1&a=2&c=3')
<QueryDict: {'a':['1', '2'], 'c':['3']}>
```
