# 7주차. React Hooks

## [useActionState](https://ko.react.dev/reference/react/useActionState)

_`useActionState` 훅은 현재 React Canary 채널에서만 사용 가능하다. (React DOM에 소속)_

`useActionState`는 폼 액션의 결과를 기반으로 state를 업데이트하는 Hook이다.

### 레퍼런스

> `useActionState(action, initialState, permalink?)`의 매개변수

- `action`: 폼이 제출될 때 호출되는 함수. 첫번째 인수로 폼의 이전 state를 전달. 초기에는 `initialState`를 전달하고, 이후에는 이전 실행의 반환값을 전달한다. 그 후 일반적으로 폼 액션에 전달하는 인수들이 이어짐.

- `initialState`: 초기 state로 설정화하는 값. 직렬화(Serialization) 가능한 값이면 가능. 첫 액션 이후로 무시된다.

- optional `permalink`: 이 폼이 수정하는 고유의 URL을 포함한 문자열. 만약 `action`이 서버 액션(`use server`)이고 JS 번들이 로드되기 전에 폼이 제출될 경우 `permalink`로 redirect 해준다.

> `useActionState`의 반환값

총 두 개의 값이 담긴 배열을 반환.

- 현재 state

- `form` 컴포넌트의 `action`이나 폼 내부 `button`의 `formAction` prop에 전달할 수 있는 새로운 액션

### 사용법

```js
import { useActionState } from "react";

async function increment(previousState, formData) {
  return previousState + 1;
}

function StatefulForm({}) {
  const [state, formAction] = useActionState(increment, 0);
  return (
    <form>
      {state}
      <button formAction={formAction}>Increment</button>
    </form>
  );
}
```

## [useCallBack](https://ko.react.dev/reference/react/useCallback)

`useCallback`은 리렌더링 간에 함수 정의를 캐싱해 주는 React Hook이다.

### 레퍼런스

> `useCallBack(fn, dependencies)`의 매개변수

- `fn`: 캐싱할 함수값. 첫 렌더링시 이 함수를 반환해주고, 렌더링마다 `dependencies`를 비교해 함수를 반환.

- `dependencies`: `fn` 내에서 참조되는 모든 반응형 값의 목록. props와 state, 그리고 컴포넌트 안에서 직접 선언된 모든 변수와 함수를 포함.

> `useCallBack`의 반환값

- 최초 렌더링에서는 전달한 `fn` 함수를 그대로 반환

- 후속 렌더링에서는 이전 렌더링에서 이미 저장해두었던 `fn` 함수를 반환하거나, 현재 렌더링 중에 전달한 `fn` 함수를 반환.

### 사용법

> 컴포넌트의 리렌더링 생략

```js
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  // ...
```

- 최초 렌더링에서 useCallback으로부터 반환되는 함수는 호출시에 전달할 함수

- 이어지는 렌더링에서 React는 의존성을 이전 렌더링에서 전달한 의존성과 비교. 의존성 중 하나라도 변한 값이 없다면, useCallback은 전과 똑같은 함수를 반환

- 그렇지 않으면 useCallback은 이번 렌더링에서 전달한 함수를 반환

- **_자바스크립트에서 function () {} 나 () => {}은 항상 다른 함수를 생성함 -> 매번 새로운 function이 생성_**

> Memoized 콜백에서 상태 업데이트 하기

```js
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []);
  // ...
```

- 최소한의 의존성을 갖도록 업데이트 함수 내부에서 상태 처리

> Effect가 너무 자주 실행되는 것 방지

```js
// 방법1. useEffect 내부 함수를 useCallback으로 감사기

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ roomId가 변경될 때만 변경

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // ✅ createOptions가 변경될 때만 변경
  // ...
```

```js
// 방법2. 함수 의존성 제거하기
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('')

  useEffect(() => {
    function createOptions() { // ✅ useCallback이나 함수 의존성이 필요하지 않습니다.
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ roomId가 변경될 때만 변경됩니다.
  // ...
```

> Custom Hook 최적화하기

- 커스텀 훅을 작성하는 경우, 반환하는 모든 함수를 `useCallback`으로 감싸는 것이 좋음

## [useContext](https://ko.react.dev/reference/react/useContext)

`useContext`는 컴포넌트에서 context를 읽고 구독할 수 있는 React Hook이다.

### 레퍼런스

> `useContext(SomeContext)`의 매개변수

- `SomeContext`: `createContext`로 생성한 context. context 자체는 정보를 담고 있지 않으며, 컴포넌트에서 제공하거나 읽을 수 있는 정보의 종류를 나타냄.

> `useContext`의 반환값

- 호출하는 컴포넌트에 대한 context 값을 반환

- 해당 값은 트리에서 호출하는 컴포넌트 위의 가장 가까운 `SomeContext.Provider`에 전달된 value로 결정.

