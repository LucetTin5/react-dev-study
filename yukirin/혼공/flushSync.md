아래 코드의 경우, 개발자는 state 갱신이 일어난 후, 스크롤 이벤트가 일어나길 바라겠지만, React는 그렇게 동작하지 않는다.
- state 갱신은 큐에 쌓여 비동기적으로 처리된다.
- 따라서, 스크롤 이벤트가 일어나는 시점엔 state가 갱신되기 이전이다. 
```javascript
setTodos([ ...todos, newTodo]);
listRef.current.lastChild.scrollIntoView();
```
이러한 경우 `react-dom` 패키지의 `flushSync`를 사용할 것을 고려해볼 수 있다.


## [flushSync](https://ko.react.dev/reference/react-dom/flushSync)란
`flushSync`는 **콜백의 모든 업데이트를 동기적으로 처리하도록 강제한다. 이는 DOM이 즉시 업데이트되는 것을 보장**한다.
```javascript
import { flushSync } from 'react-dom';

flushSync(callback) 
```

## flushSync의 주의사항
<img width="819" alt="스크린샷 2024-08-04 오후 9 26 20" src="https://github.com/user-attachments/assets/bba9d3d4-3745-4bd9-9d65-12b71c1866d5">

flushSync 공식문서에는 위와 같은 주의 문구를 포함하여 **전반적으로 사용을 주의**하고 있다. 위험 요인은 아래와 같다.

### 📛 주의사항 1. 성능 저하
`flushSync`를 사용하면 애플리케이션의 성능이 크게 저하될 수 있다.
- `flushSync`를 사용하면 callback 뿐만 아니라 큐에 있는 모든 비동기 업데이트를 즉시 처리한다.
- 즉, 짧은 시간에 많은 렌더링이 발생하게 되어, 애플리케이션이 느려질 수 있다. 따라서 가급적 사용하지 않도록 한다.

### 📛 주의사항 2. 예상치 못한 fallback
`flushSync`는 보류 중인 Suspense 바운더리의 `fallback` state를 표시하도록 강제할 수 있다.
- `<Suspense />`는 비동기적으로 데이터를 불러올 때 로딩 상태 등을 표시하기 위해 사용한다.
- flushSync를 사용하면 즉시 로딩 상태가 변경되어 Suspense의 **fallback UI가 예상보다 더 빨리 표시**될 수 있다.
```javascript
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function MyComponent() {
  const [show, setShow] = useState(false);
  const handleClick = () => flushSync(() => setShow(true));

  return (
    <div>
      <button onClick={handleClick}>Show Lazy Component</button>
      {show && (
        <Suspense fallback={<div>Loading...</div>}>
          <LazyComponent />
        </Suspense>
      )}
    </div>
  );
}
```
위 코드는 겉보기에는 잘 동작하는 것처럼 보이겠지만 아래와 같은 상황을 우려해볼 수 있다.
- 비동기 로드가 빠르다면, `setShow`가 즉시 실행되어 사용자는 안봐도 될 "Loading..." 메시지를 봐야한다.

### 📛 주의사항 3. 예상치 못한 Effect
`flushSync`는 보류 중인 Effect를 실행하고 반환되기 전에 포함된 모든 업데이트를 동기적으로 적용할 수 있다.
- 일반적으로 React의 `useEffect`와 같은 훅은 컴포넌트가 렌더링된 후 비동기적으로 실행된다.
- flushSync는 이러한 Effect들이 실행되기 전에, 모든 보류 중인 상태 업데이트가 동기적으로 처리한다.
```javascript
const [count, setCount] = useState(0);
const [message, setMessage] = useState('');

useEffect(() => {
  setMessage(`You clicked ${count} times`);
}, [count]);

function handleClick() {   // 여러 상태 업데이트를 동기적으로 처리
  flushSync(() => {
    setCount(prevCount => prevCount + 1);
  });
  flushSync(() => {
    setCount(prevCount => prevCount + 1);
  });
}
```
위 코드 역시 겉보기에는 잘 동작하는 것처럼 보이겠지만 아래와 같은 상황을 우려해볼 수 있다.
- flushSync로 인해 `setCount`가 2번 동기 처리 되고, 그로 인해 `useEffect` 또한 2번 처리 된다.
- 사용자는 +2가 된 `count`가 포함된 `message`를 볼 수도 있었지만 +1, +1 각각으로 깜빡이는 `message`를 봐야한다.

### 📛 주의사항 4. 보장할 수 없는 업데이트 순서
`flushSync`는 콜백 내부의 업데이트를 처리할 때 필요한 경우 콜백 외부의 업데이트를 처리할 수 있다.
- 예를 들어 클릭으로 보류 중인 업데이트가 있는 경우 React는 콜백 내부를 처리하기 전에 외부를 처리할 수 있다.
```javascript
function handleClick() {
  setMessage('Message updated from click'); // 비동기 처리
  flushSync(() => setCount(count + 1)); // 동기 처리
}
```
즉, 위 예시에서 `setMessage`랑 `setCount` 중 어떤 것이 먼저 처리될 지 보장할 수 없다는 것이다.
- `setMessage`는 비동기적으로 동작하기 때문에 큐에 쌓이고 다음 렌더링 사이클에서 처리된다.
- flushSync는 `setCount`를 즉시 업데이트 처리하기 위해 렌더링 사이클을 시작한다.
- 즉, flushSync로 인해 시작된 렌더링 사이클에서 예약된 업데이트들 중 어떤 것이 먼저 처리될 지 보장할 수 없다.

## flushSync는 언제 쓸까
사실 대부분의 경우는 flushSync를 쓰지 않고 원하는 동작을 구현할 수 있으며, flushSync는 많은 주의를 요한다. 그럼 flushSync는 언제 사용하는 것이 적절할까?

### 서드 파티 통합을 위한 업데이트 flushing 
브라우저 API 또는 UI 라이브러리와 같은 서드 파티 코드를 통합할 때 React가 업데이트를 처리하도록 강제할 필요가 있을 수 있다.
```javascript
const [isPrinting, setIsPrinting] = useState(false);

useEffect(() => {
  function handleBeforePrint() {
    flushSync(() => {
      setIsPrinting(true);
    })
  }
  function handleAfterPrint() {
    setIsPrinting(false);
  }

  window.addEventListener('beforeprint', handleBeforePrint);
  window.addEventListener('afterprint', handleAfterPrint);
}, []);
```
페이지를 프린트 출력하는 예제다. [여기](https://codesandbox.io/s/djmgw4?file=/src/App.js&utm_medium=sandpack)에서 전체 코드를 확인할 수 있다.
- `isPrinting`은 프린트 다이얼로그가 열리기 전에 페이지에 바로 반영되어야 할 state이다.
- 이러한 경우 flushSync를 사용하여 즉시 DOM에 반영되도록 하는 것이 적절하다고 할 수 있다.
