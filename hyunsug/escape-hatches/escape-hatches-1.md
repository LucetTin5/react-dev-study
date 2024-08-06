# 탈출구 - 1

## useRef에 대하여

useRef는 렌더링에 영향을 끼치지 않으면서 정보를 보존하는 데 이용된다.

```jsx
import { useRef } from "react";

...
const ref = useRef();
...
```

와 같이 사용된다

## useRef의 리턴타입

React에서 사용되는 ref의 타입은 크게 `LegacyRef`와 `MutableRefObject`, `RefObject`로 나눌 수 있다.

- MutableRefObject는 `const ref = useRef<T>(initialValue: T)`로 선언될 때와 같이, null을 명시하지 않을 경우의 타입이다.
  - MutableRefObject는 DOM 요소가 아닌 값을 참조하도록 하는 것이 일반적
  - 반환된 ref는 `{ current: T }`
- RefObject는 `const ref = useRef<T | null>(null)`과 같이, null로 초기화하면서 T | null로 타입을 지정할 때 생성된다.
- createRef를 통해 생성된 ref 또한 `RefObject<T>` 타입을 갖는다.
- 위 두 경우의 `RefObject<T>`는 `{ readonly current: T | null }`
- readonly이지만 런타임 시점에서는 타입스크립트가 아니라 JS의 실행이기에 React는 이 점을 넘어서 DOM이 렌더링 된 후 current에 DOM 노드를 할당함

- `LegacyRef<T>`는 여러 타입을 포함하는 유니온 타입이다
  - string (deprecated)
  - `RefCallback<T>`: 함수형 ref 콜백
  - `RefObject<T>`
  - null
  - undefined
- useRef를 null 초기화하지 않고 엘리먼트에 부여할 때 타입 에러가 발생하는 부분은 LegacyRef에 MutableRefObject를 부여할 때 발생한다.

## useRef의 동작?

리액트의 useRef는 다음과 같이 구현되어있다.

```ts
function useRef<T>(initialValue: T): { current: T } {
  const dispatcher = resolveDispatcher();
  return dispatcher.useRef(initialValue);
}
```

컴포넌트 내부에서 useRef가 실행될 때는 어떤 일이 일어날까?

`useRef(initialValue)`

1. React는 컴포넌트의 렌더링을 시작한다.
2. 훅이 사용되고 있는 컴포넌트의 렌더링은 `renderWithHooks`를 통해 진행된다. (`react-reconciler` 패키지에 위치)
3. 실행 중에 `ReactSharedInternals.H`라는 값에 `HooksDispatcherOnMount`, `HooksDispatcherOnUpdate`를 부여하는 로직이 존재한다.
   - 이는 각각 최초 마운트 단계에서의 Dispatcher, 업데이트를 할 때의 Dispatcher를 부여한다.
   - `react` 패키지(리액트 코어)의 `ReactCurrentDispatcher.current`와 동일하다.
   - 이 부여 과정에서는 `__DEV__` 단계 확인, current 부여된 값이 있는지, current.memoizedState가 있는지 등을 확인하여 진행한다.
4. `renderWithHooks` 내부에서 실제 컴포넌트의 함수가 호출 `useRef(initialValue)`
5. Mount / Update로 조금 다른 동작을 진행
   - Mount 시(HooksDispatcherOnMount)
     - 새로운 ref 객체를 생성
     - 초기값을 ref.current에 할당
     - 이 ref 객체는 hook.memoizedState에 저장
     - ref 객체를 반환
   - Update 시(HOoksDispatcherOnUpdate)
     - 기존 hook 객체를 가져옴
     - 이전에 저장된 ref 객체(hook.memoizedState)를 반환
     - 이 때 초기값은 무시함

## 주요 사용 사례

1. DOM 요소에 대한 참조

   ```jsx
   const inputRef = useRef(null);
   // ...
   <input ref={inputRef} />;
   ```

   - DOM 요소에 직접 접근할 때 유용 (예: focus(), measure 등)

2. 이전 값 저장

   ```jsx
   const prevCountRef = useRef();
   useEffect(() => {
     prevCountRef.current = count;
   });
   ```

   - 이전 렌더링의 값을 기억하고 싶을 때 사용

