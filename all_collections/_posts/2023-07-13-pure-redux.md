---
layout: post
title: 🌧 Pure Redux
date: 2023-07-13
categories: ["SamBeak", "Pure Redux", "Redux", "JavaScript"]
---

> # Arcitecture

<br>
이번 정리는 React를 떠나 Redux 자체만을 살펴보도록 할 것이다. <br><br>
오늘 살펴볼 Redux의 구조는 <br>
* 첫 번째는 store <br>
* 두 번째는 reducer <br>
* 세 번째는 dispatch <br>
* 네 번째는 subscribe <br>
이렇게 총 4가지이다. <br><br>

시작하기 앞서, Reudx를 사용하려면 <br>
라이브러리를 설치하고 모듈을 넣어주는 과정이 필요하다 <br><br>

필자는 yarn을 사용해서 설치할 것이고, <br>
yarn을 설치하거나 npm과의 차이는 따로 정리해두겠다. <br>

```
yarn add redux // 프로젝트 내 터미널에서 설치
```

> # Store

<br>
store란 state(상태) 즉, 어플리케이션에서 사용하는 data를 <br>
보관하는 곳이다. <br>
Redux는 createStore란 함수를 가지며, <br>
기본적으로 데이터가 보관될 장소를 생성해준다. <br>
사실 이게 Redux가 가장 잘하는 일이기도 하다. <br><br>

본래 createStore란 함수를 사용하였는데 <br>
라이브러리가 업데이트되며 legacy_createStore로 명칭이 바뀌었다. <br>
createStore는 reducer함수를 콜백하도록 작성한다. <br>
state 저장은 store에서 하며 절대 mutate하지 않고 <br>
저장만 한다는 것을 기억해야한다. <br>
state라는 말도, store명도 임의로 설정해도 무방하다. <br><br>

이후 state의 저장된 state를 가져오려면 <br>
getState메서드를 사용한다. <br><br>

```
import { legacy_createStore } from 'redux'; // 리덕스store함수를 사용할거야

const nameStore = legacy_createStore(reducerName);

nameStore.getState(); //
```

> # Reducer

<br>
Reducer란 처음으로 데이터를 정의하고 수정하는 함수다. <br>
modify(수정)하는 기능을 수행하고, return으로 반환한다. <br>
reducer는 current state와 action을 parameter로 갖고 <br>
불러와 action을 수행한다. <br>
다시 한번 말하지만, 데이터는 절대 mutate(변형)하지 않는다. <br>
오직 reducer에서만 유일하게 데이터를 modify할 수 있다. <br>
여기 외에선 데이터를 modify해서는 안 된다. <br><br>

reducer의 첫 번째 파라미터는 state 즉, 현재 데이터를 받고 <br>
두 번째 파라미터는 action을 받는다. <br>
첫 번째 파라미터에는 default값을 설정해줄 수 있다. <br><br>

데이터를 modify할 수 있게 핸들링하는 것이 바로 action이다. <br>
action은 두 번째 parameter로 modifier(reducer)와 소통하는 방법이다. <br>
action이 리턴하는 값은 state 즉, 데이터가 된다. <br><br>

action은 key와 value 쌍을 갖는다. <br>

공식 문서에선 reducer안에서 if/else문보다는 <br>
switch문을 사용하길 권장한다.<br>
또한, reducer안에서 Date함수를 쓰지 않길 권장하니 <br>
dispatch파트나 함수를 통해 Date를 action의 키값에 미리 할당하는 것이 좋다<br><br>

```
const ADD_TODO = "ADD_TODO";
const DELETE_TODO = "DELETE_TODO";

const addTodo = (todo) => {
    return {
        type: ADD_TODO,
        text: todo,
        id: Date.now()
    };
};

const dispatchAddTodo = (todo) => {
    todoStore.dispatch(addTodo(todo));
};

const onSubmit = (e) => {
  e.preventDefault();
  const todoText = todoInput.value;
  todoInput.value = "";
  dispatchAddTodo(todoText);
}

form.addEventListener('submit', onSubmit);

// reducer (modifier)
const todoModifier = (state =[], action) => {
    switch (action.type) {
        case ADD_TODO:
            return [...state, {text: action.text, id: action.id}];
        case DELETE_TODO:
            return [];
        default:
            return [];
    }
};

const todoStore = legacy_createStroce(todoModifier);
```

> # Dispatch

<br>
dispatch는 store의 메서드로서 <br> 
action에게 신호를 보내주는 메서드이다. <br>
key와 value 한 쌍으로 액션을 보내주게 되고 <br>
그렇게 보내면 action이 받아 reducer 안에서 인자로 사용하게 된다. <br>
dispatch가 current state와 action을 더해 리듀서에게 전달하는 방식이다. <br>
중요한건 action에게 보내는 신호는 object여야하지 <br>
절대 string은 안 된다. 오류가 난다. <br><br>

```
nameStore.dispatch({key:value});

nameStore.dispatch({type: "ADD_TODO"});
```

> # Subscribe

<br>
subscribe(구독)은 stroe안의 변화들을 알게 해주는 메서드다. <br>
변화를 감지한 이후에는 function을 둬서 작성할 수 있도록 한다. <br>
예를들어 nameStore.subscribe(fn)을 작성하면 <br>
nameStroe에 dispatch를 통해 action이 reducer로 전달 될 것이고 <br>
action을 전달받은 reducer는 데이터를 modify할 것이다. <br>
그렇게 저장소 안에 데이터가 변화게 되면 subscribe(구독자)는 <br>
알람을 받게 되고 fn을 동작하게 된다. <br>
결국, 저장소의 데이터만 변화하는 것이 목적이 아니라<br>
변화를 통해 subscribe가 새로운 동작을 시행하도록하는 설정하는 것이다. <br><br>

```
const functionName = () => {
    // 내용
};
nameStore.subscribe(functionName);
```
