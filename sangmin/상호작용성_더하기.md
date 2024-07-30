# [상호작용 추가하기](https://ko.react.dev/learn/adding-interactivity)

## [이벤트에 응답하기](https://ko.react.dev/learn/responding-to-events)

### 이벤트 핸들러

```typescript
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```
<오피셜>
- handleClick은 **이벤트 핸들러**. 
- handleClick은 주로 해당 컴포넌트 내부에서 정의됨.

<사설>
- Custom Component 아니어도, "prop 형태로 전달된다"고 표현하는구나.
- 이벤트 핸들러를 JSX내 인라인 정의하는 형태는 추상화 관점에서 좋지 않게 느껴짐. <br/>
(컴포넌트의 JSX 반환 구조는 일관된 추상화 깊이를 유지)

<br/>

> #### 이벤트 핸들러로 전달한 함수들은 호출이 아닌 전달.
> 
> | 함수를 전달(적절) | 함수를 호출(부적절) |
> |---|---|
> | `<button onClick={handleClick}/>` | `<button onClick={handleClick()}/>` |
> | `<button onClick={() => alert('...')}/>` | `<button onClick={alert('...')}/>` |
> 
> handleClick()과 alert('...')는 모두 즉시 함수를 실행하는 구조.

> #### 적절한 HTML tag: 키보드 접근성 범위<br/>
> Tab 키를 통한 이동은 화면에 렌더링된 모든 포커스 가능한 요소들에 대해 적용.
> 
> 1. 포커스 가능한 기본 HTML 요소: \<button>, \<a>, \<input>, \<textarea>, \<select>, \<details> 등
> 
> 2. tabIndex 속성이 있는 요소: tabindex 속성을 통해 포커스를 받을 수 있도록 설정된 모든 요소
>
>> #### tabIndex
>> 
>> tabIndex란 스크린리더 사용자를 위해 키보드의 Tab키를 눌렀을 때 포커스의 이동 순서를 임의로 조정할 수 있는 요소. <br/>
>> 스크린리더 사용자 뿐만 아니라 특정 정보를 다루기 위해 해당 요소에 포커스를 주는 방법으로도 사용된다.
>> |tabIndex = "음수"|tabIndex = 0|tabIndex = "양수"|
>> | --- | --- | --- |
>> | tab에서 focus를 받을 수 있는 요소를 받지 못하게 함. | tab에서 focus를 받을 수 없는 요소를 받을수 있게 함. | tab focusing 순서 |
>> 
>>```typescript
>>import React from 'react';
>>
>>function App() {
>>  return (
>>    <div>
>>      <button>Button 1</button> <!-- 1st(tabIndex가 없는 기본적으로 포커스 가능한 요소에 focus) --> 
>>      <div tabIndex={3}>Div 1 with tabIndex 3</div> <!-- 4th(일반적으로 순서 강제 제어는 권장되지 않음) --> 
>>      <div tabIndex={2}>Div 2 with tabIndex 2</div> <!-- 3rd(일반적으로 순서 강제 제어는 권장되지 않음) --> 
>>      <div tabIndex={1}>Div 3 with tabIndex 1</div> <!-- 2nd --> 
>>      <div tabIndex={-1}>Div 4 with tabIndex -1</div> <!-- 제외됨(의미 X. 원래도 tab focusing 가능한 tag 아님) --> 
>>      <div tabIndex={0}>Div 5 with tabIndex 0</div> <!-- 5th --> 
>>    </div>
>>  );
>>}
>>
>> export default App;
>>```
>> 숨겨진 요소: display: none 또는 visibility: hidden 스타일이 적용된 요소는 포커스 순서에서 제외. <br/>
> 
> <br/>

<br/>

### 이벤트 전파

