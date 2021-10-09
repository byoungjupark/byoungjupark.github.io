---
title: "TIL24.  westagram - User, Login"
excerpt: "회원가입과 로그인 API, Decorator와 토큰"
---

## 1. User model

```python
# users/models.py
class User(models.Model):
	name          = models.CharField(max_length=45)
	email         = models.CharField(max_length=200)
	password      = models.CharField(max_length=200)
	phone_number  = models.CharField(max_length=45)
	birth_date    = models.DateField()

    class Meta:
        db_table  = "users"
```
비밀번호는 암호화된 문자열이 저장되기 때문에 최대 길이를 200으로 설정했다.
전화번호를 `InteagerField`로 지정하면 0으로 시작하는 번호는 0이 무시되기 때문에 `CharField`로 설정한다.

## 2. User view

### 회원가입
```python
# users/views.py
class UserView(View):
    def post(self, request):
        data = json.loads(request.body)

        try:
            #이메일과 비밀번호는 지정한 형식에 맞는지 확인
            if (re.match("\w*@\w*.\w*", data["email"]) is None):
                return JsonResponse({"message":"INVALID_EMAIL"}, status = 400)

            if (re.match("[\w\W{8,20}]", data["password"]) is None):
                return JsonResponse({"message":"INVALID_PASSWORD"}, status = 400)  
            
            #이메일이 중복되는지
            if User.objects.filter(email = data["email"]).exists():
                return JsonResponse({"message":"EXIST_EMAIL"}, status = 400)

	    #비밀번호 암호화
            hashed_password = bcrypt.hashpw(data["password"].encode("utf-8"), bcrypt.gensalt())

            User.objects.create(
                name         = data["name"],
                email        = data["email"],
                password     = hashed_password.decode("utf-8"), #비밀번호는 db에 문자열로 저장
                phone_number = data["phone_number"],
                birth_date   = data["birth_date"]        
            )

            return JsonResponse({"message":"SUCCESS"}, status = 201)

        except KeyError:
            return JsonResponse({"message":"KEY_ERROR"}, status = 400)
```
* 이메일과 비밀번호 : 정규식 표현을 사용하여 지정한 형식에 맞는지 매치시킬 수 있다. 
`if (re.match("\w*@\w*.\w*", data["email"]) is None):`
`if (re.match("[\w\W{8,20}]", data["password"]) is None):`
커스텀한 표현식으로 작성했지만, 이메일과 비밀번호를 표현하는 방식이 비슷하기 때문에 이미 검증되어 정형화된 표현식을 사용하는 것이 좋을 듯하다.
조건문에서 괄호를 빼도 같은 결과값이 나오므로, 불필요한 괄호는 생략하는 것이 좋겠다.
`re.matach`는 정규식에 부합하면 match객체를 반환하고, 아니라면 None을 반환한다. 반환값 자체가 True, False 성질을 가지기 때문에 불필요한 `~is None`을 생략하고 `if not`문으로 쓰는 것이 바람직하다.

* 비밀번호 암호화 : 암호화에 대중적인 라이브러리 `bcyrpt` 를 사용할 수 있다. 컴퓨터가 이해할 수 있는`bytes` 타입으로 전환시킨 후, 암호화를 한다. 이후 암호화된 비밀번호를 db에는 다시 `string` 타입으로 전환하여 저장시킨다. 디코딩하지 않으면 db에 b' 표시도 함께 문자열로 저장된다. 추후 로그인 시 비교할 비밀번호와 다를 수 밖에 없다.

* Key Error : db 필드에 해당하는 key 자체가 안 들어올 수 있다. 발생할 수 있는 에러들을 try, except문으로 예외처리하여 어떤 에러인지 명확히 알 수 있다.

