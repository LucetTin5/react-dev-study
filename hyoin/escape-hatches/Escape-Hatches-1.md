[탈출구(useRef) - 노션 링크](https://cooing-dust-8b6.notion.site/Escaping-Hatches-useRef-c7f63e7b388e422d886b8b1dbc2a98e8?pvs=4)

- **useRef**
- useEffect
- customHook

### Ref로 값 참조하기

일부 정보를 기억하고 싶지만, 렌더링을 유발하고 싶지않을 때 `useRef` hook 사용

1. 외부 API와 통신해야할 때
2. DOM 엘리먼트 액세스 할때

- Refs는 렌더링에 사용되지 않는 값을 고정하기 위한 escape hatch

- 컴포넌트에 ref 추가 `const ref = useRef(initialValue);` , 인자로 초기값 전달

  ```tsx
  import { useRef } from "react";

  const ref = useRef(initialValue);

  // ref는 다음과 같은 객체를 반환
  {
    current: initialValue;
  }

  // ref.current로 접근 가능
  ```

  - ref와 state 비교
    | ref | state |
    | ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
    | useRef(initialValue) 는 { current: initialValue } 을 반환합니다. | useState(initialValue) 은 state 변수의 현재 값과 setter 함수 [value, setValue] 를 반환합니다. |
    | state를 바꿔도 리렌더 되지 않습니다. | state를 바꾸면 리렌더 됩니다. |
    | Mutable-렌더링 프로세스 외부에서 current 값을 수정 및 업데이트할 수 있습니다. | ”Immutable”—state 를 수정하기 위해서는 state 설정 함수를 반드시 사용하여 리렌더 대기열에 넣어야 합니다. |
    | 렌더링 중에는 current 값을 읽거나 쓰면 안 됩니다. | 언제든지 state를 읽을 수 있습니다. 그러나 각 렌더마다 변경되지 않는 자체적인 state의 snapshot이 있습니다. |
  - ref는 일반적인 자바스크립트 객체처럼 동작

- DOM 엘리먼트에 액세스할 때 사용 가능

  ```tsx
  import { useRef } from 'react';

  const myRef = useRef(initialValue);

  ...
  <div ref={myRef}/>
  ...
  // myRef.current = div 엘리먼트
  ```

  위와 같은 ref 어트리뷰트에 ref를 전달하면 dom 엘리먼트를 `myRef.current`에 넣음
  엘리먼트가 DOM에서 사라질 경우 myRef를 `null`로 업데이트 해준다.

### Ref로 DOM 조작하기

- ref로 노드 가져오기 (import, 선언, ref attribute에 전달)

  ```tsx
  import { useRef } from 'react';

  const myRef = useRef(initialValue);
  ...
  <div ref={myRef}/>

  // myRef.current = div 엘리먼트
  ```

  이후 이벤트 핸들러에서 DOM 노드에 접근하거나, 내장 브라우저 API 사용 가능하다.

  ex ) `myRef.current.scrollIntoView();`

- ref 콜백을 사용해 ref 리스트 관리하기
  ref attribute에 함수를 전달 ( ref 콜백 ) 하면 여러 ref를 관리할 수 있다.
  콜백의 인자 node에 대해… 알아보면 좋을듯!

  ```tsx
  import { useRef, useState } from "react";

  export default function CatFriends() {
    const itemsRef = useRef(null);
    const [catList, setCatList] = useState(setupCatList);

    function scrollToCat(cat) {
      const map = getMap();
      const node = map.get(cat);
      node.scrollIntoView({
        behavior: "smooth",
        block: "nearest",
        inline: "center",
      });
    }

    function getMap() {
      if (!itemsRef.current) {
        // 처음 사용하는 경우, Map을 초기화합니다.
        itemsRef.current = new Map();
      }
      return itemsRef.current;
    }

    return (
      <>
        <nav>
          <button onClick={() => scrollToCat(catList[0])}>Tom</button>
          <button onClick={() => scrollToCat(catList[5])}>Maru</button>
          <button onClick={() => scrollToCat(catList[9])}>Jellylorum</button>
        </nav>
        <div>
          <ul>
            {catList.map((cat) => (
              <li
                key={cat}
                ref={(node) => {
                  const map = getMap();
                  if (node) {
                    // Map에 노드를 추가합니다
                    map.set(cat, node);
                  } else {
                    // Map에서 노드를 제거합니다
                    map.delete(cat);
                  }
                }}
              >
                <img src={cat} />
              </li>
            ))}
          </ul>
        </div>
      </>
    );
  }

  function setupCatList() {
    const catList = [];
    for (let i = 0; i < 10; i++) {
      catList.push("https://loremflickr.com/320/240/cat?lock=" + i);
    }

    return catList;
  }
  ```

