# Hooks pt.1

## useActionState

• 정의: `React 19 RC`, `React Canary`에서 사용 가능한 새로운 Hook
• 목적: `Server Action`과 `Server Component` 맥락에서 클라이언트 측 상태 관리
• `Server Action`, `Server Components`는 `React 19 RC`, `React Canary`에서 사용 가능

### 주요 기능

• `Server Action`의 실행 상태를 클라이언트에서 관리 및 추적
• 서버 측 작업의 진행 상태, 결과, 오류 처리
• 클라이언트와 서버 간 상태 동기화 단순화

### 사용 방식

```jsx
const [state, dispatch] = useActionState(serverAction, initialState);
```

- serverAction: 서버에서 정의된 비동기 함수
- initialState: 초기 상태 값
- state: 현재 상태 - 최초 렌더링 시 initialState와 동일, 이후 액션의 반환값
- dispatch: 서버 액션 트리거 및 상태 업데이트 함수

### 주요 특징

1. `Server Action` 통합
   • `Server Action` 직접 호출 및 결과 관리
   • 서버 사이드 로직과 클라이언트 사이드 UI 업데이트 연결

2. 상태 관리
   • 액션 실행 상태 자동 추적 (idle, loading, success, error)
   • 로딩 인디케이터, 에러 메시지 구현 용이

3. 최적화
   • 불필요한 리렌더링 방지
   • 서버 응답 효율적 처리

4. `Server Component` 상호작용
   • `Server Component`의 액션을 클라이언트 컴포넌트에서 사용 가능
   • 서버-클라이언트 간 데이터 흐름 단순화

5. 에러 처리
   • 서버 액션 실행 중 에러 자동 캐치 및 상태 반영
   • 견고한 에러 처리 및 사용자 피드백 메커니즘 구축 가능

6. 타입 안정성
   • TypeScript 사용 시 서버 액션의 반환 타입 자동 추론

### 사용 예시

```jsx
// 서버 컴포넌트
async function fetchUserData(userId) {
  // 데이터베이스에서 사용자 정보 가져오기
}

// 클라이언트 컴포넌트
function UserProfile({ userId }) {
  const [userState, fetchUser] = useActionState(fetchUserData, null);

  useEffect(() => {
    fetchUser(userId);
  }, [userId]);

  if (userState.status === 'loading') return <LoadingSpinner />;
  if (userState.status === 'error')
    return <ErrorMessage error={userState.error} />;
  if (userState.status === 'success') return <UserInfo user={userState.data} />;

  return null;
}
```

---

## useCallback

• 정의: 메모이제이션된 콜백을 반환하는 Hook

### 주요 기능

• 특정 함수 메모이제이션으로 불필요한 리렌더링 방지
• 의존성 배열 변경 시에만 새 함수 생성

### 사용 방식

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

### 주요 특징

• 자식 컴포넌트에 콜백 전달 시 주로 사용
• useMemo와 유사하나 함수 메모이제이션에 특화
• 의존성 배열 관리 중요

### /w Vanilla JS

```jsx
function createHooks(callback) {
  let hookIndex = 0; // 현재 Hook의 인덱스를 추적

  const useMemo = (fn, deps = []) => {
    const index = hookIndex++; // 현재 Hook의 인덱스
    const oldDependencies = memos[index]?.dependencies; // 이전 의존성 배열

    // 의존성이 변경되었는지 확인
    if (
      !oldDependencies ||
      oldDependencies.some((dependency, i) => !Object.is(deps[i], dependency))
    ) {
      // 의존성이 변경되었거나 처음 호출된 경우, 새 값 계산
      memos[index] = {
        value: fn(), // 메모이제이션된 값 계산
        dependencies: deps, // 새 의존성 배열 저장
      };
    }

    return memos[index].value; // 메모이제이션된 값 반환
  };

  // useCallback은 useMemo를 이용해 구현
  // 함수 자체를 메모이제이션
  const useCallback = (fn, deps = []) => useMemo(() => fn, deps);
}
```

---

## useContext

• 정의: React 컴포넌트 트리 내 전역 데이터 공유 Hook

### 주요 기능

• 컴포넌트 트리 전체에 데이터 제공
• props drilling 문제 해결

### 사용 방식

```jsx
const value = useContext(MyContext);
```

### 주요 특징

• 가장 가까운 상위 <MyContext.Provider>의 value prop 읽기
• 컴포넌트 재사용성 고려 필요

