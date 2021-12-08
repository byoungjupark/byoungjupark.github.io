---
title: "async/await, promise"
excerpt: "자바스크립트의  비동기방식 "
---

## async/await
클라이언트 - node.js api - 파이썬 api 의 구조로, 클라이언트의 모든 요청을 node.js 서버에서 받아 각 서비스/기능에 맞는 엔드서버로 재요청을 보낸다. 
node.js에서는 비동기 처리방식이 가능하여 여러 요청을 받아 동시에 수행을 할 수 있다.
<br>

동기방식은 하나의 요청을 받아 완료되기 전까지 다른 요청이 처리되지 못하는 반면, 비동기방식은 요청이 완료되기 전에도 다른 요청을 받아 수행할 수 있다.
파이썬 서버에서 에러가 발생할 경우 아래와 같이 try-catch를 사용하여 에러핸들링이 가능하다고 생각했다. 
하지만 postSignup 함수를 실행한 결과, 1,2를 출력하고 CREATED를 반환하며 종료된다. 
에러핸들링이 진행되는 중임에도 콘솔과 같은 빠르게 처리되는 것들이 먼저 되고 리턴되며 다음 동작을 처리하지 않은 것이다.

```jsx
// client <-> node.js (through http)
function postSignup (req, res) => {
    try{
				console.log('1')
        grpc.signup(req.body.email, req.body.password, req.body.en_name);
				console.log('2')
        return res.status(201).json({'message': 'CREATED'})
    }catch(err){
				console.log('3')
        return res.status(400).json({'message': err})
    }
}
```

node.js의 이러한 특성 덕분에(?!) 회원가입 api를 만드는데에 오래 걸리는 동작의 경우 늦게 완료가 되는데 그 동작이 선행되어야 할 때 흐름을 제어할 필요가 있었다. 
선행되어야 할 함수에 async/await 과 promise를 적용하면 그 동작을 모두 수행한 후 다음 동작으로 넘어갈 수 있다.
<br>

동기적으로 처리할 함수에 대해 `async` 를 걸어준다. 
`await` 은 `async` 내부에서만 사용할 수 있으며, `await` 을 걸어준 함수는 동기적으로 실행되어 완료될 때까지 다음 동작으로 넘어가지 않는다. 
`Promise` 객체를 리턴해준다.

```jsx
// client <-> node.js (through http)
export const postSignup = async (req, res) => {
    try{
        await grpc.signup(req.body.email, req.body.password, req.body.en_name);
        return res.status(201).json({'message': 'CREATED'})
    }catch(err){
        return res.status(400).json({'message': err})
    }
}
```

```jsx
// node.js <-> 파이썬 서버 (through gRPC)
export function signup(email, password, en_name) {
    return new Promise((resolve, reject) => {
        client.CreateStaff(
            {'email':email, 'password':password, 'en_name':en_name},
            (err) => {
                if (err) {
                    reject (err.details)
                } else {
                    resolve ("")
                }
            }
        )
    })
}
```


## Promise 함수를 사용하는 방법 2가지
- new Promise(function(resolve, reject))
- Promise.resolve(function())

Promise는 then()함수를 포함하는 thenable 객체를 반환하여, then()이나 catch()에서 객체를 받아 동작할 수 있다. 
then()함수로 여러번 받아서 동작하는 것을 Promise chainning 이라고 한다.
<br>

Promise안의 함수의 파라미터로 resolve와 reject를 가지는데, 함수가 실행할 때 정상적으로 동작하면 resolve를 에러가 발생할때 reject로 반환시킨다. 
이는 그 다음 동작할 함수, async/await 함수에서 try-catch의 에러로 캐치할 수 있게한다.
<br>

토큰을 검증하는 함수에서 Promise객체로 만들게 되면 아래와 같다. 
토큰이 없다면 reject 로 반환하고, 있다면 resolve안에 반환하고 싶은 값을 넣어준다.

```jsx
export const verifyToken = (req: Request) => {
    return new Promise((resolve, reject) => {
        const token: any = req.headers.authorization;
        if (!token) {
            reject ("NOT_LOGGED_IN")
        } else {
            const uuid = jwt.verify(token, secret_key)
            resolve (uuid)
        }
        }
    )
}
```
reject 반환값은 본함수 try-catch의 에러로 잡히게 된다.

```jsx
export const updatePassword = async (req: Request, res: Response) => {
    try{
        const uuid: any = await verifyToken(req)
        await grpc.updatePassword(uuid.uuid, req.body)
        return res.status(200).json({'message': 'UPDATED'})
    }catch(err){
        return res.status(400).json({'message': err})
    }
}
```