- 다른 컴포넌트의 DOM 노드 접근하기( `forwardRef` )

  브라우저 요소 출력하는 내장 컴포넌트 (`<input/>` )에 ref 주입할 시, ref는 current 프로퍼티를 DOM 노드로 설정

  but 커스텀 컴포넌트에 ref 주입할 시에는 기본적으로 `null`

  ```tsx
  import { useRef } from "react";

  function MyInput(props) {
    return <input {...props} />;
  }

  export default function MyForm() {
    const inputRef = useRef(null);

    function handleClick() {
      inputRef.current.focus();
    }

    return (
      <>
        <MyInput ref={inputRef} />
        <button onClick={handleClick}>Focus the input</button>
      </>
    );
  }
  ```

  `MyForm` 에서 설정한 ref 가 MyInput에 주입 ⇒ `MyInput` 내의 Input에 props로 넘겨지는 구조

  but 다른 컴포넌트의 ref에 접근하는 것이기 때문에 Error 출력된다!

  이 경우에는 forwardRef API를 컴포넌트를 선언해 해결.

  ```tsx
  const MyInput = forwardRef((props, ref) => {
    return <input {...props} ref={ref} />;
  });
  ```

  하지만 이 경우는 current를 통해 DOM 입력 요소가 그대로 노출 됨.

  focus만 전달하고 싶을 경우 `useImperataiveHandle`을 통해 노출되는 기능을 제한할 수 있음

  ```tsx
  import { forwardRef, useRef, useImperativeHandle } from "react";

  const MyInput = forwardRef((props, ref) => {
    const realInputRef = useRef(null);
    // realInputRef는 실제 input의 돔노드를 가지고 있음
    useImperativeHandle(ref, () => ({
      // 오직 focus만 노출하도록 부모 컴포넌트에게 전달
      focus() {
        realInputRef.current.focus();
      },
    }));
    return <input {...props} ref={realInputRef} />;
  });

  export default function Form() {
    const inputRef = useRef(null);
    // 여기서 inputRef는 DOM 노드가 아닌 useImperativeHandle 호출로 직접 구성한 객체를 받음

    function handleClick() {
      inputRef.current.focus();
    }

    return (
      <>
        <MyInput ref={inputRef} />
        <button onClick={handleClick}>Focus the input</button>
      </>
    );
  }
  ```

- React가 ref를 부여할 때
  React 갱신 단계
  1. 렌더링 : 화면에 그려야하는 컴포넌트들을 호출 ( 계산 )
  2. 커밋 : 변경사항을 DOM에 적용
     ⇒ 그 후 브라우저 페인팅
     렌더링 과정에서 ref에 접근하는 것은 권장되지 않는다.
     ref.current를 커밋단계에서 설정하기 때문
     대부분 이벤트 핸들러 안에서 ref 접근이 일어나고, 가끔 useEffect를 사용
     `flushSync` 를 이용해 DOM 변경을 동기적으로 수행하도록 변경할 수 있다.

### Ref callback function

https://ko.react.dev/reference/react-dom/components/common#ref-callback

- ref 어트리뷰트에 객체 대신 함수를 전달하는 경우 `<div ref={(node) => console.log(node)} />`

- div DOM 노드가 화면에 추가될 때, `node` 를 파라미터로 사용해 ref 콜백을 호출
- 해당 div DOM 노드가 제거되면 `null` 을 파라미터로 사용해 ref 콜백을 호출

- return : ref가 분리될 때 일어나는 cleanup function을 반환
  반환 값이 없으면 null을 인자로하는 콜백을 한번 더 호출

```tsx
<li
  key={cat.id}
  ref={node => {
    const map = getMap();
    if (node) {
      // Map에 노드를 추가합니다
      map.set(cat, node);
    } else {
      // Map에서 노드를 제거합니다
      map.delete(cat);
    }
  }}
>

//node가 있을 때, null일 때 조건문 처리

<li
  key={cat.id}
  ref={node => {
    const map = getMap();
    // Add to the Map
    map.set(cat, node);

    return () => {
      // Remove from the Map
      map.delete(cat);
    };
  }}
/>
// cleanup function 처리
```
