# Components (React 내장 컴포넌트)

- React는 특별한 기능을 제공하는 몇 가지 내장 컴포넌트를 제공함
- 일반적인 JSX 태그처럼 사용할 수 있으며, React 앱에서 구조적으로 혹은 성능면에서 기능을 제공함

## Fragment

### 정의

- 여러 자식 요소를 하나의 부모로 그룹화하되, 추가적인 DOM Node를 생성하지 않게 하는 React 내장 컴포넌트
- 불필요한 `<div>`나 래퍼 요소 없이 여러 요소를 그룹화할수 있음

### 주요 기능

- 여러 자식 요소를 하나의 부모 요소로 감싸지 않고 그룹화
- DOM 구조를 깔끔하게 유지하면서 React의 다중 요소 반환 요구사항 충족

### 사용 방식

```jsx
<>
  <ChildA />
  <ChildB />
  <ChildC />
</>;

// 또는
import { Fragment } from 'react';

<Fragment>
  <ChildA />
  <ChildB />
  <ChildC />
</Fragment>;
```

### 주요 특징

- 불필요한 div 등의 wrapper 요소 제거로 DOM 구조 간소화
- key prop이 필요한 경우 `<React.Fragment>` 문법 사용 필요 (혹은 import하여 `<Fragment>`로)
- CSS 스타일링이나 DOM 조작이 필요 없는 경우에 이상적

## Profiler

### 정의

- React 애플리케이션의 렌더링 성능을 측정하는 컴포넌트

### 주요 기능

- 컴포넌트 트리의 렌더링 빈도와 비용을 프로그래밍 방식으로 측정
- 성능 병목 현상 식별 및 최적화 지점 파악

### 사용 방식

```jsx
<Profiler id="Navigation" onRender={onRenderCallback}>
  <Navigation {...props} />
</Profiler>
```

### 주요 특징

- 프로덕션 빌드에서는 자동으로 비활성화되어 오버헤드 방지
- 특정 부분의 성능만 측정 가능
- `onRender` 콜백을 통해 상세한 타이밍 정보 제공

## StrictMode

### 정의

- 애플리케이션의 잠재적 문제를 식별하는 도구

### 주요 기능

- 안전하지 않은 생명주기 메서드 사용 감지
- 레거시 문자열 ref 사용 경고
- 예기치 않은 부작용 검출
- 중복 렌더링을 통한 버그 발견

### 사용 방식

```jsx
<React.StrictMode>
  <App />
</React.StrictMode>
```

### 주요 특징

- 개발 모드에서만 활성화되며 프로덕션에는 영향 없음
- 자손 컴포넌트 트리에 추가 검사와 경고 적용
- 성능에 영향을 주지 않음

## Suspense

### 정의

- 자식 컴포넌트가 로딩을 완료할 때까지 대체 UI를 표시하는 컴포넌트

### 주요 기능

- 데이터 페칭, 코드 스플리팅 등의 비동기 작업 처리
- 로딩 상태를 선언적으로 관리

### 사용 방식

```jsx
<Suspense fallback={<Spinner />}>
  <SomeComponent />
</Suspense>
```

### 주요 특징

- 비동기 작업이 완료될 때까지 fallback UI 표시
- 여러 개의 Suspense 컴포넌트를 중첩하여 로딩 경계 설정 가능
- React.lazy()와 함께 사용하여 코드 스플리팅 구현 가능