- provider가 없으면 변환된 값은 해당 context에 대해 `createContext`에 전달한 `defaultValue`가 됨.

### 사용법

> 트리의 깊은 곳에 데이터 전달하기

```js
function MyPage() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  // ... 내부에서 버튼을 렌더링합니다. ...
}

function Button() {
  const theme = useContext(ThemeContext);
  // ...
}
```

`useContext`는 전달한 `context`에 대한 `context value`를 반환

`useContext`는 항상 호출하는 컴포넌트 상위에서 가장 가까운 provider를 찾음

> Context를 통해 전달된 데이터 업데이트 하기

```js
import { createContext, useContext, useState } from "react";

const CurrentUserContext = createContext(null);

export default function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);
  return (
    <CurrentUserContext.Provider
      value={{
        currentUser,
        setCurrentUser,
      }}
    >
      <Form />
    </CurrentUserContext.Provider>
  );
}

function Form({ children }) {
  return (
    <Panel title="Welcome">
      <LoginButton />
    </Panel>
  );
}

function LoginButton() {
  const { currentUser, setCurrentUser } = useContext(CurrentUserContext);

  // ...
  return (
    <Button
      onClick={() => {
        setCurrentUser({ name: "Advika" });
      }}
    >
      Log in
    </Button>
  );
}
```

> fallback 기본값 지정

```js
// const ThemeContext = createContext(null);

// 'light' 기본값 지정
const ThemeContext = createContext("light");
```

## [useDebugValue](https://ko.react.dev/reference/react/useDebugValue)

`useDebugValue`는 React DevTools에서 커스텀 훅에 라벨을 추가할 수 있게 해주는 React Hook이다.

### 레퍼런스

> `useDebugValue(value, format?)`의 매개변수

- `value`: React DevTools에 표시하고 싶은 값. 어떤 타입이든 가능.

- optional `format`: 포맷팅 함수. 컴포넌트가 검사될 때, React DevTools는 value를 인자로 포맷팅 함수를 호출하고, 포맷팅 함수가 반환한 포맷팅 된 값을 표시함.

> `useDebugValue(value, format?)`의 반환값

- 반환값이 없음.

### 사용법

특정 Hook의 최상위 레벨에서 호출

```js
export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(
    subscribe,
    () => navigator.onLine,
    () => true
  );
  useDebugValue(isOnline ? "Online" : "Offline");
  return isOnline;
}
```

이후 React Devtool의 hooks에서 해당 state의 값을 확인할 수 있다.

추가적으로 두번째 인자인 `format`으로 포맷팅 함수를 전달할 수 있다.

```js
useDebugValue(date, (date) => date.toDateString());
```

## [useDeferredValue](https://ko.react.dev/reference/react/useDeferredValue)

### 레퍼런스

`useDeferredValue`는 UI 일부 업데이트를 지연시킬 수 있는 React Hook이다.

> `useDeferredValue(value, initialValue)`의 매개변수

- `value`: 지연시키려는 값. 모든 타입을 가질 수 있음. 단, 문자열 및 숫자와 같은 원시값이거나, 컴포넌트의 외부에서 생성된 객체여야만 함.

- **_(Canary Only)_** optional `initialValue`: 컴포넌트의 초기 렌더링 동안 사용할 값. 옵션이 생략될 경우 `useDeferredValue`는 초기 렌더링 동안 값을 지연시키지 않음.

> `useDeferredValue`의 반환값

- 초기에는 사용자가 제공한 `initialValue`와 같음.

- 업데이트가 발생할 경우 새 값과 일치하도록 업데이트

### 사용법

```js
import { Suspense, useState, useDeferredValue } from "react";
import SearchResults from "./SearchResults.js";

export default function App() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

이렇게 `<Suspense/>` 안에 넣어두면 `fallback` 대신 기존 `value(query)` 값을 보여줄 수 있어서 사용성이 향상되게 된다.

추가적으로 다음과 같은 코드를 이용해 콘텐츠가 오래되었는지(기존 value와 다른 값을 받아왔을 경우) 확인할 수 있다.

```js
const isStale = query !== deferredQuery;
```

## [useEffect](https://ko.react.dev/reference/react/useEffect)

`useEffect`는 외부 시스템과 컴포넌트를 동기화하는 React Hook이다.

### 레퍼런스

> `useEffect(setup, dependencies?)`의 매개변수

- `setup(설정)`: Effect의 로직이 포함된 함수. React는 컴포넌트가 DOM에 추가된 이후에 설정 함수를 실행. 선택적으로 clean up 함수를 반환할 수 있음. 컴포넌트가 DOM에서 제거될 때 clean up 함수 실행.

- optional `dependencies`: `setup` 함수의 코드 내부에서 참조되는 모든 반응형 값들이 포함된 배열로 구성

> `useEffect`의 반환값

- `undefined`를 반환

### 사용법

```js
import { useEffect } from "react";
import { createConnection } from "./chat.js";

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```
