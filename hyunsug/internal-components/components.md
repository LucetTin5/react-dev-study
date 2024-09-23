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

### 중첩 Fragment 사용 관련하여

#### React DevTools

- `Fragment`는 실제 DOM Node를 생성하지 않기에, DevTools의 컴포넌트 트리에 나타나지 않는다.
- 중첩 `Fragment`가 많으면 해당 툴을 통한 컴포넌트 구조 이해에 방해가 될 수 있음

#### 에러 핸들링

- 스택 트레이스 상에 `Fragment`는 이름/표시를 가지지 않아 디버깅에 어려움이 있을 수 있음

## Profiler

### 정의

- React 애플리케이션의 렌더링 성능을 측정하는 컴포넌트
- 렌더링시마다 어떤 컴포넌트가 얼마나 자주 렌더링되는지, 렌더링에 시간이 얼마나 걸리는지 등에 대한 정보를 제공

### 주요 기능

- 특정 컴포넌트 트리의 렌더링 빈도와 비용을 프로그래밍 방식으로 측정
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

### `onRender` 콜백

```jsx
import { Profiler } from 'react';

function onRenderCallback(
  id, // Profiler 트리의 id값으로 문자열
  phase, // 'mount' or 'update'
  actualDuration, // 커밋된 업데이트를 렌더링하는데 걸린 시간
  baseDuration, // 메모 없이 전체 트리를 렌더링하는데 걸리는 예상 시간
  startTime, // React가 렌더링을 시작한 시간
  commitTime, // React가 해당 커밋을 완료한 시간
  interactions // 이 업데이트에 해당하는 상호작용들의 집합
) {
  // 콜백 내부 구현
}
```

### React DevTools - Profiler

- DevTools의 Profiler 탭은 렌더링 성능을 시각적으로 분석할 수 있게 함
- 코드 변경 없이 성능 측정 작업 가능
- 컴포넌트 트리 분석(컴포넌트별 렌더링 시간 측정)
- 소요시간을 시각적으로 나타냄
- 이벤트 관련 렌더링 추적 가능
- Development 모드에서만 동작

## StrictMode

### 정의

- 애플리케이션의 잠재적 문제를 식별하는 도구

### 주요 기능

- 안전하지 않은 생명주기 메서드 사용 감지
- 레거시인 `문자열 ref` 사용 경고
- 예기치 않은 부작용 검출: 개발 모드에서 일부 함수를 두번 호출하여 비순수함수, 사이드이펙트를 감지
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

### 두 번 렌더링

- 컴포넌트의 순수성, 안정성을 보장하기 위해 함수형 컴포넌트와 일부 훅을 두 번 호출
- 두 번의 호출에서 동일한 결과가 나와야 함
- 불일치 시 콘솔에 경고를 표시하도록 함

#### 두 번 호출되는 함수?

- 함수형 컴포넌트 자체
- `useEffect`와 같은 `Effect` 함수
  - `useEffect`, `useLayoutEffect`의 이펙트 함수와 클린업함수가 두 번 실행
  - 마운트 해제, 재마운트를 시뮬레이션하여 클린업함수의 정확성을 검증

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
- 여러 개의 Suspense 컴포넌트를 중첩하여 경계 설정 가능
- React.lazy()와 함께 사용하여 코드 스플리팅 구현 가능

### Thrown Promise

- 이는 React 컴포넌트가 렌더링 준비가 되지 않았음을 React에 알리는 메커니즘
- 컴포넌트가 렌더링을 시도할 때, 필요 데이터가 없다면 Promise를 throw함
- React는 이 `thrown promise`를 캐치하고 해당 컴포넌트의 렌더링을 일시 중단
- resolve되면 다시 렌더링

### Fiber/Suspense

Fiber 아키텍쳐에서 Suspense와 Thrown Promise의 상호작용

- Fiber 트리 탐색: React는 트리를 순회하며 각 fiber node를 처리
- Promise의 감지: Thrown Promise가 React에 의해 감지
- 일시 중단: React는 해당 fiber node의 작업을 일시 중단하고, 가까운 Suspense 경계를 찾음
- Fallback Render: Suspense의 fallback UI를 렌더링
- 재개: Promise가 resolve되면 중단된 지점에서 렌더링을 재개
