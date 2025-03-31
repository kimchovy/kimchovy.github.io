---
title: 작성중 / 서버와 클라이언트의 요청과 응답 구조(HTTP)
description: >-
  HTTP 요청과 응답 메시지의 헤더와 바디 구조, 쿠키, 쿼리 스트링, 패스 파라미터 등을 이해해보자.
date: 2025-03-29 02:46:00 +0900
categories: [CS, Network]
tags: [HTTP, Web, Network, Header, Body, Cookie, Parameter]
pin: false
---

## 들어가기에 앞서 1: HTTP 요청, 응답의 기본 개념

- 웹 브라우저(클라이언트)는 `사용자의 요청`을 `서버`로 보내고, 서버는 그 `요청을 처리`한 후 `응답을 반환`합니다. 
    > 이를 `HTTP 요청(Request)`과 `응답(Response)`이라고 합니다.
- 비유하자면, 식당에서 음식을 주문하는 과정과 비슷합니다.
    1. 고객(클라이언트)이 메뉴판을 보고 원하는 음식을 주문하면(요청)
    2. 주방(서버)은 그 주문을 받아 요리를 한 뒤(처리)
    3. 고객에게 음식을 내어줍니다(응답)


## 들어가기에 앞서 2: HTTP의 무상태성(Stateless)과 비연결성(Connectionless)

### 1. 무상태성(Stateless)

- HTTP 프로토콜은 일반적으로 `Stateless` 상태를 유지합니다.
    - 이는 서버가 클라이언트의 상태를 `보존하지 않는 성질`을 뜻합니다.
    > `각각의 요청/응답은 독립적으로 작동`하며, 서버는 이전에 클라이언트와 어떠한 요청/응답이 있었는지 알지 못한다는 의미입니다.
- Stateless의 장점 : 서버 장애가 발생하더라도, `응답 서버를 교체`하기 수월하며 `서버의 확장(Scale-out)`에 용이하다.
- Stateless의 단점 1: Stateful에 비해 `전송해야 하는 데이터의 양이 많다`.
> 서버에 클라이언트와의 `세션(Session)` `상태`, `정보`를 저장하는 `Stateful(대표적으로 TCP 프로토콜)` 방식과 달리 앞전의 요청/응답에 대한 기록이 남지않는 Stateless의 특징상 `매 요청마다 상태 정보를 전달`해야 하기에 전송하는 데이터의 양이 많아지는 것이다.
- Stateless의 단점 2: 흔히 `무상태(Stateless)의 한계`라고 불리는 로그인과 같이 상태가 유지되어야 하는 경우, `브라우저 쿠키`, `서버 세션`, `토큰` 등을 이용해 상태를 유지해야 한다.
> 그리고 이러한 상태유지는 서버의 자원에도 한계가 있기에 `최소한만 사용`하는 것이 좋다.
>> 캐시 서버로는 `빠른 액세스 속도`와 `휘발성`의 특징을 가진 `NoSQL`데이터베이스(Database)인 `Redis`를 주로 사용한다

### 2. 비연결성(Connectionless)

- HTTP는 기본적으로 `연결을 유지하지 않는(Connectionless)` 단방향 통신 프로토콜이다.
> 반대로 연결지향적 양방향 통신으로는 `TCP`가 있다.
- 근데 여기서 중요한 사실은 `HTTP통신도 기본적으로 TCP(양방향통신) 위에서 작동이 되는 구조`라는 것이다.(???)
> 정확히 얘기하자면 HTTP는 `단기적인 TCP 커넥션` 후, 요청에 따른 응답을 받으면 `연결이 끊어지고(Connectionless)`, `어떠한 상태도 남기지 않는(Stateless)` 것이다.
>> 간단한 비유를 들자면, 내가 클라이언트, 그리고 짝사랑중인 여사친 또는 남사친을 서버라고 가정하고 카톡으로 `좋아한다는 고백(요청 : 연애)`을 했는데 `'좋은 친구로 지내자'라는 답변(응답 : 거절)`을 받은뒤 `연락이 끊기고(Connectionless)`, 고백받은 이(서버)가 해당 카톡방을 나가면서 `앞전 대화 내용이 사라지며 까먹은 상태(Stateless)`가 되는 것이라고 볼 수 있다.
>>> 어디까지나 비유일뿐 정확한 예시는 아닐수 있다.
- `HTTP 1.0` 에서는 각각의 자원(ex. html, js, image 등등..)을 다운로드하기 위해 연결과 종료를 반복했으나, `HTTP 1.1` 버전 부터는 지속 연결(Persistent Connection)[^persconn] 기능이 도입되어 여러 HTTP 요청/응답을 한번의 연결로 처리할 수 있게 되었다.
> 이로 인해 TCP 연결/해제에 따른 오버헤드(overhead)[^overhead]를 줄일 수 있다.


`그럼 이제 요청과 응답의 구조를 하나씩 살펴볼까요?`

------

## HTTP 요청 메시지 구조

- HTTP 요청 메시지는 크게 세 부분으로 구성됩니다.

1. `start-line`
2. `headers`
3. `body`

### 1. 시작 라인(Start Line)

- 시작 라인 또한 세 부분으로 구성되어 있다.
    - `HTTP method / Request target / HTTP version`
    > 한글로 풀어서 설명하자면 대략, `어떤 행동을 할 것인지` / `어떤 리소스를 요청하는지` / `어떤 버전의 HTTP로 응답받을 것인지` 정도가 됩니다.

