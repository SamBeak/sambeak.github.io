---
layout: post
title: 🚀 JavaScript 동작 원리 완벽 가이드
date: 2025-11-04
categories:
  [
    "SamBeak",
    "JavaScript",
    "Event Loop",
    "Call Stack",
    "Asynchronous",
    "Frontend",
  ]
---

# JavaScript 동작 원리란 무엇인가

<br>
JavaScript를 사용하다 보면 신기한 현상을 경험한다. <br>
setTimeout을 0초로 설정해도 코드가 나중에 실행되거나, <br>
함수를 선언하기 전에 호출해도 동작하는 경우가 있다. <br><br>

이는 마치 식당 주방과 같다. <br>
주방장(JavaScript 엔진)은 한 번에 한 가지 요리만 만들지만(싱글 스레드), <br>
오븐에 음식을 넣고 타이머를 맞춰두면(비동기), <br>
그 사이에 다른 요리를 계속할 수 있다. <br><br>

JavaScript의 동작 원리를 이해하면 <br>
왜 코드가 특정 순서로 실행되는지, <br>
비동기 처리가 어떻게 이루어지는지 명확하게 알 수 있다. <br><br>

> ## 왜 JavaScript 동작 원리를 알아야 할까?

<br>

**문제 1: 예상치 못한 실행 순서** <br>
비동기 코드가 언제 실행될지 모르면 버그가 발생한다. <br><br>

**문제 2: 성능 문제** <br>
무거운 작업이 화면을 멈추게 만든다. <br><br>

**문제 3: 면접 단골 질문** <br>
이벤트 루프, 콜 스택은 프론트엔드 면접 필수 주제 <br><br>

**문제 4: 디버깅 어려움** <br>
실행 순서를 모르면 에러 추적이 힘들다. <br><br>

# 기본 개념 요약

<br>

## 🏷️ JavaScript는 싱글 스레드

<br>
**개념**: 한 번에 하나의 작업만 처리 <br><br>

**식당 비유**: <br>
주방장이 한 명이라 한 번에 하나의 요리만 만든다. <br>
하지만 오븐, 냉장고(Web APIs)를 활용해서 <br>
효율적으로 여러 요리를 동시에 진행할 수 있다. <br><br>

**장점**: 간단하고 예측 가능 <br>
**단점**: 무거운 작업이 전체를 막을 수 있음 <br><br>

## 🏷️ JavaScript 실행 환경

<br>

### 1. JavaScript 엔진 (V8, SpiderMonkey)

<br>

**구성 요소**:

- **힙(Heap)**: 변수, 객체 저장 (메모리)
- **콜 스택(Call Stack)**: 함수 실행 순서 관리

<br>

### 2. Web APIs

<br>
브라우저가 제공하는 비동기 기능: <br>

- setTimeout / setInterval
- DOM 이벤트
- fetch (HTTP 요청)
- Promise

<br>

### 3. 태스크 큐 (Task Queue)

<br>
비동기 작업이 완료되면 대기하는 곳 <br><br>

### 4. 이벤트 루프 (Event Loop)

<br>
콜 스택과 태스크 큐를 감시하며 작업 조율 <br><br>

## 🏷️ 콜 스택 (Call Stack)

<br>
**개념**: 함수 호출을 추적하는 스택 자료구조 <br><br>

**동작 방식**:
1. 함수 호출 → 스택에 push
2. 함수 종료 → 스택에서 pop
3. LIFO (Last In First Out)

<br>

**예시**:

```javascript
function first() {
  console.log("첫 번째");
  second();
  console.log("첫 번째 끝");
}

function second() {
  console.log("두 번째");
}

first();

// 콜 스택 순서:
// 1. first() push
// 2. console.log("첫 번째") push → pop
// 3. second() push
// 4. console.log("두 번째") push → pop
// 5. second() pop
// 6. console.log("첫 번째 끝") push → pop
// 7. first() pop
```

<br>

## 🏷️ 이벤트 루프 (Event Loop)

<br>
**개념**: 콜 스택이 비어있으면 태스크 큐에서 작업을 가져옴 <br><br>

**동작 원리**:

```
1. 콜 스택 확인
2. 비어있으면 → 태스크 큐에서 가져오기
3. 비어있지 않으면 → 대기
4. 반복
```

<br>

**핵심 규칙**: <br>
콜 스택이 완전히 비어야만 태스크 큐의 작업 실행 <br><br>

## 🏷️ 매크로태스크 vs 마이크로태스크

<br>

### 매크로태스크 (Macrotask)

<br>

- setTimeout
- setInterval
- setImmediate
- I/O 작업

<br>

### 마이크로태스크 (Microtask)

<br>

