# Hooks pt.3

## useOptimistic

- UI 낙관적 업데이트를 위한 훅으로 현재는 `React Canary`에만 존재

### 주요 기능

- 서버 응답과 관련된 처리 시, 응답을 기다리지 않고 UI를 즉시 업데이트
- UX 향상을 위한 즉각 피드백을 제공하기 위해 이용

### 사용 방식

```jsx
const [optimisticState, addOptimistic] = useOptimistic(state, updateFn);
```

#### Return Values

- `optimisticState`: 낙관적인 결과 상태값
- `addOptimistic`: 낙관적 업데이트를 트리거하는 함수

#### Params

- `state`: 초기에 반환할 값
- `updateFn`: 현재 상태와 액션을 받아 새로운 낙관적 상태를 반환하는 함수

### 주요 특징

- 응답 결과가 크리티컬하지 않은 경우 빠른 반응을 보여줄 수 있음
- 실제 서버 응답에 따른 상태 조정 또한 가능

### 사용 예시

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (currentTodos, newTodo) => [...currentTodos, newTodo]
  );

  const handleAddTodo = async (text) => {
    const newTodo = { id: Date.now(), text, completed: false };
    addOptimisticTodo(newTodo);

    try {
      const result = await submitTodo(newTodo);
      setTodos([...todos, result]);
    } catch (error) {
      // 오류 발생 시 낙관적 업데이트 롤백
      setTodos(todos);
    }
  };

  return (
    <ul>
      {optimisticTodos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

### tanstack/react-query useMutation을 이용한 낙관적 업데이트와의 비교

useOptimistic을 사용한 좋아요 기능

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function LikeButton({ postId, initialLikes }) {
  const queryClient = useQueryClient();

  const mutation = useMutation(() => likePost(postId), {
    onMutate: async () => {
      await queryClient.cancelQueries(['post', postId]);
      const previousPost = queryClient.getQueryData(['post', postId]);
      queryClient.setQueryData(['post', postId], (old) => ({
        ...old,
        likes: old.likes + 1,
      }));
      return { previousPost };
    },
    onError: (err, newTodo, context) => {
      queryClient.setQueryData(['post', postId], context.previousPost);
    },
    onSettled: () => {
      queryClient.invalidateQueries(['post', postId]);
    },
  });

  return (
    <button onClick={() => mutation.mutate()}>
      좋아요 ({mutation.data?.likes ?? initialLikes})
    </button>
  );
}
```

### Server Actions와의 결합

React 18+ Server Components, Server Actions와 함께 사용할 때 보다 적합 (Next는 13+에서 지원)

#### Server Actions?

Server Actions는 서버에서 실행되는 비동기 함수로, 클라이언트에서 직접 호출할 수 있습니다. 이를 통해 데이터 변경 로직을 서버 사이드로 옮길 수 있다.

#### useOptimistic + Server Actions

```jsx
// app/actions.js
'use server';

export async function addTodo(newTodo) {
  // 데이터베이스에 todo 추가
  await db.todos.create(newTodo);
  // 업데이트된 todo 리스트 반환
  return await db.todos.findMany();
}

// app/page.js
import { useOptimistic } from 'react';
import { addTodo } from './actions';

export default function TodoList({ initialTodos }) {
  const [todos, setTodos] = useState(initialTodos);
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, newTodo]
  );

  async function handleAddTodo(e) {
    e.preventDefault();
    const newTodo = {
      id: Date.now(),
      text: e.target.text.value,
      completed: false,
    };

    // 낙관적 업데이트
    addOptimisticTodo(newTodo);

    // 서버 액션 호출
    const updatedTodos = await addTodo(newTodo);

    // 실제 서버 상태로 업데이트
    setTodos(updatedTodos);
  }

  return (
    <div>
      <form onSubmit={handleAddTodo}>
        <input name="text" />
        <button type="submit">Add Todo</button>
      </form>
      <ul>
        {optimisticTodos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

## useReducer

- 복잡한 상태 로직을 관리하기 위한 useState의 대안 Hook

### 주요 기능

- 여러 하위 값을 포함하는 복잡한 상태 로직 관리
- 다음 상태가 이전 상태에 의존적인 경우 유용

### 사용 방식

```jsx
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

#### Return Values

- `state`: 현재 상태
- `dispatch`: 액션을 발생시키는 함수

#### Params

- `reducer`: (state, action) => newState 형태의 리듀서 함수
- `initialArg`: 초기 상태 또는 초기 상태를 생성하는 함수에 전달할 인자
- `init`: (선택적) 초기 상태를 생성하는 함수

### 주요 특징

- Redux 패턴과 유사한 상태 관리 방식
- 상태 업데이트 로직을 컴포넌트 외부로 분리 가능
- 복잡한 상태 전환을 명확하게 표현

### 사용 예시

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error('알 수 없는 액션 타입');
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  );
}
```

## useRef

- 컴포넌트의 생명주기 동안 유지되는 가변값을 저장하는 Hook

### 주요 기능

- DOM 요소에 직접 접근
- 렌더링에 영향을 주지 않는 값 저장

### 사용 방식

```jsx
const refContainer = useRef(initialValue);
```

#### Return Value

- `refContainer`: `{ current: initialValue }` 형태의 변경 가능한 ref 객체

#### Params

- `initialValue`: ref 객체의 초기값 (선택적)

### 주요 특징

- .current 프로퍼티로 값에 접근/수정
- 값이 변경되어도 리렌더링 발생하지 않음
- 컴포넌트가 언마운트될 때까지 동일한 객체 유지

### 사용 예시

```jsx
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>입력란에 포커스</button>
    </>
  );
}
```

## useState

- 함수형 컴포넌트에서 상태를 추가하는 Hook

### 주요 기능

- 컴포넌트에 로컬 상태 추가
- 상태 업데이트 시 컴포넌트 리렌더링

### 사용 방식

```jsx
const [state, setState] = useState(initialState);
```

#### Return Values

- `state`: 현재 상태 값
- `setState`: 상태를 업데이트하는 함수

#### Params

- `initialState`: 초기 상태 값 또는 초기 상태를 반환하는 함수

### 주요 특징

- 함수형 업데이트 지원 (이전 상태를 기반으로 새 상태 계산)
- 지연 초기화 지원 (초기 상태를 계산하는 함수 전달 가능)

### 사용 예시

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <>
      <p>클릭 횟수: {count}</p>
      <button onClick={() => setCount((prevCount) => prevCount + 1)}>
        클릭
      </button>
    </>
  );
}
```

## useSyncExternalStore

- 외부 저장소를 구독하고 강제로 동기화하는 Hook

### 주요 기능

- 외부 데이터 소스와 React 컴포넌트 상태 동기화
- 동시성 기능과 호환되는 외부 상태 관리

### 사용 방식

```jsx
const state = useSyncExternalStore(subscribe, getSnapshot[, getServerSnapshot]);
```

#### Return Value

- `state`: 외부 저장소의 현재 값

#### Params

- `subscribe`: 외부 저장소를 구독하는 함수
- `getSnapshot`: 외부 저장소의 현재 값을 반환하는 함수
- `getServerSnapshot`: (선택적) 서버 렌더링 시 사용할 스냅샷을 반환하는 함수

### 주요 특징

- 외부 상태 변경 시 컴포넌트 자동 리렌더링
- 서버 사이드 렌더링 지원
- Tearing 현상 방지

### 사용 예시

```jsx
const store = {
  state: { count: 0 },
  listeners: new Set(),
  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  },
  getSnapshot() {
    return this.state;
  },
  increment() {
    this.state = { count: this.state.count + 1 };
    this.listeners.forEach((listener) => listener());
  },
};

function Counter() {
  const state = useSyncExternalStore(store.subscribe, store.getSnapshot);
  return (
    <>
      <p>카운트: {state.count}</p>
      <button onClick={() => store.increment()}>증가</button>
    </>
  );
}
```

## useTransition

- UI를 차단하지 않고 상태를 업데이트하는 Hook

### 주요 기능

- 긴 렌더링 작업 중 UI 응답성 유지
- 상태 업데이트의 우선순위 조정

### 사용 방식

```jsx
const [isPending, startTransition] = useTransition();
```

#### Return Values

- `isPending`: 전환 중인지 나타내는 boolean 값
- `startTransition`: 우선순위가 낮은 상태 업데이트를 시작하는 함수

### 주요 특징

- 우선순위가 낮은 상태 업데이트를 표시
- 사용자 상호작용을 차단하지 않음
- 전환 중 로딩 상태 표시 가능

### 사용 예시

```jsx
function SearchResults() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    setQuery(e.target.value);
    startTransition(() => {
      // 무거운 필터링 작업
      setFilteredResults(heavyComputation(e.target.value));
    });
  };

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending ? (
        <p>결과 업데이트 중...</p>
      ) : (
        <ResultsList results={filteredResults} />
      )}
    </>
  );
}
```