3. 컴포넌트 내부에서 변경 가능한 값 저장

   ```jsx
   const timerRef = useRef(null);
   useEffect(() => {
     timerRef.current = setInterval(() => {
       // 작업 수행
     }, 1000);
     return () => clearInterval(timerRef.current);
   }, []);
   ```

   - 렌더링에 영향을 주지 않는 값을 저장할 때 유용

## 주의사항

1. ref.current의 변경은 리렌더링을 트리거하지 않음

   - 상태 업데이트와 달리, ref 값 변경은 컴포넌트를 다시 렌더링하지 않음

     ```jsx
     function Example() {
       const ref = useRef(0);

       // 렌더링 중 ref 변경
       ref.current++;

       return <div>Count: {ref.current}</div>;
     }
     ```

2. 렌더링 중 ref.current 읽거나 쓰지 않기

   - 렌더링 결과의 일관성을 해칠 수 있음, 아래 예시는 렌더링마다 서로 다른 결과를 나타내게 됨
   - 컴포넌트의 렌더링 결과는 props와 state에만 의존하여 진행하는 것을 기본으로 하기에 리액트의 최적화에 방해요소가 됨

   ```jsx
   function Counter() {
     const ref = useRef(0);
     // 문제의 코드
     return <div>{ref.current++}</div>;
   }
   ```

## ref 콜백 함수

DOM 요소(node)가 마운트/언마운트될 때 호출되는 함수를 제공하는 방식
useRef는 hook.memoizedState에 `{ current: initialValue }`를 저장하는 방식이라면,
ref 콜백함수는 컴포넌트의 마운트/언마운트 단계에서 함수를 호출한다.

- 함수는 node의 마운트 시 node를 인자로 받고, 언마운트 시 null을 인자로 받는다.
- ref의 설정 시점을 제어할 수 있음
- 조건부 ref 설정 혹은 ref 설정 시 추가 로직의 실행이 가능

```jsx
import React, { useCallback, useState } from 'react';

function MeasureExample() {
  const [height, setHeight] = useState(0);

  // 여기서 useCallback을 사용하지 않고 넘기면 렌더링마다 새 함수가 생성되
  const measuredRef = useCallback((node) => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}
```

여기서 ref함수를 타입을 부여해보자면 다음과 같다.

```ts
type RefCallback<T> = (instance: T | null) => void;
```

## forwardRef

forwardRef는 React 컴포넌트가 받은 ref를 자식 컴포넌트로 전달할수 있게 하는 함수

- 컴포넌트 내부의 특정 DOM 요소에 접근해야 할 필요가 있는 경우가 존재
- 전달받은 ref를 대상 요소에 전달하여 부모가 자식의 특정 컴포넌트를 참조할 수 있게 됨
- 여기서 전닳하는 ref는 RefObject일수도, ref callback일수도 있음

```tsx
const Component = forwardRef<HTMLInputElement, ComponentProps>(props, ref) => {
   return <div>
      <input ref={ref} />
   </div>
}
```

## 여러 개의 ref가 필요할 경우

단순하게 useRef를 여러번 선언하는 경우가 아니라, 하나의 컴포넌트에 여러개의 ref가 부여되어야 하는 경우를 생각해봅시다
일반적인 케이스는 분명 아니지만, 외부 라이브러리를 사용함에 있어 부여해야하는 ref가 있고
나의 구현에 맞게 ref를 부여해야하는 상황이 동시에 발생한다고 생각해보면 될 것 같습니다

그럼 우리는 여러개의 ref를 함께 ref={}에 넘겨줘야 합니다.

```ts
import { useCallback, MutableRefObject, LegacyRef, RefCallback } from 'react';

type ReactRef<T> = MutableRefObject<T> | LegacyRef<T>;

function mergeRefs<T>(...refs: ReactRef<T>[]): RefCallback<T> {
  return (instance: T | null) => {
    refs.forEach((ref) => {
      if (typeof ref === 'function') {
        ref(instance);
      } else if (ref !== null) {
        (ref as MutableRefObject<T | null>).current = instance;
      }
    });
  };
}

function useMergeRefs<T>(...refs: ReactRef<T>[]) {
  return useCallback(mergeRefs(...refs), refs);
}
```