- 이벤트 핸들러는 자식 컴포넌트의 이벤트를 수신할 수도 있다. (기본 동작이 bubbling) 

    사실 **거의** 모든 이벤트는 버블링된다. <br/>
    물론 `focus` 이벤트와 같이 버블링 되지 않는 이벤트도 있다.

  > #### `focus` 이벤트가 버블링되지 않는 이유
  > 
  > focus 이벤트는 주로 입력 요소(input, textarea, select 등)와 관련된 이벤트. 특정 요소에 초점이 맞춰질 때 발생한다. <br/>
  > focus 이벤트는 캡처링 단계에서는 발생할 수 있지만, 버블링 단계에서는 발생하지 않는다.
  >
  > 브라우저는 focus와 같은 특정 이벤트가 부모 요소로 전파되면서 예기치 않은 동작을 일으킬 수 있기 때문에, <br/>
  > 이를 방지하기 위해 focus 이벤트를 버블링하지 않도록 설계했다.
  > 1. 폼 필드의 중복 처리: 부모 요소가 여러 자식 요소의 focus 이벤트를 처리하려고 할 때, <br/>
  > 모든 포커스 변경이 부모 요소에 의해 처리되어 혼란을 초래할 수 있다.
  > 
  > 2. 이벤트 핸들러의 오작동: 특정 자식 요소에만 적용해야 하는 포커스 관련 로직이 부모 요소에도 적용되어 의도치 않은 동작을 유발할 수 있다.

- 부여된 JSX 태그 내에서만 실행되는 onScroll을 제외한 React 내의 모든 이벤트는 전파된다.

- 관습을 따르자면 “event”를 의미하는 e로 호출되는 것이 일반적.

- `e.stopPropagation()`를 호출하여 이벤트 전파를 멈출 수 있다.

  한 요소의 특정 이벤트를 처리하는 핸들러가 여러개인 상황에서, <br/>
  핸들러 중 하나가 버블링을 멈추더라도 나머지 핸들러는 여전히 동작한다.

  event.stopPropagation()은 위쪽으로 일어나는 버블링은 막아주지만, 다른 핸들러들이 동작하는 건 막지 못한다. <br/>
  버블링을 멈추고, 요소에 할당된 다른 핸들러의 동작도 막으려면 `event.stopImmediatePropagation()`을 사용.

  ```typescript
  const handleFirstClick = (event) => {
    console.log('First handler');
    event.stopImmediatePropagation(); // 다른 핸들러와 버블링을 막음
  };

  const handleSecondClick = (event) => {
    console.log('Second handler');
  };

  const handleParentClick = (event) => {
    console.log('Parent handler');
  };

  return (
    <div onClick={handleParentClick}>
      <button onClick={handleFirstClick} onClick={handleSecondClick}>
        Click Me
      </button>
    </div>
  );
  ```

- 사실 이벤트 버블링을 막아야 하는 경우는 거의 없다. <br/>
  버블링을 막아야하는 문제라면 핸들러의 event 객체에 데이터를 저장해 다른 핸들러에서 읽을 수 있게 하거나, 커스텀 이벤트를 사용해 해결할 수 있다.

  > #### 커스텀 이벤트<br/>
  > 
  > 커스텀 이벤트는 기본 DOM 이벤트 외에 사용자 정의 이벤트를 생성하고 디스패치(발생)하는 기능. <br/>
  > CustomEvent를 사용하여 커스텀 이벤트를 생성하고, dispatchEvent를 사용하여 이벤트를 발생시킬 수 있다. <br/> 
  > 이를 통해 특정 상황에서 이벤트 버블링을 대체하거나 필요한 경우 데이터를 전달할 수 있다.
  >
  > ```typescript
  > const triggerCustomEvent = () => {
  > const customEvent = new CustomEvent('customEvent', {
  >   detail: { message: 'Hello, World!' },
  >   bubbles: false, // 버블링 방지
  > });
  >
  > divRef.current.dispatchEvent(customEvent);
  > };
  >```

<br/>

> #### 이벤트 캡처링(capturing)<br/>
> 
> 캡처링이라는 이벤트 흐름이 존재한다. <br/>
> 실제 코드에서 자주 쓰이진 않지만(캡처링에 관한 코드를 발견하는 일은 거의 없음), 종종 유용한 경우가 있으므로 확인만 한다.
>
> 표준 DOM 이벤트에서 정의한 이벤트 흐름엔 3가지 단계가 존재.
> 
> 캡처링 단계 – 이벤트가 하위 요소로 전파되는 단계 <br/>
> 타깃 단계 – 이벤트가 실제 타깃 요소에 전달되는 단계 <br/>
> 버블링 단계 – 이벤트가 상위 요소로 전파되는 단계
> 
> 클릭하면 이벤트가 최상위 조상에서 시작해 아래로 전파되고(캡처링 단계) <br/>
> 이벤트가 타깃 요소에 도착해 실행된 후(타깃 단계), <br/>
> 다시 위로 전파(버블링 단계).
> 
> 캡처링 단계에서 이벤트를 잡아내려면 addEventListener의 capture 옵션을 true로 설정.
>
> ```typescript
> // 두가지 형태 모두 가능
> elem.addEventListener(..., {capture: true});
> elem.addEventListener(..., true);
> ```
> false이면(default 값) 핸들러는 버블링 단계에서 동작. <br/>
> true이면 핸들러는 캡처링 단계에서 동작.
> 
> 공식적으론 총 3개의 이벤트 흐름이 있지만, 이벤트가 실제 타깃 요소에 전달되는 단계인 ‘타깃 단계’는 별도로 처리되지 않는다. <br/>
> 캡처링과 버블링 단계의 핸들러는 타깃 단계에서 트리거된다. <br/>