- Promise.then
- queueMicrotask
- MutationObserver

<br>

**우선순위**: 마이크로태스크가 매크로태스크보다 먼저 실행 <br><br>

**실행 순서**:
1. 콜 스택 실행
2. 모든 마이크로태스크 실행
3. 매크로태스크 하나 실행
4. 다시 마이크로태스크 모두 실행
5. 반복

<br>

# 실전 예시

<br>

> ## 콜 스택 동작 이해하기

<br>

```javascript
function multiply(a, b) {
  return a * b;
}

function square(n) {
  return multiply(n, n);
}

function printSquare(n) {
  const result = square(n);
  console.log(result);
}

printSquare(4);

// 콜 스택 변화:
// 1. [printSquare(4)]
// 2. [printSquare(4), square(4)]
// 3. [printSquare(4), square(4), multiply(4, 4)]
// 4. [printSquare(4), square(4)]  // multiply 리턴
// 5. [printSquare(4)]  // square 리턴
// 6. [printSquare(4), console.log(16)]
// 7. []  // 모두 완료

// 출력: 16
```

<br>

> ## 비동기 코드 실행 순서

<br>

```javascript
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

console.log("3");

// 출력:
// 1
// 3
// 2

// 왜?
// 1. console.log("1") 실행 (콜 스택)
// 2. setTimeout은 Web API로 이동 (콜 스택에서 제거)
// 3. console.log("3") 실행 (콜 스택)
// 4. 콜 스택 비움
// 5. 이벤트 루프가 태스크 큐에서 setTimeout 콜백 가져옴
// 6. console.log("2") 실행
```

<br>

> ## Promise vs setTimeout 우선순위

<br>

```javascript
console.log("시작");

setTimeout(() => {
  console.log("setTimeout");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("Promise 1");
  })
  .then(() => {
    console.log("Promise 2");
  });

console.log("끝");

// 출력 순서:
// 시작
// 끝
// Promise 1
// Promise 2
// setTimeout

// 이유:
// 1. 동기 코드 먼저 (시작, 끝)
// 2. 마이크로태스크 (Promise)
// 3. 매크로태스크 (setTimeout)
```

<br>

> ## 이벤트 루프 상세 예제

<br>

```javascript
console.log("Script start");

setTimeout(function timeout() {
  console.log("setTimeout");
}, 0);

Promise.resolve()
  .then(function promise1() {
    console.log("Promise 1");
  })
  .then(function promise2() {
    console.log("Promise 2");
  });

requestAnimationFrame(function animation() {
  console.log("requestAnimationFrame");
});

console.log("Script end");

// 출력 순서:
// Script start
// Script end
// Promise 1
// Promise 2
// requestAnimationFrame (다음 프레임)
// setTimeout

// 실행 순서:
// 1. 동기 코드 실행 (콜 스택)
// 2. 마이크로태스크 큐 비우기 (Promise)
// 3. 렌더링 (requestAnimationFrame)
// 4. 매크로태스크 큐에서 하나 실행 (setTimeout)
```

<br>

> ## 무한 루프로 인한 스택 오버플로우

<br>

```javascript
// ❌ 스택 오버플로우 발생
function recursiveFunction() {
  recursiveFunction();
}

// recursiveFunction(); // Uncaught RangeError: Maximum call stack size exceeded

// ✅ setTimeout으로 해결
function safeRecursive() {
  console.log("실행");
  setTimeout(safeRecursive, 0);
}

// safeRecursive(); // 무한 실행되지만 스택은 안전
```

<br>

**이유**: <br>
setTimeout은 콜백을 태스크 큐로 보내므로 <br>
콜 스택이 매번 비워진다. <br><br>

> ## 블로킹 vs 논블로킹

<br>

### 블로킹 코드 (나쁜 예)

<br>

```javascript
// ❌ 5초 동안 화면이 멈춤
const startTime = Date.now();
while (Date.now() - startTime < 5000) {
  // 5초 대기
}
console.log("5초 경과");
```

<br>

### 논블로킹 코드 (좋은 예)

<br>

```javascript
// ✅ 화면이 멈추지 않음
setTimeout(() => {
  console.log("5초 경과");
}, 5000);

console.log("다른 작업 계속 가능");
```

<br>

# 실행 컨텍스트 (Execution Context)

<br>

## 🏷️ 실행 컨텍스트란?

<br>
**개념**: 코드가 실행되는 환경 <br><br>

**구성 요소**:
1. **Variable Environment**: 변수, 함수 선언 저장
2. **Lexical Environment**: 스코프 체인
3. **this 바인딩**

<br>

## 🏷️ 실행 컨텍스트 생성 과정

<br>

### 1. Creation Phase (생성 단계)

<br>

