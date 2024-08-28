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

<br/>

# [useId](https://ko.react.dev/reference/react/useId)
접근성 어트리뷰트에 전달할 수 있는 고유 ID를 생성한다.
```javascript
const id = useId();
/*
const passwordHintId = useId();
return (
  <>
    <input type="password" aria-describedby={passwordHintId} />
    <p id={passwordHintId}>
  </>
)
*/
```
### 매개변수
- 어떠한 매개변수도 받지 않는다.
### 반환값
- `id`: `useId`를 호출한 특정 컴포넌트와 특정 `useId`에 관련된 고유 ID 문자열을 반환한다.
### 유용한 경우
- `aria-describedby`와 같은 HTML 접근성 어트리뷰트를 사용하면 두 태그가 연관되어 있음을 명시할 수 있다.
- 서버 렌더링과 클라이언트 hydration 과정에서 일관되고 고유한 ID를 생성하고 싶은 경우.

<br/>

# [useImperativeHandle](https://ko.react.dev/reference/react/useImperativeHandle)
[ref](https://ko.react.dev/learn/manipulating-the-dom-with-refs)로 노출되는 핸들을 사용자가 직접 정의할 수 있게 해준다.
```javascript
useImperativeHandle(ref, createHandle, dependencies?);
/*
const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() { inputRef.current.focus(); },
      scrollIntoView() { inputRef.current.scrollIntoView(); },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});
*/
```
### 매개변수
- `ref`: `forwardRef` 렌더링 함수에서 두 번째 인자로 받은 ref이다. (부모가 전달한 ref)
- `createHandle`: 인자가 없고 노출하려는 ref 핸들을 반환하는 함수.
  - 해당 ref 핸들은 어떠한 유형이든 될 수 있으며 일반적으로 노출하려는 메서드가 있는 객체를 반환.
- `dependencies`: `createHandle` 코드 내에서 참조하는 모든 반응형 값을 나열한 목록.
### 반환값
- `undefined`: `useEffect`는 `useImperativeHandle`를 반환한다.
### 유용한 경우
- 부모 컴포넌트에게 자식 DOM 노드의 전체 액세스 권한이 아닌 특정 메서드만 호출할 수 있도록 한다.
- 다양한 타입의 자식 element ref에 접근하여 액션을 지정하고 싶은 경우.

<br/>

# [useInsertionEffect](https://ko.react.dev/reference/react/useInsertionEffect)
layout Effects 가 실행되기 전에 전체 요소를 DOM에 주입한다.
```javascript
useInsertionEffect(setup, dependencies?);
/*
function useCSS(rule) { // CSS-in-JS 라이브러리 안에서
  useInsertionEffect(() => {
    // ... <style> 태그를 여기에서 주입하세요 ...
  });
  return rule;
}
*/
```
### 매개변수
- `setup`: Effect 로직이 포함된 함수. 컴포넌트가 DOM에 추가되기 전에, layout Effects 가 실행되기 전에, React는 `setup` 실행.
- `dependencies`: `setup` 코드 내에서 참조하는 모든 반응형 값을 나열한 목록.
### 반환값
- `undefined`: `useInsertionEffect`는 `undefined`를 반환한다.
### 유용한 경우
- 성능이 저하되지 않는 선에서 런타임에 `<style />` 태그를 주입하고 싶은 경우.

<br/>

# [useLayoutEffect](https://ko.react.dev/reference/react/useLayoutEffect)
브라우저가 화면을 다시 그리기 전에 실행되는 `useEffect`이다.
```javascript
useLayoutEffect(setup, dependencies?);
/*
const ref = useRef(null);
const [tooltipHeight, setTooltipHeight] = useState(0);

useLayoutEffect(() => {
  const { height } = ref.current.getBoundingClientRect();
  setTooltipHeight(height);
}, []);
*/
```
### 매개변수
- `setup`: Effect 로직이 포함된 함수. 컴포넌트가 DOM에 추가되기 전에, React는 `setup` 실행.
- `dependencies`: `setup` 코드 내에서 참조하는 모든 반응형 값을 나열한 목록.
### 반환값
- `undefined`: `useLayoutEffect`는 `undefined`를 반환한다.
### 유용한 경우
- 컴포넌트의 화면상 위치와 크기인 레이아웃 정보를 사용해서 컴포넌트를 렌더링하고자 하는 경우.

<br/>

# [useMemo](https://ko.react.dev/reference/react/useMemo)
리렌더링 사이에 계산 결과를 캐싱할 수 있게 해준다.
```javascript
const cachedValue = useMemo(calculateValue, dependencies);
/*
function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  // ...
}
*/
```
### 매개변수
- `calculateValue`: 캐싱하려는 값을 계산하는 함수. 순수해야 하며 인자를 받지 않고, 모든 타입의 값을 반환할 수 있어야 함..
  - 렌더링할 때, `dependencies` 변경되면, `calculateValue` 호출되어 새로운 값을 계산. 그렇지 않으면, 이전 저장 값 반환.
- `dependencies`: `calculateValue` 코드 내에서 참조하는 모든 반응형 값을 나열한 목록.
### 반환값
- `cachedValue`: 캐싱된 `calculateValue`를 호출한 결과를 반환.
### 유용한 경우
- 비용이 높은 로직이 재계산되는 것을 생략하여 성능 향상 하고 싶은 경우.
- `memo`로 감싸진 컴포넌트에 prop로 전달할 경우 값이 변경되지 않으면 렌더링 생략하고 싶을 때.
- 또 다른 Hook의 종속성으로 사용할 경우, 객체 자체를 메모제이션 하고 싶은 경우.