- 각 핸들러는 아래와 같은 event 객체의 프로퍼티에 접근할 수 있습니다.

  event.target – 이벤트가 발생한 **가장 안쪽**의 요소(타깃 단계) <br/>
  event.currentTarget (=this) – 이벤트를 핸들링 하는 현재 요소 (핸들러가 실제 할당된 요소) <br/>
  event.eventPhase – 현재 이벤트 흐름 단계(캡처링=1, 타깃=2, 버블링=3)

<br/>

일부 브라우저 이벤트는 그와 관련된 기본 브라우저 동작을 가진다. <br/>
일례로 \<form>의 제출 이벤트는 그 내부의 버튼을 클릭 시 페이지 전체를 리로드하는 것이 기본 동작.

이러한 일이 발생하지 않도록 막기 위해 `e.preventDefault()`를 이벤트 오브젝트에서 호출할 수 있다.

> #### e.stopPropagation() vs. e.preventDefault()<br/>
> e.stopPropagation()은 이벤트 핸들러가 상위 태그에서 실행되지 않도록 멈춘다.<br/>
> e.preventDefault() 는 기본 브라우저 동작을 가진 일부 이벤트가 해당 기본 동작을 실행하지 않도록 방지한다.

<br/>
<br/>

## [State: 컴포넌트의 기억 저장소](https://ko.react.dev/learn/state-a-components-memory)

### 일반 변수로 충분하지 않은 경우

- React는 **컴포넌트별 메모리**를 state라고 부른다.

- 컴포넌트 내 useState hook이 아닌 지역변수의 사용이 문제가 될 수 있는 이유

  1. 지역 변수는 렌더링 간에 유지되지 않는다. 
  
  React는 이 컴포넌트를 두 번째로 렌더링할 때 지역 변수에 대한 변경 사항은 고려하지 않고 처음부터 렌더링. <br/>

  React 컴포넌트는 함수이기 때문에, 컴포넌트가 다시 렌더링될 때마다 함수가 다시 호출. <br/> 
  각 함수 호출은 독립적인 실행 컨텍스트를 가지므로, 함수 내에서 정의된 지역 변수는 각 호출마다 새로 생성.

  <br/>

  2. 지역 변수를 변경해도 렌더링을 일으키지 않는다.
  
  React는 새로운 데이터로 컴포넌트를 다시 렌더링해야 한다는 것을 인식하지 못한다.

  React의 상태 관리 및 렌더링 시스템은 useState 훅을 사용하여 상태 변화를 감지. <br/>
  useState를 통해 상태가 변경될 때, React는 해당 상태와 연결된 컴포넌트를 다시 렌더링.

- 따라서, 컴포넌트를 새로운 데이터로 업데이트하기 위해선 다음 두 가지가 필요하다.

  1. 렌더링 사이에 데이터를 유지.
  
  2. React가 새로운 데이터로 컴포넌트를 렌더링하도록 유발.

  `useState`는 이 조건을 만족한다.

<br/>

### state 변수 추가하기 

React에서 useState와 같이 “use”로 시작하는 다른 모든 함수를 hook이라고 한다. <br/>
훅은 React가 오직 렌더링 중일 때만 사용할 수 있는 특별한 함수. 이를 통해 다양한 React 기능을 “연결”한다.

hook은 컴포넌트의 최상위 수준 또는 커스텀 훅에서만 호출할 수 있다. <br/>
조건문, 반복문 또는 기타 중첩 함수 내부에서는 훅을 호출할 수 없다. <br/>
훅은 함수이지만 컴포넌트의 필요에 대한 무조건적인 선언으로 생각하면 도움이 된다.

<br/>

