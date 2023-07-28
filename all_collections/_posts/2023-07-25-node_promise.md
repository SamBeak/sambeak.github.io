---
layout: post
title: 🔗 Node Promise Async [5편]
date: 2023-07-25
categories: ["SamBeak", "Promise", "Async", "Mysql2", "Node"]
---

> # Promise Async

<br>
앞서 Promise와 Async는 포스팅한 적이 있으니 <br>
자세한 설명은 생략하고 4편에서 작성한 코드를 살펴보면 <br>
문제점이 하나 있다는 것을 알 수 있다. <br><br>

해당 라이브러리는 mysql에서 requset(SQL문)를 전달하고 <br>
SQL문을 받는 동안까지 기다림의 시간을 Promise로 처리를 했는데 <br>
그 처리에 대한 return값을 함수가 제대로 받지 못하는 것이다. <br>
여기서 Promise로 처리했다는 말이 무슨 말이냐면 <br>
콜백함수처리했다는 말이다. funcion(err,req,fields){}; 이 부분! <br>
함수를 만들어서 그 안에 pool.query를 통해 SQL문을 요청하고, <br>
바로 콜백함수가 나오게 처리했다는 말이다. <br><br>

문제는 콜백함수가 반환될 때까지 시간이 걸린다는 것인데 <br>
왜 문제냐면 pool.query문 뒤로 console.log("vlavla")가 있어도 <br>
console.log가 먼저 나오고 쿼리 결과가 출력되기 때문이다. <br>
우리는 원하는 때에 원하는게 동작되기를 원한다. <br><br>

이를 위한 방법으론 Promise로 처리를 하든지 <br>
Async/Await을 써서 해결하는 것이다.<br><br>

JavaScript의 동작과정은 CALL STACK과 <br>
MICROTASK QUEUE, MACROTASK QUEUE가 존재하는데 <br>
MICROTASK QUEUE가 먼저 처리되고 <br>
MACROTASK QUEUE가 처리되는 방식으로 진행된다. <br>
내가 작성한 mysql 함수는 MICROTASK QUEUE 여기 해당한다. <br>
문제는 console같은 경우는 CALL STACK에 해당돼 바로 처리된다는 것이다. <br>
CALL STACK이 0순위, MICROTASK QUEUE가 1순위, <br>
MACROTASK QUEUE가 2순위인셈이고 <br>
결국 우리가 하고자하는 것은 콜스택으로 처리하기 위해 <br>
Promise나 Async/Await처리를 하는 것이다. <br><br>