```javascript
console.log(name); // undefined (호이스팅)
var name = "홍길동";

// Creation Phase:
// - name 변수 선언 (값은 undefined)
// - 함수 선언 전체를 메모리에 저장
```

<br>

### 2. Execution Phase (실행 단계)

<br>

```javascript
var name = "홍길동";
console.log(name); // "홍길동"

// Execution Phase:
// - name에 "홍길동" 할당
// - 코드 한 줄씩 실행
```

<br>

## 🏷️ 호이스팅 (Hoisting)

<br>
**개념**: 선언이 스코프 최상단으로 끌어올려지는 것처럼 동작 <br><br>

### var 호이스팅

<br>

```javascript
console.log(age); // undefined
var age = 30;

// 위 코드는 내부적으로 이렇게 동작:
var age;
console.log(age); // undefined
age = 30;
```

<br>

### 함수 호이스팅

<br>

```javascript
// ✅ 함수 선언식: 호이스팅됨
sayHello(); // "Hello!"

function sayHello() {
  console.log("Hello!");
}

// ❌ 함수 표현식: 호이스팅 안 됨
sayBye(); // TypeError: sayBye is not a function

var sayBye = function () {
  console.log("Bye!");
};
```

<br>

### let/const는 호이스팅되지만 TDZ

<br>

```javascript
// ❌ ReferenceError
console.log(name); // Cannot access 'name' before initialization
let name = "홍길동";

// TDZ (Temporal Dead Zone):
// 선언은 호이스팅되지만 초기화 전까지 접근 불가
```

<br>

# 스코프 (Scope)

<br>

## 🏷️ 스코프란?

<br>
**개념**: 변수에 접근할 수 있는 범위 <br><br>

### 1. 전역 스코프 (Global Scope)

<br>

```javascript
const globalVar = "전역 변수";

function test() {
  console.log(globalVar); // 접근 가능
}
```

<br>

### 2. 함수 스코프 (Function Scope)

<br>

```javascript
function outer() {
  const outerVar = "outer";

  function inner() {
    console.log(outerVar); // 접근 가능 (스코프 체인)
  }

  inner();
}

// console.log(outerVar); // ReferenceError
```

<br>

### 3. 블록 스코프 (Block Scope)

<br>

```javascript
if (true) {
  let blockVar = "블록 변수";
  const blockConst = "블록 상수";
  var functionVar = "함수 변수";
}

// console.log(blockVar); // ReferenceError
// console.log(blockConst); // ReferenceError
console.log(functionVar); // "함수 변수" (var는 블록 스코프 무시)
```

<br>

## 🏷️ 스코프 체인

<br>

```javascript
const global = "전역";

function outer() {
  const outerVar = "outer";

  function inner() {
    const innerVar = "inner";
    console.log(innerVar); // inner
    console.log(outerVar); // outer (상위 스코프 참조)
    console.log(global); // 전역 (최상위 스코프 참조)
  }

  inner();
}

outer();

// 스코프 체인:
// inner → outer → global
```

<br>

# 클로저 (Closure)

<br>

## 🏷️ 클로저란?

<br>
**개념**: 함수가 선언될 때의 렉시컬 환경을 기억하는 함수 <br><br>

```javascript
function makeCounter() {
  let count = 0; // 외부 함수의 변수

  return function () {
    count++;
    return count;
  };
}

const counter = makeCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3

// counter 함수는 count 변수를 기억함 (클로저)
```

<br>

## 🏷️ 클로저 활용 예시

<br>

### 1. 데이터 은닉 (Private 변수)

<br>

```javascript
function createWallet() {
  let balance = 0; // private 변수

  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (balance >= amount) {
        balance -= amount;
        return balance;
      }
      return "잔액 부족";
    },
    getBalance() {
      return balance;
    },
  };
}

const myWallet = createWallet();
myWallet.deposit(1000); // 1000
myWallet.withdraw(300); // 700
console.log(myWallet.getBalance()); // 700

// balance에 직접 접근 불가
console.log(myWallet.balance); // undefined
```

<br>

### 2. 함수 팩토리

<br>

```javascript
function makeMultiplier(multiplier) {
  return function (number) {
    return number * multiplier;
  };
}

const double = makeMultiplier(2);
const triple = makeMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

<br>

### 3. 이벤트 핸들러

<br>

```javascript
function setupButtons() {
  for (let i = 0; i < 3; i++) {
    const button = document.getElementById(`btn${i}`);
    button.addEventListener("click", function () {
      console.log(`버튼 ${i} 클릭`); // i를 기억 (클로저)
    });
  }
}