> #### React는 어떤 state를 반환할지 어떻게 알 수 있을까요? 
> 
> React가 규칙(“최상위 수준에서만 훅 호출”)을 따르면서, 컴포넌트의 모든 렌더링에서 훅을 호출하는 순서를 안정적으로 유지하기 때문.
> 
> 함수형 컴포넌트에서 훅을 항상 같은 순서로 호출하면, React는 상태가 어떤 훅과 연결되는지 추적할 수 있다. <br/>
> 이를 위해 React는 각 컴포넌트에 대해 상태 배열과 현재 인덱스를 유지한다.
> ```typescript
> import React, { useState } from 'react';
> 
> function Counter() {
>   const [count, setCount] = useState(0); // 첫 번째 useState 호출
>   const [name, setName] = useState('');  // 두 번째 useState 호출
> 
>   const increment = () => {
>     setCount(count + 1);
>   };
> 
>   return (
>     <div>
>       <p>{name}, you clicked {count} times</p>
>       <button onClick={increment}>Click me</button>
>       <input
>         type="text"
>         value={name}
>         onChange={(e) => setName(e.target.value)}
>       />
>     </div>
>   );
> }
> 
> export default Counter;
> ```
> React는 Counter 컴포넌트에 대해 [countState, nameState] 상태 배열을 유지하는 형태. <br/>
> 컴포넌트가 렌더링될 때 React는 인덱스를 0으로 설정하며, useState가 호출되면 React는 현재 인덱스(처음엔 0)의 상태를 반환하고, 인덱스를 증가시킴.

<br/>

### State는 격리되고 비공개로 유지됩니다 

- State는 화면에서 컴포넌트 인스턴스에 지역적. 
  
  동일한 컴포넌트를 두 번 렌더링한다면 각 복사본은 완전히 격리된 state를 가진다는 것. <br/>
  그중 하나를 변경해도 다른 하나에는 영향을 미치지 않는다.

  ```typescript
  import Gallery from './Gallery.js';

  export default function Page() {
    return (
      <div className="Page">
        <Gallery />
        <Gallery />
      </div>
    );
  }
  ```
  `<Gallery />` 컴포넌트를 두 번 렌더링했으므로 그들의 state는 별도로 저장된다.

<br/>
<br/>

## [렌더링 그리고 커밋](https://ko.react.dev/learn/render-and-commit)

주방에서 요리사가 컴포넌트를 재료로 맛있는 요리를 한다. <br/>
이 시나리오에서 **React**는 고객들의 요청을 받고 주문을 가져오는 **웨이터**.

UI를 요청하고 제공하는 세 가지 단계가 있다.

- 렌더링 트리거 (손님의 주문을 주방으로 전달)

- 컴포넌트 렌더링 (주방에서 주문 준비하기)

- DOM에 커밋 (테이블에 주문한 요리 내놓기)

<br/>

### 1단계: 렌더링 트리거 

1. 컴포넌트 초기 렌더링

    앱을 시작할 때 초기 렌더링을 트리거

<br/>

2. 컴포넌트 state가 업데이트

    컴포넌트가 처음으로 렌더링 된 후에는 set 함수를 통해 상태를 업데이트하여 추가적인 렌더링을 트리거. <br/> 
    컴포넌트의 상태를 업데이트하면 자동으로 렌더링 대기열에 추가.

<br/>

### 2단계: React 컴포넌트 렌더링 

“렌더링”은 React에서 컴포넌트를 호출하는 것.

재귀적 단계: 업데이트된 컴포넌트가 다른 컴포넌트를 반환하면 React는 다음으로 해당 컴포넌트를 렌더링하고,<br/>
해당 컴포넌트도 컴포넌트를 반환하면 반환된 컴포넌트를 다음에 렌더링. (top-down)

중첩된 컴포넌트가 더 이상 없고 React가 화면에 표시되어야 하는 내용을 정확히 알 때까지 이 단계가 지속.

