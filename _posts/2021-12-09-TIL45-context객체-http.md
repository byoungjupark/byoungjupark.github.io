---
title: "TIL 45.context 객체와 http1.1, 2.0의 차이"
excerpt: "context객체란, http1.1과 2.0의 차이"
---

## context 객체
애플리케이션(객체)의 현재 상태의 맥락(context)를 의미한다.
컨텍스트는 새로 생성된 객체가 지금 어떤 일이 일어나고 있는지 알 수 있도록 하는 역할
<br>

추상클래스, rpc와 관련된 정보와 행동들을 제공하는 역할

```
class _Context(grpc.ServicerContext):

    def __init__(self, rpc_event, state, request_deserializer):
        self._rpc_event = rpc_event
        self._state = state
        self._request_deserializer = request_deserializer

		def set_code(self, code):
		        with self._state.condition:
		            self._state.code = code
		def set_details(self, details):
		        with self._state.condition:
		            self._state.details = _common.encode(details)
```

```
from grpc._server import _Context

def set_exception(context: _Context, error):
    if error == InvalidArgument:
        context.set_code(grpc.StatusCode.INVALID_ARGUMENT)
        context.set_details("INVALID_ARGUMENT")
```

## HTTP1.1과 2.0의 차이
HTTP1.1은 기본적으로 connection 1개당 1개의 요청을 처리하도록 설계되어 있다.
동시에 여러개의 리소스를 주고받는 것이 불가능하고, 요청과 응답이 순차적으로 발생한다.
리소스(image, css, script)를 처리할 때 리소스 개수에 비례하여 대기시간이 길어진다.

### HTTP1.1의 단점
- 무거운 header 구조 : 헤더에 많은 메타정보 저장, 중복된 헤더값을 전송
- RTT(Round Trip Time, *패킷 왕복시간) 증가 : 패킷이 목적지 도달 후 응답이 다시 출발지로 돌아오기까지의 시간 / connection 1개당 1개의 요청 처리 / 3-way-handshake 반복적 발생
- HOL(Head Of Line) Blocking 특정 응답의 지연 : 네트워크에서 같은 큐에 있는 패킷이 첫번째 패킷에 의해 지연될 때 발생하는 성능 저하 현상
** 패킷(Packet) : pack+bucket / 네트워크를 통해서 전송되는 데이터 조각

### HTTP2.0이 빠른 이유
- Multiplexed streams : 한 커넥션에 여러개의 메세지 동시에 주고받기 가능
- HOL Blocking 발생하지 않는다.
- Server push : 클라이언트가 요청하지 않은 리소스를 사전에 푸쉬를 통해 전송 가능 → 클라이언트 요청 최소화
- Header Compression : 헤더정보 압축


