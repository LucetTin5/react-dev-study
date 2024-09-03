# 8주차. React Hooks (2)

## [useId](https://ko.react.dev/reference/react/useId)

`useId`는 접근성 어트리뷰트에 전달할 수 있는 고유 ID를 생성하기 위한 React Hook이다.

### 레퍼런스

> `useId`의 매개변수

- `useId`는 매개변수를 받지 않음

> `useId`의 반환값

- 특정 컴포넌트에서 사용할 수 있는 고유 ID 문자열을 반환

### 사용법

```js
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...
```

- 사용할 컴포넌트 최상단에서 호출해서 사용

- 생성된 ID를 다른 어트리뷰트로 전달할 수 있음

## [useImperativeHandle](https://ko.react.dev/reference/react/useImperativeHandle)

`useImperativeHandle`은 ref로 노출되는 핸들을 사용자가 직접 정의할 수 있게 해주는 React Hook이다.

### 레퍼런스

> `useImperativeHandle`의 매개변수

- `ref`: `forwardRef`에서 두번째 인자로 받은 ref

- `createHandle`: 노출하려는 ref 핸들을 반환하는 함수. 어떠한 유형이든 반환할 수 있음.

- optional `dependencies`: `createHandle` 코드 내에서 참조하는 모든 반응형 값을 나열한 목록

> `useImperativeHandle`의 반환값

- undefined 반환

### 사용법

```js
import { forwardRef, useRef, useImperativeHandle } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        focus() {
          inputRef.current.focus();
        },
        scrollIntoView() {
          inputRef.current.scrollIntoView();
        },
      };
    },
    []
  );

  return <input {...props} ref={inputRef} />;
});
```

이런식으로 `useImperativeHandle`를 통해 `createHandle`들을 작성해주면

```js
import { useRef } from "react";
import MyInput from "./MyInput.js";

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();

    // 이 작업은 DOM 노드가 노출되지 않으므로 작동 X
    // ref.current.style.opacity = 0.5;
  }

  return (
    <form>
      <MyInput placeholder="Enter your name" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

## [useInsertionEffect](https://ko.react.dev/reference/react/useInsertionEffect)

`useInsertionEffect`는 layout Effects가 실행되기 전에 전체 요소를 DOM 에 주입 할 수 있다.

### 레퍼런스

> `useInsertionEffect`의 매개변수

- `setup`: Effects의 로직이 포함된 함수. 컴포넌트가 DOM에 추가되기 전, layout effects가 실행되기 전 setup 함수 실행. `dependencies`가 변경되어 클린업 함수를 실행할 경우, 다음 새 값으로 setup 함수 실행

- optional `dependencies`: `setup` 코드 내에서 참조된 모든 반응형 값의 목록

> `useInsertionEffect`의 반환값

- undefined 반환

### 사용법

```js
import { useInsertionEffect } from "react";

// CSS-in-JS 라이브러리 안에서
function useCSS(rule) {
  useInsertionEffect(() => {
    // ... <style> 태그를 여기에서 주입하세요 ...
  });
  return rule;
}
```

## [useLayoutEffect](https://ko.react.dev/reference/react/useLayoutEffect)

- `useLayoutEffect`는 브라우저가 화면을 다시 그리기 전에 실행하는 `useEffect`이다.

### 레퍼런스

> `useLayoutEffect`의 매개변수

- `setup`: Effect 로직이 포함된 함수. (cleanup 함수 반환 가능)

- optional `dependencies`: `setup` 코드 내에서 참조된 모든 반응형 값의 목록

> `useLayoutEffect`의 반환값

- undefined 반환

### 사용법

```js
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);
  // ...
```

- 컴포넌트 최상위 레벨 또는 커스텀 Hook에서만 사용 가능. (반복문 및 조건문 X)

- Strict Mode에서 두번의 setup + cleanup 사이클 실행.

- 필요 이상으로 Effect가 다시 실행될 수 있으므로 불필요한 객체 의존성이나 함수 의존성은 최대한 제거하자.

- 클라이언트 환경에서만 동작(서버 렌더링 중에 실행 X)

- `useLayoutEffect` 내부의 코드와 이로 인한 모든 state 업데이트는 브라우저가 화면을 다시 그리는 것을 막음(속도 저하)

## [useMemo](https://ko.react.dev/reference/react/useMemo)

`useMemo`는 재렌더링 사이에 계산 결과를 캐싱할 수 있게 해주는 React Hook이다.

### 레퍼런스

> `useMemo`의 매개변수

- `calculateValue`: 캐싱하려는 값을 계산하는 함수

- `dependencies`: `calculateValue` 코드 내에서 참조된 모든 반응형 값들의 목록

> `useMemo`의 반환값

- 초기 렌더링에서는 인자 없이 `calculateValue`를 호출한 결과를 반환

- 이후 렌더링시 마지막 렌더링에서 저장된 값을 반환하거나(종속성 변경 X), `calculateValue`를 다시 호출하고 반환된 값을 저장

### 사용법

```js
import { useMemo } from "react";

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

- state와 ref를 사용하는 것이 더 적합할 수 있음

- 개별 JSX Node를 메모이제이션 할 수 있음

```js
export default function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  const children = useMemo(() => <List items={visibleTodos} />, [visibleTodos]);
  return <div className={theme}>{children}</div>;
}
```