> #### 성급하게 성능 최적화 진행하지 말 것.
> 
> 불필요한 복잡성 증가: 성급한 최적화는 코드를 더 복잡하게 만들 수 있다. <br/>
> 과도한 최적화: React는 이미 효율적인 렌더링 시스템을 가지고 있다. 많은 경우 추가적인 최적화가 필요하지 않을 수 있다. <br/>
> 잘못된 최적화: 병목 지점을 정확히 파악하지 않고 최적화를 하면, 오히려 성능을 저하시킬 수 있다. <br/>
> 가독성 저하: 성능 최적화를 위해 코드를 복잡하게 만들면, 완성되지 않은 코드의 가독성이 떨어질 수 있다.
>
> 먼저 애플리케이션을 개발하고 기능을 완성한 후, 성능 문제가 실제로 발생하는지 모니터링. <br/>
> 성능 문제가 확인되면, 프로파일링 도구를 사용해 정확한 병목 지점을 찾는다. <br/>
> 확인된 문제에 대해 targeted한 최적화를 수행.

<br/>

### 3단계: React가 DOM에 변경사항을 커밋 

컴포넌트를 렌더링(호출)한 후 React는 DOM을 수정.

초기 렌더링의 경우 React는 appendChild() DOM API를 사용하여 생성한 모든 DOM 노드를 화면에 표시한다. <br/>
리렌더링의 경우 React는 필요한 최소한의 작업(렌더링하는 동안 계산된 것!)을 적용하여 DOM이 최신 렌더링 출력과 일치하도록 한다.

React는 렌더링 간에 차이가 있는 경우에만 DOM 노드를 변경(Virtual DOM 사용). <br/>

<br/>

### 에필로그: 브라우저 페인트 

렌더링이 완료되고 React가 DOM을 업데이트한 후 브라우저는 화면을 다시 그린다. <br/>
이 단계를 “브라우저 렌더링”이라고 하지만 이 문서의 나머지 부분에서 혼동을 피하고자 여기서는 “페인팅”이라고 부를 것.

<br/>
<br/>

## [스냅샷으로서의 state](https://ko.react.dev/learn/state-as-a-snapshot)

**state는 스냅샷처럼 동작한다.**

<br/>

### 렌더링은 그 시점의 스냅샷을 찍습니다. 

“렌더링”이란 React가 컴포넌트(=함수)를 호출한다는 뜻. <br/>
해당 함수에서 반환하는 JSX는 시간상 UI의 스냅샷과 같다. <br/>
prop, 이벤트 핸들러, 로컬 변수는 모두 렌더링 당시의 state를 사용해 계산된다.

React가 컴포넌트를 다시 렌더링할 때, React가 (컴포넌트)함수를 다시 호출. <br/>
함수가 새로운 JSX 스냅샷을 반환하고, React가 함수가 반환한 스냅샷과 일치하도록 화면을 업데이트.

컴포넌트의 메모리로써 state는 함수가 반환된 후 사라지는 일반 변수와는 다르다. <br/>
state는 실제로 함수 외부에 마치 선반에 있는 것처럼 React 자체에 “존재”.

```typescript
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```
각각의 row에서 React는 `"다음 렌더링에서 number를 1로 변경할 준비를 합니다."`가 모두 동일하게 발동된다. <br/>
이것이 이벤트 핸들러가 완료된 후 React가 컴포넌트 안의 number를 3이 아닌 1로 다시 렌더링하는 이유.

<br/>

### 시간 경과에 따른 State 

```typescript
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        alert(number); <!-- alert가 먼저 동작 후, 새 state를 반영하는 리렌더링 수행 --> 
      }}>+5</button>
    </>
  )
}
```
```typescript
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setTimeout(() => {
          alert(number);
        }, 3000); <!-- 새 state를 반영하는 리렌더링 수행 후, alert가 동작 --> 
      }}>+5</button>
    </>
  )
}
```
```typescript
import { useState } from 'react';

export default function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Hello');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`You said ${message} to ${to}`); // 5초 내에 Alice를 Bob으로 바꿔도, Alice 출력.
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{' '}
        <select
          value={to}
          onChange={e => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}
```
비동기가 Queue 자료구조 내에서 동작한다고 이해하면 된다.

React는 렌더링의 이벤트 핸들러 내에서 state 값을 “고정”으로 유지. <br/>
코드가 실행되는 동안 state가 변경되었는지를 걱정할 필요가 없다.

하지만 다시 렌더링하기 전에 최신 state를 읽기 위해서는 다음 페이지에서 설명하는 state 갱신 함수를 사용한다.

<br/>
<br/>

## [State 업데이트 큐](https://ko.react.dev/learn/queueing-a-series-of-state-updates)

