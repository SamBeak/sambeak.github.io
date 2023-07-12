---
layout: post
title: ♻ 리액트 useContext
date: 2023-07-09
categories: ["SamBeak", "Context", "Hook", "React"]
---

> # 서론

<br>
최근 백엔드 쪽을 공부하다보니 자연스럽게 리액트에 소홀했던 것 같다.<br>
그래서 오늘은 리액트의 훅 중 useContext에 대해 정리하고자 한다. <br>
오늘 정리는 알렉스 뱅크스, 이브 포셀로의 '러닝 리엑트'에서 참고해 정리한다. <br><br>

리액트를 사용하며 당연스럽게 작성해왔던 훅들이 <br>
최근 당연하지 않게 다가왔다. <br>
'상태관리' 라는 단어를 접한 후, useState와 Redux 그리고 MobX 등을 찾아보게 된 것도 그렇고 <br>
오늘 정리할 useContext에 대해서 의문이 든 것도 자연스러운 수순이었다. <br>
도대체 useContext는 왜 사용하는 것이고, <br>
어떻게하다 지금처럼 작성하게 된 것일까? <br><br>

> # 리액트 useContext 왜 사용하는가?

<br>
트리 루트의 한 위치에 상태를 저장하는 패턴은 <br>
리액트의 초기 버전이 성공할 수 있는 이유 중 하나였다. <br>
상태를 프롭을 통해 위 아래로 전달할 수 있는 것은 개발자들에겐 <br>
어쩌면 필수 통과 의례 같은 것이었다. <br><br>

다시 말해, 모든 리액트 개발자는 어떻게 상태를 양방향으로 전달할 수 있는지 <br>
반드시 알고 있어야만 했다. <br>
그러나 리액트가 더욱 발전하고 진화함에 따라서 <br>
컴포넌트 트리도 거대해졌고 이에 따라 상태를 한 군데 유지하는 것은 <br>
점차 비현실적인 사실이 되기 시작했다. <br>
상태를 십여 개의 노드를 거쳐 트리 위 아래로 전달하는 과정은 <br>
오류 발생 위험이 높았고, 버그도 빈번히 발생했다. <br><br>

마치 기차를 타고 출발역에서 종착지로 가는 셈이다. <br>
종착지가 목적지이지만 모든 역에서 정차했다가 가야하는 서운함이 있다.<br>
이럴 땐 오히려 비행기가 더 효율적이다. <br>
출발지에서 목적지까지 바로 가니 말이다. <br><br>

context는 이 때문에 등장했다. <br>
기존의 방식은 모든 역에 정차하는 기차와도 같았다. <br>
그러나 context 훅은 모든 역을 정차하지 않고 바로 목적지로 향한다. <br>
데이터를 한 위치에 저장할 수 있지만, <br>
데이터를 사용하지 않을 경우 여러 컴포넌트를 거쳐 최종 컴포넌트에 전달할 필요가 없어진 것이다. <br><br>

> # context의 2가지 컴포넌트

<br>
리액트에서는 새로운 컨텍스트 객체를 만들 때는 createContext라는 함수를 사용한다. <br>
그렇게 새로운 context 객체를 생성하고 나면 해당 컨텍스트는 <br>
Provider와 Consumer, 2가지 component를 갖는다. <br><br>

Provider는 컴포넌트 트리 전체나 트리 일부를 감싸는 리액트 컴포넌트이다. <br>
데이터가 비행기에 타는 출발지 공항이라 생각하면 쉽다. <br>
또한, 데이터 허브이기도 하다. <br>
모든 비행기는 Provider에서 출발해 다른 목적지로 향한다. <br><br>

각 목적지를 Consumer라고 한다. <br>
Consumer는 데이터를 읽어들여 작업을 수행할 도착지 공항이라 생각하면 된다. <br>
비행기는 만들어주면 반드시 띄워야한다는 것을 기억하자 <br>
무슨 말이냐면 import createContext로 만들었다면 export로 비행기를 띄우라는 뜻이다. <br><br>

간혹 context를 Provider와 Consumer 2개의 Component만으로 정확하게 분리할 때가 있는데 <br>
그렇게 여기기보다 되려 <br>
비행기를 만들었고(createContext), <br>
비행기에 싣고(Provider) <br>
비행기가 도착한다.(Consumer) <br>
로 구분하는게 이해하기 좋다 <br>
왜냐하면 createContext를 통해 default context 컴포넌트를 만들고, <br>
Provider를 만들고, Consumer를 만들 때도 있기 때문이다. <br>
그러면 Provider 에는 createContext도 없고, export도 없기 때문이다. <br><br>

> # context.Provider

<br>
Provier는 꼭 하나만 존재해야하는 것은 아니다. <br>
필요에 따라 여러개 사용해도 되고, 일부 컴포넌트 트리만 감싸도 된다. <br>
사실, 일부만 감싸는게 더 효율적으로 작동된다고도 한다. <br>
그럼에도 유의해야하는 것은 provider는 자신이 감싸는 컴포넌트의 자식들에게만 <br>
context를 제공한다는 것을 기억해야한다. <br><br>

Proivder는 출발지의 항공기와 같아서, 데이터를 싣고 띄워야한다. <br>
데이터를 싣는 작업은 value를 통해서 할 수 있다. <br>

```javascript
import { createContext } from "react";
import datas from "./data.json";
import colors from "./color-data.json";
export const DataContext = createContext();

render(
  const [color, setColor] = React.useState(colors);
  <DataContext.Provider value={{ datas, color, setColor }}>
    <Component />
  </DataContext.Provider>
);
```

또한, Provider는 useState 기능을 사용해 상태 관리를 더할 수 있다. <br>
만약 value에 setState가 더해진다면, 해당 함수가 호출될 때마다 <br>
State의 상태가 직접 변경된다는 것도 알 수 있다. <br>
그러나, context에 추가하는 것이 좋은 생각은 아닐 수 있다. <br>
나중에 다른 개발자가 이 함수를 사용하면서 실수할 여지가 있기 때문이다. <br>
value값을 직접 바꾸는 것보다는 함수를 만들어 context에 추가하는 편이 낫다. <br><br>

```javascript
import { createContext } from "react";
import datas from "./data.json";
import colors from "./color-data.json";
export const DataContext = createContext();

render(
  const [color, setColor] = React.useState(colors);

  const addColor = (title, colors) =>
  setColor([
    ...color,
    {
      id: v4(),
      rating: 0,
      title,
      colors
    }
  ]);

  const rateColor = (id, rating) =>
  setColor(
    color.map(item => (item.id === id ? {...item, rating} : item))
  );

  const removeColor = id => setColor(color.filter(item => item.id !== id));

  <DataContext.Provider value={{ datas, color, addColor, rateColor, removeColor }}>
    <Component />
  </DataContext.Provider>
);
```

> # contetxt.Consumer

<br>
useContext는 Consumer로부터 필요한 값을 얻는다. <br>
useContext를 사용하면 직접 접근할 수 있다. <br><br>

```javascript
import { useContext } from "react";
import { DataContext } from "./"; // 파일 path 위치 명시한다.

export default function NameComponent() {
  const { arryName } = useContext(DataContext);
  return();
}
```

이런 식으로 Consumer를 통해 데이터에 접근할 수도 있다. <br>

```javascript
import { useContext } from "react";
import { DataContext } from "./"; // 파일 path 위치 명시한다.

export default function NameComponent() {
  return (
    <DataContext.Consumer>
      {(context) => {
        return <div>{context.datas.map()}</div>;
      }}
    </DataContext.Consumer>
  );
}
```