### Context와 리렌더링

• Context 값 변경 시 리렌더링 동작:

- Context를 사용(consume)하는 모든 하위 컴포넌트가 리렌더링됨
- 단순히 Provider 내부에 있는 것만으로는 리렌더링되지 않음
- useContext를 호출하거나 Class 컴포넌트의 contextType/Consumer를 사용하는 컴포넌트만 영향 받음

• 최적화 전략:

1. 컨텍스트 분할:

   - 자주 변경되는 값과 그렇지 않은 값을 별도의 컨텍스트로 분리

   ```jsx
   const ThemeContext = React.createContext(null);
   const UserContext = React.createContext(null);
   ```

2. 메모이제이션 사용:

   - React.memo로 불필요한 리렌더링 방지

   ```jsx
   const MemoizedChild = React.memo(ChildComponent);
   ```

3. 컨텍스트 값 최적화:
   - 객체 대신 원시 값 사용 고려
   - 객체 필요 시 useMemo로 안정화
   ```jsx
   const value = useMemo(() => ({ user, theme }), [user, theme]);
   ```

### 성능 고려사항

• 대규모 애플리케이션에서 Context 남용 시 성능 저하 가능
• 필요한 경우에만 Context 사용, 가능하면 컴포넌트 합성(composition) 고려
• Context 값이 자주 변경되는 경우, 상태 관리 라이브러리 (예: Redux, MobX) 사용 검토

---

## useDebugValue

• 정의: React DevTools에서 사용자 정의 Hook 레이블 표시 Hook

### 주요 기능

• 사용자 정의 Hook 디버깅 지원
• `React DevTools`가 설치 되어 있을때만 의미 있음
• DevTools에 추가 정보 표시
• 커스텀 Hook의 상위 레벨에서

### 사용 방식

```jsx
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);
  // ...
  useDebugValue(isOnline ? 'Online' : 'Offline');
  return isOnline;
}
```

### 주요 특징

• 개발 모드에서만 작동
• 복잡한 값 표시 시 포맷 함수 사용 가능
• 프로덕션 빌드에서는 자동으로 제거됨

---

## useDeferredValue -- 미완 --

• 정의: UI의 긴급하지 않은 부분 렌더링 지연 Hook

### 주요 기능

• 성능 최적화를 위한 값 업데이트 지연
• 중요 UI 업데이트 우선 처리

### 사용 방식

```jsx
const deferredValue = useDeferredValue(value);
```

### 주요 특징

• Concurrent 모드에서 효과적 작동
• useEffect, useLayoutEffect와 유사한 내부 메커니즘
• 디바운싱, 스로틀링과 유사하나 React 렌더링 메커니즘에 최적화

---

## useEffect

• 정의: 함수 컴포넌트에서 부수 효과 수행 Hook

### 주요 기능

• 컴포넌트 렌더링 후 작업 수행
• 데이터 페칭, 구독 설정, DOM 수동 조작 등 수행

### 사용 방식

```jsx
useEffect(() => {
  // 부수 효과 수행
  return () => {
    // 정리(clean-up) 함수 (선택적)
  };
}, [dependency1, dependency2]);
```

### 주요 특징

• 의존성 배열 변경 시 effect 재실행
• 빈 의존성 배열 사용 시 마운트 시에만 실행
• 의존성 배열 생략 시 매 렌더링마다 실행
• 정리 함수로 구독 해제 등 수행 가능
• 브라우저 화면 그린 후 비동기적 실행

## Dependency Array

• `useEffect`, `useCallback` 등의 훅은 `dependency array`를 가지고 있으며, array 변화를 추적한다.
• 이 때 react의 dependency array 변화는 react의 패키지 중 `shared`에 포함된 아래를 통해 진행
• 기본적으로 `Object.is`를 사용하기에 얕은 비교임을 알 수 있음

```js
/**
 * Copyright (c) Meta Platforms, Inc. and affiliates.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 *
 * @flow
 */

/**
 * inlined Object.is polyfill to avoid requiring consumers ship their own
 * https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
 */
function is(x: any, y: any) {
  return (
    (x === y && (x !== 0 || 1 / x === 1 / y)) || (x !== x && y !== y) // eslint-disable-line no-self-compare
  );
}

const objectIs: (x: any, y: any) => boolean =
  // $FlowFixMe[method-unbinding]
  typeof Object.is === 'function' ? Object.is : is;

export default objectIs;
```