state 변수를 설정하면 다음 렌더링이 큐에 들어간다. <br/>
그러나 때에 따라 다음 렌더링을 큐에 넣기 전에, 값에 대해 여러 작업을 수행하고 싶을 때도 있다. <br/>
이를 위해서는 React가 state 업데이트를 어떻게 배치하면 좋을지 이해하는 것이 좋다.

<br/>

### React state batches 업데이트

```typescript
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```
이전 세션에서의 내용(각 렌더링의 state 값은 고정)에 추가로 한가지 요인이 더 있다. <br/>
React는 state 업데이트를 하기 전에 이벤트 핸들러의 모든 코드가 실행될 때까지 기다린다. <br/>
이 때문에 리렌더링은 모든 setNumber() 호출이 완료된 이후에만 일어난다.

setState가 이벤트 핸들러인 것이 아니고, onClick 내부에서 수행되는 함수 setState의 동기적 호출을 말하고 있는 것.

이렇게 하면 너무 많은 리렌더링이 발생하지 않고도 여러 컴포넌트에서 나온 다수의 state 변수를 업데이트할 수 있다. <br/>
하지만 이는 이벤트 핸들러와 그 안에 있는 코드가 완료될 때까지 UI가 업데이트되지 않는다는 의미이기도 하다. <br/>
**batching**라고도 하는 이 동작은 React 앱을 훨씬 빠르게 실행할 수 있게 해준다.

React는 클릭과 같은 여러 의도적인 이벤트에 대해 batch를 수행하지 않으며, 각 클릭은 개별적으로 처리된다.

<br/>

### 다음 렌더링 전에 동일한 state 변수를 여러 번 업데이트하기 

(흔한 사례는 아니지만)다음 렌더링 전에 동일한 state 변수를 여러 번 업데이트 하고 싶다면, <br/>
setNumber(number + 1) 대신, setNumber(n => n + 1)와 같이 이전 큐의 state를 기반으로 다음 state를 계산하는 함수를 전달. <br/> 

```typescript
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

| queued update | `n` | returns |
|---|---|---|
|`n => n + 1`|`0`|`0 + 1 = 1`|
|`n => n + 1`|`1`|`1 + 1 = 2`|
|`n => n + 1`|`2`|`2 + 1 = 3`|

<br/>
button컴포넌트를 다음의 형태로 교체한다면?

```typescript
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

| queued update | `n` | returns |
|---|---|---|
|"replace with 5"|`0` (unused)|`5`|
|`n => n + 1`|`5`|`5 + 1 = 6`|
|"replace with 42"|`6` (unused)|`42`|

<br/>

업데이터 함수 인수의 이름은 해당 state 변수의 첫 글자로 지정하는 것이 일반적이다.

```typescript
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```


<br/>
<br/>

## [객체 state 업데이트하기](https://ko.react.dev/learn/updating-objects-in-state)

### 변경이란?

리액트 state가 가진 객체를 직접 변경하면 안된다. <br/>
객체를 업데이트하고 싶을 때는 새로운 객체를 생성하여 (또는 기존 객체의 복사본을 만들어), state가 복사본을 사용하도록 한다.

```typescript
const [position, setPosition] = useState({ x: 0, y: 0 });
position.x = 5;

console.log(position) // { x: 5, y: 0 }
```

기술적으로 객체 자체 의 내용은 바꿀 수 있다. 이것을 "변경(mutation)"이라고 한다. <br/>
하지만 리액트 state의 객체들이 기술적으로 변경 가능할지라도, 숫자, 불리언, 문자열과 같이 불변성을 가진 것처럼 다루어야 한다.

<br/>

### 전개 문법으로 객체 복사하기 

새로 생성하는 객체에 존재하는 데이터를 포함하고 싶을 수 있다. <br/>
`spread("...")` 객체 전개 구문을 사용하면 모든 프로퍼티를 각각 복사하지 않아도 된다.

```typescript
const [person, setPerson] = useState({
  firstName: 'Barbara',
  lastName: 'Hepworth',
  email: 'bhepworth@sculpture.com'
});

setPerson({
  ...person, // 이전 필드를 복사
  firstName: e.target.value // 새로운 부분은 덮어쓰기
});
```

spread 전개 문법은 **얕은 복사**이다. 이것은 1 depth 깊이의 내용만 복사한다.

<br/>

### 중첩된 객체 갱신하기 

리액트에서는 state를 변경할 수 없는 것으로 다루어야 한다. <br/>
city를 바꾸기 위해서는 먼저 (이전 객체의 데이터로 생성된) 새로운 artwork 객체를 생성한 뒤, <br/>
그것을 가리키는 새로운 person 객체를 만들어야 한다.

