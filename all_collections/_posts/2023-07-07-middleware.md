---
layout: post
title: 💻 서버사이드 자바스크립트 Node [2편]
date: 2023-07-07
categories: ["SamBeak", "Node", "MiddleWare", "Express"]
---

> # 미들웨어에 대하여

<br>
미들웨어는 요청과 응답의 중간 단계에서 이뤄지는 소프트웨어 구성 요소를 의미한다. <br>
미들웨어는 클라이언트와 서버 간의 통신을 처리하고 <br>
어플리케이션의 동작을 제어하며 기능을 확장시키는 역할을 수행한다. <br>
보통은 요청 처리와 응답처리, 어플리케이션 제어, 기능 확장 등을 주요 목적으로 사용되는데 <br>
각 목적은 다음과 같다. <br><br>

- 요청 처리 : 미들웨어는 인증, 인가, 데이터 유효성 검사 등 <br>
  클라이언트로부터의 요청을 처리하고 필요한 동작을 수행한다. <br>
- 응답 처리 : 서버가 클라이언트에게 보낸 응답을 훔쳐 수정 처리한다. <br>
  응답에 헤더를 추가하거나 데이터를 압축하거나 캐싱을 사용하는 것이 그 예이다. <br>
- 어플리케이션 제어 : 미들웨어는 조건 처리를 하거나 다음 미들웨어로 넘기거나 에러 처리 등 <br>
  어플리케이션의 흐름을 제어하고 요청과 응답의 전달을 조정한다. <br>
- 기능 확장 : 미들웨어는 어플리케이션의 기능을 확장한다. <br>
  외부API를 호출하거나 DB 연결하거나 캐싱 작업을 수행하는 것이 그 예이다. <br>

미들웨어를 통해 어플리케이션을 모듈화 시킬 수 있고, 유연하고 재사용 가능한 구성 요소로 분리해 <br>
관리할 수 있다. <br><br>

> # 미들웨어 유형

<br>
미들웨어에서 일반적으로 사용되는 유형은 로깅 미들웨어, 인증 및 인가 미들웨어, <br>
라우팅 미들웨어, 에러 처리 미들웨어, 압축 미들웨어, 캐싱 미들웨어, DB관리 미들웨어 등이 있다. <br><br>

로깅 미들웨어는 어플리케이션의 로그를 기록하고 저장하는 미들웨어다. <br>
로깅은 디버깅, 모니터링, 오류 추적 등에 사용되고 어플리케이션의 동작을 파악하는데 유용하다. <br><br>

인증 및 인가 미들웨어는 사용자의 신원을 확인하고 권한을 부여하는데 사용되는 미들웨어다. <br>
사용자 인증을 처리하고, 접근 제어 및 권한 검사를 수행해 보안을 강화하는 역할을 한다. <br><br>

라우팅 미들웨어는 요청한 URL 경로를 기반으로 해당하는 핸들러나 컨트롤러 요청을 라우팅하는 미들웨어다. <br>
다양한 엔드포인트를 관리하고 적절한 처리 로직으로 요청을 전달한다. <br><br>

에러 처리 미들웨어는 발생한 에러를 처리하고 클라이언트에게 응답하는 미들웨어다. <br>
예외 처리, 오류 메시지 생성, 에러 페이지 표시 등을 담당하는 역할을 한다. <br><br>

압축 미들웨어는 요청과 응답의 데이터를 압축 전달하는 미들웨어다. <br>
대용량 데이터 전송을 최적화하고 네트워크 대역폭을 절약하는 역할을 한다. <br><br>

캐싱 미들웨어는 반복적 요청에 대한 응답을 캐싱하여 성능을 향상시키는 미들웨어다. <br>
캐싱의 역할은 앞서 여러 차례 언급함으로 생략하고자 한다. <br><br>

> # 미들웨어 요청 처리 흐름

<br>
* 클라이언트 요청 : 클라이언트 요청이 서버로 도달한다. 요청은 라우터를 통해 어플리케이션에 전달된다. <br>
* 미들웨어 실행 : 요청이 전달되면 첫 번째 미들웨어가 동작한다. 주로 요청을 가로채 필요한 전처리 작업을 수행한다. <br>
그래서 보통은 로깅 미들웨어가 로그를 생성하고 저장하는 단계가 이뤄지는 구간이다. <br>
* 다음 미들웨어 실행 : 인증 및 인가, 데이터 유효성 검사 등 각 미들웨어가 특정한 역할을 담당해 순차적으로 실행된다. <br>
* 요청 전달 : 요청을 적절한 핸들러나 컨트롤러로 전달하고 필요한 로직을 수행한다. <br>
* 응답 생성 : 핸들러나 컨트롤러는 요청에 대한 응답을 생성하고 다시 미들웨어를 거쳐 클라이언트에게 전달된다. <br>
* 응답 처리 : 응답이 미들웨어 체인을 통과하며 압축 미들웨어 같은 후처리 작업을 거친다. <br>
* 응답 전달 : 최종 응답이 클라이언트에게 전달된다. <br>

> # 어플리케이션의 확장

<br>
미들웨어를 사용하면 어플리케이션의 기능을 확장할 수 있다. <br>
앞서 말한 인증과 인가 혹은 캐싱과 로깅이 대표적인 예시고, <br>
이 밖에도 에러를 처리하거나 세션을 관리하는 등의 작업을 수행해 <br>
어플리케이션의 기능을 이전보다 확장시킬 수 있다. <br>
또한, 미들웨어는 데이터를 변환하고 가공하는데 사용하기도 하는데 <br>
요청과 응답의 데이터를 변환하고 가공할 수 있게 만들어준다. <br><br>

> # Node.js Express 미들웨어 예시

<br>
Express에서 미들웨어를 사용할 때, 미들웨어만 모아서 표현하기도 하는데 <br>
원래 표현 과정은 이렇다고 한다. <br><br>

```
app.get('/user/:userId', (req, res, next) => {
    const filteredData = data.filter(item => {item.id == req.params.userId });
    next();
});
```