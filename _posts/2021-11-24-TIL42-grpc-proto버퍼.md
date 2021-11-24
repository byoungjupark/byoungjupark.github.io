---
title: "TIL42. gRPC proto버퍼"
excerpt: "gRPC란, proto버퍼 설정"
---

## RPC
Remote Procedure Call는 원격 프로시저 호출이란 뜻으로, 별도의 원격제어를 위한 코딩없이 함수나 프로시저를 실행할 수 있게 해주는 프로세스간 통신이다.
위치에 상관없이 원하는 함수를 사용할 수 있다.
서로 다른 언어 MSA구조로 서비스를 만들면 다른 언어와 프레임워크로 개발되는 경우가 많다. 
이러한 Polyglot한 구조에서 RPC를 사용하면 언어에 구애받지 않고 원격에 있는 프로시저를 호출할 수 있다.

## gRPC
gRPC는 구글에서 만든 rpc(Remote Procedure Call)이다.
고성능이며 경량으로 xml이나 json에 비해 네트워크에 전송되는 양이 적다. 
HTTP2.0 프로토콜을 사용한다.
IDL(Interface Definition Language)로 프로토콜 버퍼를 사용한다. 
프로토콜을 정의하여 클라이언트와 서버간의 통신을 어떻게 할 것인지 미리 설정해놓을 수 있다.(일종의 스키마파일)

## gRPC proto 버퍼 설정
grpc는 homi 라이브러리를 사용해 서버를 실행시켰다. 아래링크 참조
<br>
https://github.com/spaceone-dev/homi/tree/master/example/helloworld

프로토버퍼에서 인터페이스를 설정하며, 함수의 구현체는 따로 만든다.
원격에 있는 함수 singup은 request 객체로 email, password를 받고, response 객체로 message, status_code를 전송한다.

크게 service 와 message로 나눈다.
service 부분에는 서버가 사용할 함수를 적어준다. 
message에는 클라이언트와 연결할 때 request와 response의 key값과 타입을 설정해준다. service에서 전달할 데이터의 직렬화 포맷을 정의한다.

```protobuf
service Account{
// rpc [함수] (요청 메세지) returns (응답 메세지)
  rpc signup (CreateAccountRequest) returns (CreateAccountResponse) {}
}

// 요청 메세지
message CreateAccountRequest {
  string email = 1;
  string password = 2;
}

// 응답 메세지
message CreateAccountResponse {
  string message = 1;
  int32 status_code = 2;
}
```

회원가입 시 들어올 request를 email과 password라고 가정하고 각각 문자열 타입으로 지정해준다. 
들어올 index 값은 각각 1,2 순서대로이다.
signup 함수를 실행 후 response할 key값은 message와 status_code로 문자열과 정수형으로 설정한다.

프로토버퍼를 만든 후 `protoc` 컴파일러를 사용하면 구현할 언어의 클래스나 메서드로 변환시켜준다.
컴파일을 위해 `python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. [프로토파일 이름].proto` 
명령어를 입력하면, `account_pb2.py`, `account_pb2_grpc.py` 파일을 생성해준다.
- `account_pb2.py` : 요청과 응답을 위한 프로토버퍼 코드가 있다. 
- `accout_pb2_grpc.py` : 서비스에 정의된 메서드를 사용하기 위한 인터페이스가 있다.