```typescript
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});

const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

> #### 중첩된 객체는 없다.
> 
> ”중첩”은 객체의 동작에 대한 부정확한 생각. <br/>
> 코드가 실행될 때, “중첩된” 객체라는 것은 없다. 당신은 두 개의 다른 객체를 보는 것.
> ```typescript
> let obj = {
>   name: 'Niki de Saint Phalle',
>   artwork: {
>     title: 'Blue Nana',
>     city: 'Hamburg',
>     image: 'https://i.imgur.com/Sd1AgUOm.jpg',
>   }
>  };
> ```
> 이것은 사실
> ```typescript
> let obj1 = {
>   title: 'Blue Nana',
>   city: 'Hamburg',
>   image: 'https://i.imgur.com/Sd1AgUOm.jpg',
> };
> 
> let obj2 = {
>   name: 'Niki de Saint Phalle',
>   artwork: obj1
> };
> ```
> obj1 객체는 obj2 “안”에 있는 것이 아니다.

<br/>

### Immer로 간결한 갱신 로직 작성하기 

state가 깊이 중첩되어있다면 평탄화를 고려할 수 있다. <br/>
state 구조를 바꾸고 싶지 않다면, 중첩 전개할 수 있는 더 간편한 방법이 존재한다. 

Immer는 편리하고, 변경 구문을 사용할 수 있게 해주며 복사본 생성을 도와주는 인기 있는 라이브러리. <br/>
Immer를 사용하면 작성한 코드는 “법칙을 깨고” 객체를 변경하는 것처럼 보일 수 있다.

```typescript
updatePerson(draft => {
  draft.artwork.city = 'Lagos';
});
```

일반적인 '변경'과는 달리 이전 state를 덮어쓰지 않는다.

> #### Immer의 작동 원리
> 
> Immer가 제공하는 draft는 Proxy라고 하는 아주 특별한 객체 타입으로, 당신이 하는 일을 “기록”. <br/>
> 객체를 원하는 만큼 자유롭게 변경할 수 있는 이유. <br/>
> Immer는 내부적으로 draft의 어느 부분이 변경되었는지 알아내어, 변경사항을 포함한 완전히 새로운 객체를 생성한다.

<br/>
<br/>

## [배열 state 업데이트하기](https://ko.react.dev/learn/updating-arrays-in-state)

### 변경하지 않고 배열 업데이트하기

JavaScript에서 배열은 다른 종류의 객체. <br/>
객체와 마찬가지로 React state에서 배열은 읽기 전용으로 처리해야 한다. <br/>
즉 `arr[0] = 'bird'`처럼 배열 내부의 항목을 재할당해서는 안 되며, push()나 pop()같은 함수로 배열을 변경해서는 안된다.

대신 배열을 업데이트할 때마다 새 배열을 state 설정 함수에 전달해야 합니다. <br/>
이를 위해 state의 원본 배열을 변경시키지 않는 filter()와 map() 같은 함수를 사용하여 원본 배열로부터 새 배열을 만들 수 있다.

React state 내에서 배열을 다룰 때는 왼쪽 열에 있는 함수들의 사용을 피하는 대신 오른쪽 열에 있는 함수들을 선호해야 한다.

#### 배열 항목 추가

|| 선호되지 않음 (배열을 변경) | 선호됨 (새 배열을 반환) |
|---|---|---|
|추가|`push`, `unshift`|`concat, `[...arr]` 전개 연산자|
|제거|`pop`, `shift`, `splice`|`filter`, `slice`|
|교체|`splice`, `arr[i] = ...` 할당|`map`|
|정렬|`reverse`, `sort`|배열을 복사한 이후 처리|

또는 두 열의 함수를 모두 사용할 수 있도록 하는 Immer를 사용할 수 있다.

<br/>

#### 배열 항목 제거

배열에서 항목을 제거하는 가장 쉬운 방법은 필터링하는 것다. <br/>
다시 말해서 해당 항목을 포함하지 않는 새 배열을 제공하는 것.

```typescript
let initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];
const [artists, setArtists] = useState(
  initialArtists
);

setArtists(
  artists.filter(a => a.id !== artist.id)
);
```

<br/>

#### 배열 변환

