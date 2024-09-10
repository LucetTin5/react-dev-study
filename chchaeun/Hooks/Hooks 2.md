# Hooks 2
## useId
- 접근성 어트리뷰트에 전달할 수 있는 고유 ID를 생성하기 위한 React Hook
- 컴포넌트의 최상위에서 호출하여 고유 ID를 생성
### 반환값
- useId를 호출한 특정 컴포넌트와 특정 useId에 관련된 고유 ID 문자열 반환
### 접근성 어트리뷰트
- aria-describedby: 두 개의 태그가 서로 연관돼있다는 걸 명시
- 컴포넌트가 여러 번 렌더링돼도 아이디는 고유해야 함
### 이점
- 서버렌더링과 함께 작동하도록 보장
## useImperativeHandle
- 노출할 ref 핸들을 사용자가 직접 정의
## useInsertionEffect
- CSS in JS 라이브러리 작성자를 위함
- useInsertionEffect를 호출하여 layout을 읽어야 하는 Effect가 호출되기 전에 스타일을 주입할 수 있음
### CSS in JS 동적 스타일 주입
1. 컴파일러를 사용하여 CSS 파일로 정적 추출
2. 인라인 스타일, 예 `<div style={{opacity: 1}}>`
3. 런타임에 `<style>` 태그 주입
   - 권장X
   - 브라우저에서 스타일을 훨씬 더 자주 다시 계산하게 하고
   - 생명주기 중 잘못된 시점에 발생하면 속도가 매우 느려질 수 있음
## useLayoutEffect
- 브라우저가 화면을 다시 그리기 전에 실행되는 useEffect
### 사용법
- 브라우저가 화면을 다시 그리기 전에 레이아웃 계산하기
## useMemo
- 재렌더링 사이에 계산결과를 캐싱할 수 있게 해주는 React Hook

### useMemo를 정말 사용해야 하는가
[dev-blog/JavaScript/should-you-really-use-usememo.md at master · yeonjuan/dev-blog](https://github.com/yeonjuan/dev-blog/blob/master/JavaScript/should-you-really-use-usememo.md)

**다음과 같은 경우에서는 useMemo의 사용을 피하세요.**
* **최적화하려는 계산의 비용이 크지 않은 경우.** 이러한 경우, useMemo를 사용할 때 발생하는 오버헤드가 이점보다 클 수 있습니다.
* **메모이제이션이 필요한지 확실하지 않은 경우.** useMemo 없이 시작한 다음, 문제가 발생하면 코드에 점진적으로 최적화를 적용하는 것이 좋습니다.
* **메모하고 있는 값이 컴포넌트로 전달되지 않는 경우.** 이 값이 JSX에서만 사용되고 컴포넌트 트리에 더 깊이 전달되지 않으면 대부분의 경우 최적화를 피할 수 있습니다. 다른 컴포넌트의 렌더링에 영향을 주지 않으므로 참조를 기억할 필요가 없습니다.
* **의존성 배열이 너무 자주 변경되는 경우.** 이 경우 useMemo는 항상 재계산되므로 성능적인 이점을 제공하지 않습니다.

https://dev.to/usulpro/how-react-forget-will-make-react-usememo-and-usecallback-hooks-absolutely-redundant-4l68