- 예시: 
    - GET /index.html HTTP/1.1
        > GET : 요청 메서드(`POST`, `PUT`, `DELETE` 등)

        > /index.html : 요청 대상(`URI` path와 GET 메서드의 경우 절대경로 뒤에 '?'로 시작하는 `쿼리 스트링`도 들어갈 수 있습니다)

        >HTTP/1.1 : HTTP 버전(`1.0`, `1.1`, `2`, `3`)
        >> `0.9버전`으로 불리는 one-line protocol인 HTTP 초기 버전도 있긴 있다.~~(현재 쓰이는 곳 아시는분은 제보 바랍니다)~~


### 2. 요청 헤더(Request Headers)

- 요청 헤더는 요청에 대한 추가 정보를 담고 있다.
> 메시지 크기, 압축 여부, 인증(Authentication), 브라우저 정보, 서버 정보, 캐시 등
    - key:value 형식을 따릅니다.
- 예시:
    - Host: www.example.com
    > Host : 요청을 보낼 서버의 주소
    - User-Agent: Mozilla/5.0
    > User-Agent : 요청을 보낸 브라우저 또는 프로그램 정보
    - Content-Type: application/json
    > Content-Type : 보내는 데이터의 타입 (JSON, HTML 등)
    >> GET 메서드는 무조건 URL 끝에 value=text 형식의 쿼리가 붙어서 보내지기에 Content-type을 따로 지정해줄 필요가 없다고 한다.
    >>> 즉, POST나 PUT처럼 body에 데이터를 보낼때 필요로 한다.
- 요청 헤더와 바디를 구분하기 위해 헤더가 모두 끝나는 지점에는 공백라인이 들어간다.

### 3. 요청 바디(Request Body)

- POST 요청처럼 데이터를 함께 보낼 때 요청 바디[^body]를 사용합니다.
> 요청, 응답 구분없이 바디 부분에는 byte로 표현 가능한 모든 데이터가 들어갈 수 있습니다.
>> 데이터 예시: html 문서, 이미지, 영상, JSON 등

```json
{
  "username": "kimchovy",
  "password": "1234",
  "bank_account_balance": 0,
  "stay_up_all_night": true
}
```
```html
<html>
    <body>
        <h1>This is my awesome blog!</h1>
    </body>
</html>
```
```image
<html>
    <body>
        <span>...</span>
    </body>
</html>
```

------
## HTTP 응답 메시지 구조

- 서버가 요청을 처리한 후 응답을 보낼 때도 요청과 비슷한 구조를 가집니다.

### 1. 상태 라인(Status Line)
HTTP/1.1 200 OK
HTTP/1.1 : HTTP 버전

200 : 상태 코드 (요청이 성공했음을 의미)

OK : 상태 메시지

### 2. 응답 헤더(Response Headers)
응답 헤더도 요청 헤더처럼 추가 정보를 담고 있어요.

Content-Type: text/html
Set-Cookie: session_id=abcd1234; Path=/; HttpOnly
Content-Type : 응답 데이터의 타입 (HTML, JSON 등)

Set-Cookie : 클라이언트에 저장할 쿠키 정보

### 3. 응답 바디(Response Body)
응답 바디에는 실제 응답 데이터가 포함됩니다.

<html>
  <body>
    <h1>Welcome to my website!</h1>
  </body>
</html>

------
## 요청과 응답에서 자주 쓰이는 파라미터들

### 쿼리 스트링(Query String)
URL 뒤에 ?를 붙여 키-값 쌍으로 데이터를 전달

예: https://example.com/search?query=jekyll&sort=recent

사용 예: 검색어 입력, 필터 옵션 전달

### 패스 파라미터(Path Parameter)
URL 경로 일부로 데이터를 전달

예: https://example.com/users/123

사용 예: 특정 사용자 정보 요청

### 쿠키(Cookie)
서버가 클라이언트에 저장하도록 하는 작은 데이터

예: 로그인 유지, 장바구니 정보 저장

Set-Cookie: user_id=chejjeok; Path=/; HttpOnly

### 헤더(Header) 기반 토큰
API 요청 시 인증 정보 전달

예: Authorization: Bearer abcd1234

------
## 정리
HTTP 요청은 요청 라인, 요청 헤더, 요청 바디로 구성됨

HTTP 응답은 상태 라인, 응답 헤더, 응답 바디로 구성됨

쿼리 스트링, 패스 파라미터, 쿠키, 헤더를 통해 데이터를 주고받음

------
## Reference

<https://sehun5515.tistory.com/92>

<https://developing-move.tistory.com/293>

<https://junuuu.tistory.com/m/36?category=974977>

<https://blog.naver.com/ghdalswl77/222517833354>

<https://velog.io/@mokyoungg/HTTP-HTTP-%EA%B5%AC%EC%A1%B0-%EB%B0%8F-%ED%95%B5%EC%8B%AC-%EC%9A%94%E3%85%85>

<https://blueyikim.tistory.com/1999>

------
[^overhead]: 오버헤드(overhead)는 어떤 처리를 하기 위해 들어가는 간접적인 처리 시간이나 메모리 등을 말합니다.
[^persconn]: HTTP 지속 연결(Persistent Connection)은 `HTTP keep-alive` 또는 `HTTP 연결 재사용` 이라고도 불립니다.
[^body]: 요청, 응답에 구분없이 body는 Payload라고도 부른다.