배열의 일부 또는 전체 항목을 변경하고자 한다면, map()을 사용해 새로운 배열을 만들 수 있다. <br/>
map에 전달할 함수는 데이터나 인덱스(또는 둘 다)를 기반으로 각 항목을 어떻게 처리할지 결정할 수 있다.

```typescript
let initialShapes = [
  { id: 0, type: 'circle', x: 50, y: 100 },
  { id: 1, type: 'square', x: 150, y: 100 },
  { id: 2, type: 'circle', x: 250, y: 100 },
];
const [shapes, setShapes] = useState(
  initialShapes
);

const nextShapes = shapes.map(shape => {
  if (shape.type === 'square') {
    return shape;
  } else {
    return {
      ...shape,
      y: shape.y + 50,
    };
  }
})

setShapes(nextShapes);
```

<br/>

#### 배열 항목 교체

배열에서 하나 이상의 항목을 교체하는 경우가 특히 흔하다. <br/>
`arr[0] = 'bird'`와 같은 할당은 원본 배열을 변경시키므로, 이 경우에도 map을 사용하는 편이 좋다.

<br/>

#### 배열 항목 삽입

가끔은 시작도, 끝도 아닌 위치에 항목을 삽입하고 싶을 수 있다. <br/>
이를 위해, ... 배열 전개 구문과 slice() 함수를 함께 사용할 수 있다. <br/>

slice() 함수를 사용하면 배열의 “일부분”을 잘라낼 수 있다. <br/>
항목을 삽입하려면 삽입 지점 앞에 자른 배열을 전개하고, 새 항목과 원본 배열의 나머지 부분을 전개하는 배열을 만든다.

<br/>

#### 배열 기타 변경 적용

배열을 뒤집거나 정렬하고 싶을 수 있다. <br/>
JavaScript의 reverse() 및 sort() 함수는 원본 배열을 변경시키므로 직접 사용할 수 없다.

대신, 먼저 배열을 복사한 뒤 변경해야 한다.

배열을 복사하더라도 배열 내부 에 기존 항목을 직접 변경해서는 안된다. <br/>
이는 얕은 복사이기 때문에 복사한 새 배열에는 원본 배열과 동일한 항목이 포함된다. <br/>
따라서 복사된 배열 내부의 객체를 수정하면 기존 state가 변경된다.

```typescript
const nextList = [...list];
nextList[0].seen = true; // 문제: list[0]을 변경시킴.
setList(nextList);
```

nextList와 list는 서로 다른 배열이지만, nextList[0]과 list[0]은 동일한 객체를 가리킨다.

<br/>

### 배열 내부의 객체 업데이트하기

객체는 실제로 배열 “내부”에 위치하지 않는다. <br/> 
코드에서 “내부”로 나타낼 수 있지만 배열의 각 객체는 배열이 “가리키는” 별도의 값. <br/>
이것이 list[0]처럼 중첩된 필드를 변경할 때 주의해야 하는 이유입니다. 

중첩된 state를 업데이트할 때, 업데이트하려는 지점부터 최상위 레벨까지의 복사본을 만들어야 한다.

```typescript
const myNextList = [...myList];
const artwork = myNextList.find(a => a.id === artworkId);
artwork.seen = nextSeen; // 문제: 기존 항목을 변경시킴.
setMyList(myNextList);
```
```typescript
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // 변경된 새 객체를 만들어 반환
    return { ...artwork, seen: nextSeen };
  } else {
    return artwork;
  }
}));
```

map을 사용하면 이전 항목의 변경 없이 업데이트된 버전으로 대체할 수 있다. <br/>
이 접근 방식을 사용하면, 기존 state 항목이 변경되지 않고, 버그가 수정된다.

<br/>

#### Immer, 또 당신입니까?

```typescript
updateMyTodos(draft => {
  const artwork = draft.find(a => a.id === artworkId);
  artwork.seen = nextSeen;
});
```

이는 원본 state를 변경하는 것이 아니라, Immer에서 제공하는 특수 draft 객체를 변경. <br/>
마찬가지로 push()와 pop()같은 변경 함수들도 draft의 컨텐츠에 적용할 수 있다.

내부적으로 Immer는 항상 draft에서 수행한 변경 사항에 따라 처음부터 다음 state를 구성. <br/>
이렇게 하면 state를 변경하지 않고도 이벤트 핸들러를 매우 간결하게 유지할 수 있다.

<br/>
<br/>