## 3. LoginView
### 로그인
```python
# users/views.py
class LoginView(View):
    def post(sef, request):
        data = json.loads(request.body)

        try:
            # 이메일, 비밀번호를 입력하지 않은 경우
            if data["email"] == "" or data["password"] == "":
                return JsonResponse ({"message":"INVALID_USER"}, status = 401)
	    
            # db에 존재하는 이메일이 맞는지
            if not User.objects.filter(email = data["email"]).exists():
                return JsonResponse({"message":"INVALID_USER"}, status = 401)
	    
            # 존재하는 이메일이라면 해당 User객체 user로 할당
            user = User.objects.get(email = data["email"])
            
            # 비밀번호가 user의 비밀번호랑 일치하는지
            if not bcrypt.checkpw(data["password"].encode("utf-8"), user.password.encode("utf-8")):
                return JsonResponse({"message":"INVALID_USER"}, status = 401)
	    
            # 이메일과 비밀번호 모두 일치 : 토큰 생성하여 보내주기
            access_token = jwt.encode({"id":user.id}, SECRET_KEY, algorithm="HS256")

            return JsonResponse({"token":access_token}, status = 200)

        except KeyError:
            return JsonResponse({"message":"KEY_ERROR"}, status = 400)
```
* 비밀번호 : 비밀번호는 다른 유저의 비밀번호와 같을 수 있다. 유저의 이메일이 검증되었다면, 요청한 비밀번호가 **그 유저의 비밀번호**와 일치하는지를 비교해야한다. 일치 확인도 `bcrypt` 메소드를 사용한다. 현재 비밀번호는 `string` 타입이기 때문에 다시 인코딩하여 비교하도록 한다.

* 토큰 : 이메일과 비밀번호 모두 일치하면 토큰을 생성하여 보내준다. 토큰 생성은 `Json Web Token` 라이브러리를 사용한다. 해당 유저의 id값을 담는다. 이메일도 중복될 수 없기 때문에 가능하지만, id가 고유값이고 용량도 적기 때문에 id로 구분하는 것이 좋을 듯 하다. 추후 유저가 토큰을 보낼 때 이 id값으로 해당 유저인지 확인할 수 있다.

## 4. Decorator를 이용한 토큰 검증

```python
# users/utils.py
def is_user(func):
    def wrapper(self, request, *args, **kwargs):        
        try:
            token 	 = request.headers.get("Authorization") [1]
            payload      = jwt.decode(token, SECRET_KEY, algorithms="HS256") # 인코딩 및 해싱된 토큰을 디코딩해준다
            request.user = User.objects.get(id = payload["id"]) [2]
                        
            if payload: # 토큰 값이 있다면 데코레이터 다음의 본함수를 호출한다.
                return func(self, request, *args, **kwargs)

        except KeyError:
            return JsonResponse({"message":"KEY_ERROR"}, status = 401)
        
        except jwt.DecodeError: # 토큰없이 보낸 경우, 토큰 id 값이 db에 없는 경우
            return JsonResponse({"message":"INVALID_USER"}, status = 401)
        
    return wrapper
```
로그인에 성공한 유저가 토큰을 보내왔다. 인스타그램은 로그인 후에 대부분의 기능을 이용할 수 있기 때문에 매번 토큰을 확인하기보다는 데코레이터를 사용하는 것이 효율적이다.

[1] 토큰은 request의 Header 중 Authoriaztion에 담겨서 온다. Authorization 값이 있다면 그 값을, 없다면 None을 반환한다. 

[2] User 객체 중 토큰의 id 값과 일치하는 객체를 `request.user`로 할당한다.
> **장고의 HttpRequest 객체**
django.http 모듈의 `HttpRequest`, `HttpResponse` 객체가 있다. 
View에서 항상 넣어준 request가 사실은 HttpRequest 객체이다. ** 장고는 View 메소드의 1번째 인자로 HttpRequest 객체를 넣는다.** (self를 제외한)
장고는 request를 받을 때마다 관련된 middelware를 거쳐 HttpRequest 객체를 생성한다. 필요에 따라 미들웨어에서 이 객체를 가공하고 속성을 만들어준다.
`HttpRequest.user`는 Authentication 미들웨어를 통해 설정된 속성이다. 


<div><img src="https://images.velog.io/images/byoungju1012/post/ada04bfa-e746-435f-bd6b-6e866715272d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-07-31%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.28.35.png"></div>

아직 정확하지는 않지만 토큰이 Authorization에 담겨 전송되었기 때문에, HttpRequest 객체가 Authorization 미들웨어를 거쳐 user 속성이 설정될 수 있던 것이 아닐까 한다.

참조
https://supplementary.tistory.com/255
https://docs.djangoproject.com/ko/3.2/ref/request-response/
