# [useActionState](https://ko.react.dev/reference/react/useActionState)
폼 액션의 결과를 기반으로 state를 업데이트할 수 있도록 한다.
```javascript
import { useActionState } from "react";

const [state, formAction] = useActionState(fn, initialState, permalink?);
```
### 매개변수
- `fn`: 폼 제출 시 호출될 함수. 함수의 첫번째 인수로 이전 state를 전달.
  - 보통 `action(currentState, formData)`와 같이 생겼으며 두번째 인수는 제출된 폼 데이터.
- `initialState`: 초기 state로 설정하고자 하는 값.
- `permalink`: 폼이 수정하는 고유의 URL이 포함된 문자열.
  - 특정 게시물을 수정하는 폼이라면 그 게시물의 고유 URL을 가리킨다.
  - 자바스크립트 번들 로드 전 제출되면 브라우저는 현재의 페이지 URL이 아닌 명시된 `permalink`의 URL로 이동.
### 반환값
- `state`: 현재 state. 초기에는 `initialState`였다가 액션이 실행되면 액션에서 반환한 값.
- `formAction`: `<form />`의 `action` prop에 전달하거나 내부 `<button />`의 formAction prop에 전달하는 새로운 액션.

<br/>

# [useCallback](https://ko.react.dev/reference/react/useCallback)
리렌더링 간에 함수 정의를 캐싱하도록 한다.
```javascript
const cachedFn = useCallback(fn, dependencies);
/*
const handleSubmit = useCallback((orderDetails) => {
  post('/product/' + productId + '/buy', {referrer, orderDetails});
}, [productId, referrer]);
*/
```
### 매개변수
- `fn`: 캐싱할 함수. 이 함수는 필요에 따라 인자나 반환값도 가질 수 있다. 
  - useCallback 훅에서 React는 이 **함수를 호출하는 것이 아닌 반환**한다.
  - 다음 렌더링에서 `dependencies` 같으면 저장했던 같은 함수 반환, 다르면 이번 렌더링에서 전달된 함수 반환.
- `dependencies`: `fn` 내에서 참조되는 모든 반응형 값의 목록.
### 반환값
- `fn`: 캐시된 `fn` 함수를 반환한다.
### 유용한 경우
- 리렌더링 성능 최적화를 위해 자식 컴포넌트에 넘기는 함수를 캐싱할 때.
- Effect dependency인 함수로 인해 Effect가 너무 자주 실행될 때. (하지만 **함수를 Effect 내부에서 선언하여 dependency를 제거하는 것이 더 좋음.**)
- 커스텀 Hook을 작성하는 경우 반환하는 모든 함수를 `useCallback`으로 감싸는 것이 좋음.

<br/>

# [useContext](https://ko.react.dev/reference/react/useContext)
컴포넌트에서 [context](https://ko.react.dev/learn/passing-data-deeply-with-context)를 읽고 구독할 수 있도록 한다.
```javascript
import { SomeContext } from "../store";

const value = useContext(SomeContext);
```
### 매개변수
- `SomeContext`: `createContext`로 생성한 context다.
  - context 자체는 정보를 담고 있지 않으며, 컴포넌트에서 제공하거나 읽을 수 있는 정보의 종류를 나타낸다. 
### 반환값
- `context`: 호출하는 컴포넌트에 대한 context 값을 반환한다. 이 값은 가장 가까운 `Provider`에 전달된 value로 결정된다.

<br/>

# [useDebugValue](https://ko.react.dev/reference/react/useDebugValue)
[React DevTools](https://ko.react.dev/learn/react-developer-tools)에서 커스텀 훅에 라벨을 추가할 수 있게 해준다.
```javascript
useDebugValue(value, format?);
/*
function useOnlineStatus() {
  // ...
  useDebugValue(isOnline ? 'Online' : 'Offline');
  // ...
}
*/
```
### 매개변수
- `value`: React DevTools에 표시하고 싶은 값을 나타낸다. 어떤 타입이든 될 수 있다.
- `format`: 컴포넌트 검사 시, React DevTools이 `value`를 인자로 호출하는 포맷팅 함수로, 이 함수의 반환 값이 표시된다.
### 유용한 경우
- 컴포넌트 내에서 어떤 Hook을 사용하고 있는지 헷갈리는 경우 디버깅 할 때.

<br/>

# [useDeferredValue](https://ko.react.dev/reference/react/useDeferredValue)
상태 값 변화에 낮은 우선순위를 지정하여 UI의 일부 업데이트를 지연시킬 수 있게 한다.
```javascript
const currentValue = useDeferredValue(value, initialValue?);
/*
function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query); // query를 낮은 우선순위로 상태 변경
  // ...
}
*/
```
### 매개변수
- `value`: 지연시키려는 값으로, 모든 타입을 가질 수 있다.
  - 초기 렌더링 중에 `currentValue`은 사용자가 제공한 `value`와 일치하나, 업데이트가 발생하면 뒤쳐진다.
- `initialValue`: 구성 요소의 초기 렌더링 중 사용하여, 지연시키도록 한다. (Canary only)
### 반환값
- `currentValue`: `value`가 변경되면 `currentValue`는 다른 상태변화가 다 발생하고 가장 나중에 변경된다.
  - `useDeferredValue`로 인한 백그라운드 리렌더링은 화면에 커밋될 때까지 Effects를 실행하지 않는다.
### 유용한 경우
- `<Suspense />`와 함께 새 콘텐츠가 로딩되는 동안 오래된 콘텐츠를 표시하는 경우.
- UI 리렌더링을 의도적으로 지연해서 성능을 최적화하고 싶을 때.

<br/>

# [useEffect](https://ko.react.dev/reference/react/useEffect)
외부 시스템과 컴포넌트를 동기화한다.
```javascript
useEffect(setup, dependencies?);
/*
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => connection.disconnect();
}, [serverUrl, roomId]);
*/
```
### 매개변수
- `setup`: Effect의 로직이 포함된 함수로, 내부에서 선택적으로 claenup 함수를 반환할 수 있다.
  - React는 컴포넌트가 DOM에 추가된 이후에 `setup` 함수를 실행하며 DOM이 제거된 경우에도 실행한다.
- `dependencies`: `setup` 함수 내부에서 참조되는 모든 반응형 값들이 포함된 배열로 구성된다. 
### 반환값
- `undefined`: `useEffect`는 `undefined`를 반환한다.
