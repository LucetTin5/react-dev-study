# React - Suspense

- React 16 후반부 (16.6?7?~)에 도입된 기능, 컴포넌트의 렌더링을 특정 조건 만족시까지 중단시킬수 있게 함
- 주요 사용은 비동기 데이터 로딩 혹은 코드 스플리팅(lazy loading)상황

## 비동기 렌더링 과정에서

- 데이터 로딩 함수 -> `Promise`를 throw
- `React`는 가까운 `Suspense` 경계를 찾음
- `Suspense` 컴포넌트는 `fallback UI`를 렌더링
- `Promise`가 `resolve`되면 `React`는 원래의 컴포넌트를 렌더링

## Fiber와 Suspense

- `Suspense` 컴포넌트는 특수한 타입을 갖는 `Fiber Node`가 됨
- 자식 컴포넌트에서 `Promise`가 throw 시, React는 가까운 `Suspense`경계를 찾음
- `