// var 사용 시 문제
function setupButtonsBad() {
  for (var i = 0; i < 3; i++) {
    const button = document.getElementById(`btn${i}`);
    button.addEventListener("click", function () {
      console.log(`버튼 ${i} 클릭`); // 항상 3 출력
    });
  }
}
```

<br>

# 비동기 처리 패턴

<br>

## 🏷️ 콜백 (Callback)

<br>

```javascript
function fetchUser(callback) {
  setTimeout(() => {
    const user = { name: "홍길동", age: 30 };
    callback(user);
  }, 1000);
}

fetchUser((user) => {
  console.log(user); // { name: "홍길동", age: 30 }
});
```

<br>

**문제점**: 콜백 지옥 (Callback Hell) <br>

```javascript
// ❌ 콜백 지옥
fetchUser((user) => {
  fetchPosts(user.id, (posts) => {
    fetchComments(posts[0].id, (comments) => {
      console.log(comments);
    });
  });
});
```

<br>

## 🏷️ Promise

<br>

```javascript
function fetchUser() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const user = { name: "홍길동", age: 30 };
      resolve(user);
    }, 1000);
  });
}

fetchUser()
  .then((user) => {
    console.log(user);
    return fetchPosts(user.id);
  })
  .then((posts) => {
    console.log(posts);
    return fetchComments(posts[0].id);
  })
  .then((comments) => {
    console.log(comments);
  })
  .catch((error) => {
    console.error(error);
  });
```

<br>

## 🏷️ Async/Await

<br>

```javascript
async function getUserData() {
  try {
    const user = await fetchUser();
    console.log(user);

    const posts = await fetchPosts(user.id);
    console.log(posts);

    const comments = await fetchComments(posts[0].id);
    console.log(comments);
  } catch (error) {
    console.error(error);
  }
}

getUserData();
```

<br>

**장점**: 동기 코드처럼 읽기 쉬움 <br><br>

# 실전 체크리스트

<br>

## ✅ 콜 스택 이해

<br>

- [ ] 함수 호출 순서를 추적할 수 있는가?
- [ ] 스택 오버플로우 원인을 아는가?
- [ ] 재귀 함수의 콜 스택을 이해하는가?

<br>

## ✅ 이벤트 루프

<br>

- [ ] 비동기 코드의 실행 순서를 예측할 수 있는가?
- [ ] 마이크로태스크와 매크로태스크 차이를 아는가?
- [ ] 블로킹 코드를 피할 수 있는가?

<br>

## ✅ 실행 컨텍스트

<br>

- [ ] 호이스팅을 이해하는가?
- [ ] var, let, const 차이를 아는가?
- [ ] TDZ(Temporal Dead Zone)를 아는가?

<br>

## ✅ 스코프와 클로저

<br>

- [ ] 스코프 체인을 이해하는가?
- [ ] 클로저를 활용할 수 있는가?
- [ ] private 변수를 만들 수 있는가?

<br>

## ✅ 비동기 처리

<br>

- [ ] Promise를 작성할 수 있는가?
- [ ] async/await를 사용할 수 있는가?
- [ ] 에러 처리를 올바르게 하는가?

<br>

# 요약

<br>
JavaScript 동작 원리를 이해하면 더 나은 코드를 작성할 수 있다. <br><br>

**💎 핵심 포인트**:

1. **싱글 스레드**: 한 번에 하나씩 처리
2. **이벤트 루프**: 비동기를 가능하게 함
3. **콜 스택**: 함수 실행 순서 관리
4. **마이크로태스크 우선**: Promise가 setTimeout보다 먼저
5. **호이스팅**: 선언이 끌어올려짐
6. **클로저**: 렉시컬 환경 기억

<br>

**🚀 비동기 처리 진화**:

1단계: Callback (콜백 지옥 문제) <br>
2단계: Promise (체이닝 가능) <br>
3단계: Async/Await (동기 코드처럼 작성) <br><br>

**⚠️ 주의사항**:

- 무거운 동기 작업은 화면을 멈춤
- setTimeout(fn, 0)도 즉시 실행되지 않음
- var 대신 let/const 사용
- 클로저로 메모리 누수 주의

<br>

**📊 실행 순서 정리**:

```
1. 동기 코드 (콜 스택)
2. 마이크로태스크 (Promise, queueMicrotask)
3. 렌더링 (requestAnimationFrame)
4. 매크로태스크 (setTimeout, setInterval)
```

<br>

JavaScript는 싱글 스레드지만 <br>
이벤트 루프와 Web APIs 덕분에 <br>
비동기 처리가 가능하다. <br><br>

이 원리를 이해하면 <br>
코드의 실행 순서를 예측하고, <br>
효율적인 비동기 코드를 작성하며, <br>
디버깅도 훨씬 쉬워진다. <br><br>
