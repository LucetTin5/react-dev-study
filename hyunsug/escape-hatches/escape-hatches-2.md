# 탈출구 - 2

## Effect in React

React에서 말하는 **Effect**란 무엇일까?

공식문서에 따르면, Effect와 구분지어야 할 것을 React 컴포넌트 `inside`에서 일어나는 두가지의 로직을
`Rendering code`, `Event handlers`로 규정짓고 있는 듯 하다.

`Effect`는 렌더링에 의해 발생하는 `side effect`를 특정지어지며, 리액트에서의 `Effect`란 렌더링 이후
사이드 이펙트를 수행하기 위한 메커니즘으로 볼 수 있다.

모든 Effect는 `렌더링 이후`에 실행되며, `useEffect`훅을 통해 Effect를 구현하게 된다.

```jsx
useEffect(() => {
  // Effect
  return () => {
    // Clean Up function
  };
}, [...dependencies]);
```

## useEffect의 주요 특징

1. **렌더링 후 실행**: useEffect 내의 코드는 컴포넌트가 화면에 렌더링된 후에 실행된다.
2. **의존성 배열**: 두 번째 인자로 전달되는 배열은 effect가 의존하는 값들을 포함한다. 이 값들이 변경될 때만 effect가 재실행된다.
3. **정리(Clean-up) 함수**: effect가 수행한 작업을 정리해야 할 때 사용한다. 컴포넌트가 언마운트되거나 다음 effect가 실행되기 전에 호출된다.

## Effect 사용 시 고려사항

1. **필요성 검토**: 모든 상호작용이나 상태 업데이트에 Effect가 필요한 것은 아니다. 가능한 한 이벤트 핸들러를 사용하는 것이 좋다.
2. **의존성 배열 관리**: 불필요한 재실행을 막기 위해 의존성 배열을 신중히 관리해야 한다.
3. **정리(Clean-up) 함수의 중요성**: 구독, 타이머 등을 설정한 경우 반드시 정리 함수를 통해 해제해야 한다.

## Effect의 동작 원리 간단하게

1. **렌더링 단계**:
   React는 컴포넌트를 렌더링하면서 useEffect 호출을 만난다. 이때 Effect 함수와 의존성 배열을 저장해둔다.

   ```javascript
   function ReactInternals() {
     let effects = [];

     function useEffect(effectFn, deps) {
       effects.push({ effectFn, deps });
     }

     // ...렌더링 로직
   }
   ```

2. **커밋 단계**:
   React는 실제 DOM을 업데이트한다. 이 시점에서는 아직 Effect가 실행되지 않는다.

3. **레이아웃 단계**:
   브라우저가 레이아웃을 계산하고 화면을 그린다.

4. **Effect 실행 단계**:
   React는 저장해둔 Effect들을 순회하면서 실행한다. 이전 렌더링의 의존성 배열과 현재의 의존성 배열을 비교하여 변경이 있는 경우에만 Effect를 실행한다.

   ```javascript
   function runEffects() {
     effects.forEach((effect) => {
       if (shouldRunEffect(effect.deps)) {
         // 이전 Effect의 cleanup 함수 실행
         if (effect.cleanup) effect.cleanup();

         // 새로운 Effect 실행 및 cleanup 함수 저장
         effect.cleanup = effect.effectFn();
       }
     });
   }

   function shouldRunEffect(deps) {
     // 의존성 배열 비교 로직
     // ...
   }
   ```

5. **cleanup 함수 실행**:
   컴포넌트가 언마운트되거나 다음 렌더링에서 Effect가 다시 실행되기 전에 이전 Effect의 cleanup 함수가 실행된다.

이러한 단순화된 모델을 통해 우리는 Effect가 어떻게 동작하는지, 그리고 왜 렌더링 이후에 실행되는지를 이해할 수 있다. 실제 React의 구현은 이보다 훨씬 복잡하고 최적화되어 있지만, 이 모델은 useEffect의 기본적인 동작 원리를 설명하는 데 도움이 된다.

Effect의 이런 동작 방식은 개발자가 Side Effect를 선언적으로 관리할 수 있게 해주며, React가 적절한 시점에 이를 처리할 수 있게 한다. 이는 컴포넌트의 생명주기와 밀접하게 연관되어 있으면서도, 클래스 컴포넌트의 생명주기 메서드보다 더 유연하고 강력한 방식으로 Side Effect를 다룰 수 있게 해준다.

## Effect 최적화 전략

1. **불필요한 의존성 제거**: Effect 내부에서 사용되는 값들을 최소화한다.
2. **객체와 함수의 안정성**: useCallback, useMemo 등을 활용하여 불필요한 재생성을 방지한다.
3. **Effect 분리**: 독립적인 로직은 별도의 Effect로 분리하여 관리한다.

## 일반적인 실수와 해결 방법

1. **무한 루프**: 의존성 배열에 포함된 상태를 Effect 내에서 업데이트할 때 발생할 수 있다. 조건문이나 이전 값 비교를 통해 해결할 수 있다.
2. **과도한 Effect 사용**: 단순한 상태 업데이트나 계산은 렌더링 과정에서 직접 처리하는 것이 효율적이다.
3. **데이터 페칭의 경쟁 상태(Race condition)**: 비동기 작업의 결과가 예상과 다른 순서로 도착할 때 발생한다. 취소 토큰이나 플래그 변수를 사용하여 관리할 수 있다.

## useLayoutEffect와의 차이 - useLayoutEffect는 훅 API 파트를 다룰 때 다시 디테일하게 다뤄보기로

`useLayoutEffect`는 `useEffect`와 유사하지만, 브라우저가 화면을 다시 그리기 전에 동기적으로 실행된다. 이는 DOM을 조작하는 작업에 유용할 수 있지만, 렌더링을 지연시킬 수 있으므로 주의가 필요하다.

### 의도치 않은 useLayoutEffect 사용

때로는 의도치 않게 `useLayoutEffect`를 사용하게 되는 경우가 있다:

1. **서드파티 라이브러리 사용**: 일부 라이브러리들이 내부적으로 `useLayoutEffect`를 사용할 수 있다.
2. **커스텀 훅 체인**: 여러 커스텀 훅이 연결되어 있을 때, 그 중 하나라도 `useLayoutEffect`를 사용하면 전체가 `useLayoutEffect`로 동작할 수 있다.
3. **조건부 훅 사용**: 조건에 따라 `useEffect`나 `useLayoutEffect`를 선택적으로 사용하는 경우, React는 항상 `useLayoutEffect`로 처리한다.

이러한 의도치 않은 `useLayoutEffect` 사용은 성능 저하나 서버 사이드 렌더링 문제를 일으킬 수 있으므로, 주의 깊게 코드를 검토하고 필요한 경우에만 `useLayoutEffect`를 사용해야 한다.

Effect는 React 애플리케이션에서 부수 효과를 관리하는 강력한 도구이지만, 적절한 사용과 최적화가 중요하다. 컴포넌트의 생명주기와 상태 관리를 깊이 이해하고 Effect를 활용할 때, 더 효율적이고 예측 가능한 React 애플리케이션을 구축할 수 있다.

#### 참고

[[번역] useEffect는 종종 페인트(paint) 이전에 동작합니다.](https://velog.io/@lky5697/unintentional-layout-effect)
