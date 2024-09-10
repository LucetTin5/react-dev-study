[Hooks-1](https://cooing-dust-8b6.notion.site/Hooks-1-cff9d3421350467a8e770baa6332f0eb?pvs=4)

### useActionState (canary)

- useActionState(fn, initialState, permalink?) : 폼 액션 결과로 state를 업데이트할 수 있는 hook
  ```tsx
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
  ### 매개변수
  - fn : 폼이 제출되거나 버튼을 눌렀을 때 호출될 함수
  - initialState : 초기 state로 설정하고자 하는 값
  - 선택사항 permalink : 이 폼이 수정하는 고유의 URL이 포함된 문자열
  ### 반환값
  두 개의 배열을 반환
  [현재 state, form의 action prop이나 button의 formAction에 전달할 수 있는 새로운 액션]
  `const [state, formAction] = useActionState(fn, initialState, permalink?);`

### useCallback

- useCallback(fn, dependencies) : 함수를 캐싱하는 hook

  ```jsx
  import { useCallback } from 'react';

  export default function ProductPage({ productId, referrer, theme }) {
    const handleSubmit = useCallback((orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    }, [productId, referrer]);
  ```

  ### 매개변수

  fn : 캐싱할 함수값

  dependencies : fn 내에서 참조되는 모든 반응형 값의 목록. [Object.is](http://Object.is) 메서드를 통해 의존성 목록이 변화했는지 판단

  ### 반환값

  최초 렌더링 : 전달한 fn 함수

  후속 렌더링 : 이미 저장한 fn 함수 반환 or 의존성 목록 변했을 시 현재 렌더링에서 전달한 fn 함수

- 자바스크립트에서 `function (){}` 나 `() =>{}` 는 항상 다른 함수를 생성. `{}` 객체 리터럴이 항상 새로운 객체를 생성하는 방식과 유사

  ```jsx
  function ProductPage({ productId, referrer, theme }) {
    // React에게 리렌더링 간에 함수를 캐싱하도록 요청합니다...
    const handleSubmit = useCallback((orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    }, [productId, referrer]); // ...이 의존성이 변경되지 않는 한...

    return (
      <div className={theme}>
        {/* ...ShippingForm은 같은 props를 받게 되고 리렌더링을 건너뛸 수 있습니다.*/}
        <ShippingForm onSubmit={handleSubmit} />
      </div>
    );
  ```

  리렌더링 건너뛰기 같은 성능 최적화 용도로만 `useCallback` 사용할 것!

- useMemo와 useCallback의 차이

  - useMemo : 함수 호출 결과값을 캐싱
  - useCallback : 함수 자체를 캐싱

  ```jsx
  // (React 내부의) 간단한 구현
  function useCallback(fn, dependencies) {
    return useMemo(() => fn, dependencies);
  }
  ```

- useEffect 안에서 함수를 호출해야할 때 useCallback으로 감싸줄 수 있다. 굳이?
  하지만 그냥 함수를 effect안에서 선언하면 문제해결
      ```jsx
      function ChatRoom({ roomId }) {
        const [message, setMessage] = useState('');

        const createOptions = useCallback(() => {
          return {
            serverUrl: 'https://localhost:1234',
            roomId: roomId
          };
        }, [roomId]); // ✅ roomId가 변경될 때만 변경됩니다.

        useEffect(() => {
          const options = createOptions();
          const connection = createConnection();
          connection.connect();
          return () => connection.disconnect();
        }, [createOptions]); // ✅ createOptions가 변경될 때만 변경됩니다.
        // ...

      =>

      function ChatRoom({ roomId }) {
        const [message, setMessage] = useState('');

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

### useContext

- useContext(SomeContext) : 컴포넌트에서 context를 읽고 구독하는 hook
  ```tsx
  import { useContext } from 'react';

  function MyComponent() {
    const theme = useContext(ThemeContext);
    // ...
  ```
  ### 매개변수
  - SomeContext : `createContext` 로 생성한 context
  ### 반환값
  - 호출하는 컴포넌트에 대한 context value를 반환. 트리에서 호출하는 컴포넌트 위의 가장 가까운 `SomeContext.Provider` 에 전달된 value로 결정. provider가 없으면 createContext에 전달한 `defaultValue`
  - 컴포넌트 내의 provider는 고려하지 않음

### useDebugValue

- useDebugValue(value, format?) : devtool에서 커스텀 훅에 라벨을 추가해주는 hook
  ```tsx
  import { useDebugValue } from "react";

  function useOnlineStatus() {
    // ...
    useDebugValue(isOnline ? "Online" : "Offline");
    // ...
  }
  ```
  ### 매개변수
  - `value` : devtools에 표시하고 싶은 값
  - 선택사항 `format` : 포매팅 함수. 옵셔널이므로 없어도됨
    ```tsx
    useDebugValue(date, (date) => date.toDateString());
    ```
  ### 반환값
  반환값은 없음

### useDeferredValue

- useDeferredValue(value, initialValue?) : UI 일부 업데이트를 지연시키는 hook
  ```tsx
  import { useState, useDeferredValue } from "react";

  function SearchPage() {
    const [query, setQuery] = useState("");
    const deferredQuery = useDeferredValue(query);
    // ...
  }
  ```
  ### 매개변수
  - value : 지연시키려는 값
  - `실험적기능` optional initialValue : 첫 렌더 시 사용될 값
  ### 반환값
  - `currentValue` : 지연된 값 (state 업데이트를 지연)
  값을 지연시키는 것
  디바운싱 : 타이핑을 멈출 때까지 기다렸다가 목록을 업데이트하는 것
  스로틀링 : 가끔씩 목록을 업데이트하는 것

### useEffect

- useEffect(setup, dependencies?) : 외부시스템과 컴포넌트를 동기화하는 hook
  ```tsx
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
  ### 매개변수
  - setup(설정) : Effect 로직이 포함된 함수
  - 선택사항 dependencies : 함수 코드 내부에서 참조되는 모든 반응형 값들이 포함된 배열. [Object.is](http://Object.is) 비교법을 통해 이전값과 비교
  ### 반환값
  useEffect는 `undefined`를 반환
  setup 함수 내에서 `설정코드` 와 `정리코드` 를 선언할 수 있음.
  setup의 return 에 정리코드를 반환하면 된다.
  - 외부 시스템의 예시
    - setInterval() , clearInterval() 등의 타이머
    - window.addEventListner(), window.removeEventListener() 등의 이벤트 구독
    - animation.start() , animation.reset() 등의 서드파티 애니메이션 라이브러리 API
    - connect, disconnet 등의 채팅서버 연결
