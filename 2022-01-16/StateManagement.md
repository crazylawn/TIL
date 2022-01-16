# recoil vs redux vs contextapi

https://recoiljs.org/ko/
https://ko.redux.js.org/introduction/getting-started/
https://ko.reactjs.org/docs/context.html

3개다 상태관리 라이브러리이다. 이 3개의 라이브러리가 어떻게 다른지 알아보자!

## recoil

recoil 은 context API 기반으로 구현된 함수 컴포넌트에서만 사용 가능한 페이스북에서 만든 상태관리 라이브러리이다.

첫번째로 배우기 쉽다. api 가 단순하고 hook을 사용하고 있는 사람들에게 익숙할거 같다.
recoil 의 기본 Atom
atom 은 기존의 redux에서 쓰이는 store 와 유사한 개념으로, 하나의 상태를 의미한다.
useState 에서 사용하는 state 와 비슷한데
atom 의 값을 변경할때 해당 atom 을 구독하고 있는 모든 컴포넌트들이 리렌더링 되면 해당 변경된 atom의 값을 사용하게 된다.

redux에서는 reducer 단위로 state 을 구성했는데
atom 은 이런 reducer 단위가 이닌 더 작게 쪼개진 state 단위로 상태를 관리 할 수있게 된다.

atom 을 생성할때는 2가지의 값을 필수로 설정 해줘야한다.

key : 고유한 key값 (보톤 해당 atom을 생성하는 변수 명으로 지정함)
default : atom 의 초기값을 정의하는데 정적인 값 (int, string) 등등 다른 atom의 값으로 설정할 수 있다.

4가지 훅을 설명 하겠다.

```

useRecoilState -기존 useState 와 같이 변경되는 값과 해당 값을 변경하는 함수를 반환한다.
useState 와 같은 방식을 사용한다.

useRecoilValue - setter 함수 없이 atom의 값을 반환한다.값의 update 함수가 필요없을 경우 사용된다.

useSetRecoilState - setter 함수만 반환한다.

useResetRecoilState - 값을 기본값으로 reset 시키는 함수를 반환한다.

```

atom이 업데이트 되면, 해당 atom을 구독하고 있던 모든 컴포넌트들의 state가 새로운 값으로 리렌더 된다.
unique 한 id인 key로 구분되는 각 atom은,
여러 컴포넌트에서 atom을 구독하고 있다면 그 컴포넌트들도 똑같은 상태를 공유한다.
(상태가 바뀌면 바뀐 값으로 해당 컴포넌트들이 re-render 됩니다)

recoil 사용을 위한 세팅

```
 <RecoilRoot>
      <App />
    </RecoilRoot>
```

프로젝트 최상단 위치에 recoilRoot 를 다음과 같이 설정
redux 에서는 store 설정을 해주는 부분이 있었는데 휠씬 더 간결하게 설정

```
state.js


export const coffeeState = atom({
  key: 'coffeeState',
  default: [
      {
          name : 'Coffee',
          type :'MegaCoffee'
      }
  ]
});
```

key 로 고류한 atom 을 구분하고
default 에 기본으로 atom에 저장되는 값을 지정할 수 있다.

```
Coffee.js

import React from 'react'
import { coffeeState } from '../../state';
import { useRecoilState } from 'recoil';

const Coffee = () => {
	const [coffee, setCoffee] = useRecoilState(coffeeState);

  return(
    <div>
        {coffee.map(coffee => (
          <Card
            coffee={coffee}
            key={coffee.id}
            idx={coffee.id}
           />
       ))}
      </div>
  );
}
export default Coffee;
```

React 의 기본 hook인 useState 와 유사한 형태를 가지고 있다.

```
const [coffee , setCoffee] = useRecoilState(coffeeState);
```

구조 분해 할당으로 state와 state를 set 하는 함수를 각각 coffee 와 setCoffee 에서 받아올 수 있다. 이것을 통해서 atom 에 어떤 컴포넌트에서든 전역으로 받아올 수 있고, setCoffee 로 state 를 변경할 수도 있다. 당연한것은 어떤 컴포넌트에서 state를 변경하더라도 이것을 사용하고 있는 컴포넌트들은 모두 그 값이 갱신되어 re-render 된다.

atom 에서는 state
loadable 에서는 state or contents
snapshot 에서는 snapshot의 value에 접근하거나 value을 set 할수있는 함수를 불러올 수 있는 hook 에 해당한다.

우리는 useRecoilState() 이값을 전역에서 관리할것이기 때문에
value 만 필요한 컴포넌트도 있을거고, state 만 변경하는 컴포넌트도 있을건데, 이 상황을 위해서 useRecoilState()의 역할을 쪼개서

useRecoilValue() 그리고 useSetRecoilState() 로 나눌 수 있다.

```
import { useRecoilValue, useSetRecoilState}  from 'recoil';
import {coffeeState} from  '../state';

const Coffee = useRecoilValue(coffeeState);
const setCoffee = useSetRecoilState(coffeeState);

useRecoilValue , useSetRecoilState 를 사용하면  useRecoilState() 을 사용했을 때와 같은역할을 하게 된다.
```

useResetRecoilState() 라는 hook 도 있는데
위에 hook은 인자로 받아온 atom 의 state를 default 값으로 reset 시키는 역할을 한다.

```
const resetCoffee = useResetRecoilState(coffeeState)
```

selector
api 통신과 같은 비동기 처리 부분은 selector 에서 한 번에 처리할 수 있다.하지만 순수함수여야 한다.

```
selector 는 공식문서의 설명에 파생된 state 를 저장하고 있다.
다른 atom에 의존하는 동적인 데이터를 만들 수 있게 해준다.

Recoil의 selector 는 기존에 우리가 알던 selector의 개념과는 조금 다르다. redux의 reselect 와 MobX의 @computed처럼 동작하는 "get" 함수를 가지고 있다. 그렇지만 하나이상의 atom을 업데이트를 할 수 있는 set 함수를 옵션을 받을 수 있다.
```

```
const coffeeListState  = selector({
    key : 'coffeeNameState',
    get : ({get}) => {
        return `${get(coffeeState)}`;
    },
    set : ({set}) => {

    }
})
```

Recoil 특징 1.비동기 처리를 기반으로 작성되어 동시성 모드를 제공하기 때문에, Redux와 같이 다른 비동기 처리 라이브러리에 의존할 필요가 없다.

2.atom -> selector를 거쳐 컴포넌트로 전달되는 하나의 data-flow를 가지고 있어, 복잡하지 않은 상태 구조를 가지고 있다.

## redux

Flux 아키텍쳐 기반으로 데이터의 흐름이 단방향이다.

action , reducer , dispatcher, store , view 등에 대한 개념이 나온다.

대규모 프로젝트에서 사용하기는 적합하지 않다.
주요 개념

action
redux 에서 action 은 state 를 방식이다.
액션객체를 가지고 있고 , 반드시 type 필드가 있어야 한다.

dispatch
action 을 발생시키는 거승로 action 객체를 파라미터로 받는다.

reducer
변화를 일으키는 함수로 action 의 결과 stat 를 어떤식으로 바꿀지
구체적으로 정의하는 부분이다.
리듀서가 현재 상태와 전달받은 액션 객체를 파라미터로 받아서 새로운 상태로 반환해준다.
리듀서는 파라미터 외의 값을 의존하면 안되고, 이전상태는 건드리지 않은 상태로 새로운 상태 객체를 만들어 반환하는 순수 함수 여야한다.

store
프로젝트에 리덕스를 적용하기 위해서 필요한데 프로젝트에는 단 한개의 store만 갖고 있는 중앙 저장소라고 할 수 있다. store 안에는 리듀서와 내장함수 등이 포함되어 있다.

dispatch 와 subscribe 들도 스토어의 내장함수이다.
상태를 읽을때는 getState() , 상태를 바꿀때는 dispatch()를 호출한다.

상태를 전역적으로 관리하기 때문에 어느 컴포넌트에 상태를 뒤야할지 고민할 필요가 없게 된다.

그래서 redux 는 여러 위치에 많은 양의 상태 값이 존재 할때 ,
거대한 코드베이스를 여러사람이 작업 할때,
상태 변경 시각화가 필요 할때,

액션을 하나 추가하는데, 작성에 필요한 부분들이 많고, 컴포넌트와 스토어를 연결하는 필수적인 부분들이 있어 코드량이 많아질 수 있다!

현재 프로젝트에서는 redux 를 활용해서 쓰고 있는데 코드량이 조금 많아
지기는 하지만 익숙해지면 괜찮은거 같다.

## contextapi

context 는 컴포넌트안에서 전역적으로 데이터를 공유하도록 나온 개념이다.

리액트 16.3 부터 도입된 context api 를 사용하면 네이티브 리액트 만으로 전역 상태관리가 가능하다.
contextAPI 는 단지 종속성 주입의 한 형태일뿐 아무것도 관리하지 않는다.
지금 부터 세가지 주요 개념에 대해서 알아보겠다.
createContext , Provider , Consumer

- createContext(defaultValue)

### context 객체를 만들때 사용하는 함수이다.

### createContext 함수를 호출하면 Provider 와 Consumer 컴포넌트를 반환한다.

### defaultValue 는 초기값을 말한다.(Provider 사용하지 않을때 적용되는 값)

- context.Provider

### context 의 value 를 변경하는 역할

### Provider 를 사용할때에 value을 꼭 명시해주어야만 동작함

### 전달받는 컴포넌트의 수는 제한이 없다.

- context.Consumer

### context 변화를 구독하는 컴포넌트를 말한다.

### 설정한 값을 불러와야 할 때 사용

```
//createContext 사용
import { createContext } from "react";

const CoffeeContext = createContext({ coffee: "megacoffee" });
```

```
//consumer 사용

<CoffeeContext.Consumer>
</CoffeeContext.Consumer>

```

```
//provider 사용

<CoffeeContext.Provider vlaue={{변경될 내용추가 }}>
</CoffeeContext.Provider>
```

그리고 Context 를 사용하면 컴포넌트 재사용이 어려워짐으로 꼭 필요할 때에만 사용해야 하고 Context는 Flux와 같은 상태 관리 시스템을 대체할 수 없다.
