[Hooks-2](https://cooing-dust-8b6.notion.site/Hooks-2-c52546b3f77d44a6bae45a7f8d13eba2?pvs=4)

### useId

- useId() : 접근성 어트리뷰트에 전달할 수 있는 고유 ID를 생성하는 hook

  ```tsx
  import { useId } from 'react';

  function PasswordField() {
    const passwordHintId = useId();
    // ...
  ```

  ### 매개변수

  없음

  ### 반환값

  useId를 호출한 특정 컴포넌트와 특정 useId에 관련된 고유 ID 문자열을 반환

  useId를 리스트의 key를 생성하기 위해 사용하면 안됨. key는 데이터로부터 생성해야 한다.

- useId를 통해 생성된 ID를 다른 어트리뷰트로 전달할 수 있다. `aria-decribedby` 와 같은 HTML 접근성 어트리뷰트 사용시 두개의 태그가 서로 연관되어 있다는 것을 명시할 수 있음
  ```tsx
  // 일반적인 경우
  <label>
    Password:
    <input
      type="password"
      aria-describedby="password-hint"
    />
  </label>
  <p id="password-hint">
    The password should contain at least 18 characters
  </p>

  import { useId } from 'react';

  function PasswordField() {
    const passwordHintId = useId();
    // useId 사용
    return (
      <>
        <label>
          Password:
          <input
            type="password"
            aria-describedby={passwordHintId}
          />
        </label>
        <p id={passwordHintId}>
          The password should contain at least 18 characters
        </p>
      </>
    );
  }
  ```
  화면에 PasswordField 컴포넌트가 두개 이상 있어도 ID가 충돌하지 않는다..!
  `React가 서버 렌더링과 함께 작동하도록 보장한다`
  useId는 호출한 컴포넌트의 부모 경로에서 생성된다.
- 여러 연관된 엘리먼트에 `useId` 를 사용해 공유 접두사로써 활용가능하다.

- `identifierPrefix` 를 `createRoot` 또는 `hydrateRoot` 호출에 대한 옵션으로 전달하면 생성된 모든 id에 공유 접두사 지정이 가능하다.
  ```tsx
  const root1 = createRoot(document.getElementById("root1"), {
    identifierPrefix: "my-first-app-",
  });
  root1.render(<App />);

  const root2 = createRoot(document.getElementById("root2"), {
    identifierPrefix: "my-second-app-",
  });
  root2.render(<App />);
  ```
  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/398391fa-6c95-4271-8669-27c8262c6ad0/034fde20-2ddb-4b8a-88d8-bd6f7733c307/image.png)

### useImperativeHandle

- useImpertaiveHandle(ref, createHandle, dependencies?): ref로 노출되는 핸들을 사용자가 직접 정의할 수 있게 해주는 hook
  ```tsx
  import { forwardRef, useImperativeHandle } from 'react';

  const MyInput = forwardRef(function MyInput(props, ref) {
    useImperativeHandle(ref, () => {
      return {
        // ... 메서드를 여기에 입력하세요 ...
      };
    }, []);
    // ...
  ```
  ### 매개변수
  - `ref` : `forwardRef` 렌더링 함수에서 두 번째 인자로 받은 ref
  - `createHandle` : 인자가 없고 노출하려는 ref 핸들을 반환하는 함수. 일반적으로는 노출하려는 메서드가 있는 객체를 반환
  - 선택적 `dependencies` : `createHandle` 코드 내에서 참조하는 모든 반응형 값을 나열한 목록. 생략시 createHandle 함수가 다시 실행되고 새로 생성된 핸들이 ref에 할당됨
  ### 반환값
  `undefined` 반환
  - 부모 컴포넌트에 커스텀 ref핸들 노출

    기본적으로 컴포넌트는 자식 컴포넌트의 DOM 노드를 부모 컴포넌트에 노출하지 않음

    ```tsx
    import { forwardRef } from "react";

    const MyInput = forwardRef(function MyInput(props, ref) {
      return <input {...props} ref={ref} />;
    });

    // MyInput의 부모 컴포넌트가 <input> DOM 노드에 접근하려면
    // 다음과 같이 forwardRef를 사용하여 선택적으로 참조에 포함해야 합니다.
    ```

    위 코드에서 MyInput에 대한 ref는 <input> DOM 노드를 받게 됨.

    이거 대신 사용자 지정 값을 노출할 수도 있다.

    ```tsx
    import { forwardRef, useImperativeHandle } from "react";

    const MyInput = forwardRef(function MyInput(props, ref) {
      useImperativeHandle(
        ref,
        () => {
          return {
            // ... 메서드를 여기에 입력하세요 ...
          };
        },
        []
      );

      return <input {...props} />;
    });
    ```

    위 코드에서는 input에 대한 ref가 전달되지 않음.

    전체 input DOM 노드가 아닌, focus와 scrollIntoView 두 메서드만을 노출하고 싶을 때, 실제 브라우저 DOM을 별도의 ref에 유지해야됨.

    ```tsx
    import { forwardRef, useRef, useImperativeHandle } from "react";

    const MyInput = forwardRef(function MyInput(props, ref) {
      const inputRef = useRef(null);

      useImperativeHandle(
        ref,
        () => {
          return {
            focus() {
              inputRef.current.focus();
            },
            scrollIntoView() {
              inputRef.current.scrollIntoView();
            },
          };
        },
        []
      );

      return <input {...props} ref={inputRef} />;
    });
    ```

    이 상태에서 MyInput에 대한 ref를 부모컴포넌트가 가져오면 focus와 scrollIntoView 메서드를 호출할 수 있다.

    `ref.current.focus()` , `ref.current.scrollIntoView()`

  - 사용자 정의 imperative 메서드 노출
    여러 ref에 대한 커스텀 handle을 노출시킬수 있음.

### useInsertEffect :

for `CSS-in-JS` 라이브러리 작성자,
그 외에는 useEffect || useLayoutEffect 사용할것.

- useInsertEffect(setup, dependecies?) : layout Effects 실행되기 전에 전체 요소를 DOM에 주입하는 hook
  ```tsx
  import { useInsertionEffect } from "react";

  // CSS-in-JS 라이브러리 안에서
  function useCSS(rule) {
    useInsertionEffect(() => {
      // ... <style> 태그를 여기에서 주입하세요 ...
    });
    return rule;
  }
  ```
  ### 매개변수
  - `setup` : Effects의 로직이 포함된 함수. 클린업 함수를 반환할 수도 있음.
  - 선택사항 `dependencies` : setup 코드 내의 모든 반응형 값의 목록
  ### 반환값
  `undefined`
  - CSS-in-JS 라이브러리에서 동적 스타일 주입 시 사용

### useLayoutEffect :

성능저하 이슈 있음. 가능한 경우라면 useEffect 사용할것.
두번의 렌더링이 진행되지만 최종 결과만 보여주는 점이 useEffect와 다름.

- useLayoutEffect(setup, dependencies?) : 브라우저가 화면을 다시 그리기 전에 실행되어 레이아웃을 계산하는 `useEffect`

  ```tsx
  import { useState, useRef, useLayoutEffect } from 'react';

  function Tooltip() {
    const ref = useRef(null);
    const [tooltipHeight, setTooltipHeight] = useState(0);

    useLayoutEffect(() => {
      const { height } = ref.current.getBoundingClientRect();
      setTooltipHeight(height);
    }, []);
    // ...
  ```

  ### 매개변수 : useInsertEffect, useEffect 동일

  ### 반환값 : undefined

  - `useLayoutEffect` 내부의 코드와 이로인한 state업데이트는 브라우저가 화면을 다시 그리는 것을 막기 때문에 성능 저하 발생할 수도 있다. 왠만하면 `useEffect` 사용

- 브라우저가 화면을 다시 그리기 전에 레이아웃 계산하기
  마우스 커서를 올리면 툴팁이 요소 옆에 나타나는 경우…!
  충분한 공간이 있을시 요소 위에 나타나지만,
  공간 부족시 요소 아래에 나타나야한다면 ???
  ⇒ 올바른 위치에 렌더링하기 위해선 툴팁의 높이를 알아야한다.
  따라서 두번의 렌더링이 필요.
  1. 툴팁을 (잘못된 위치라도) 아무 위치에 렌더링합니다
  2. 툴팁의 높이를 계산해서 툴팁을 배치할 위치를 결정합니다.
  3. 올바른 위치에 툴팁을 *다시* 렌더링합니다.
  이 작업이 브라우저가 화면을 다시 그리기 전에 모두 이뤄져야 함!
  ```tsx
  function Tooltip() {
    const ref = useRef(null);
    const [tooltipHeight, setTooltipHeight] = useState(0); // 아직 실제 높이를 모릅니다.

    useLayoutEffect(() => {
      const { height } = ref.current.getBoundingClientRect();
      setTooltipHeight(height); // 실제 높이를 알았으니 다시 렌더링합니다.
    }, []);

    // ...아래에 올 렌더링 로직에서 tooltipHeight를 사용하세요...
  }
  ```
  1. `Tooltip` 은 초기화된 값인 `tooltipHeight = 0`으로 렌더링 됩니다 (따라서 툴팁의 위치는 잘못될 수 있습니다).
  2. React가 이 툴팁을 DOM에 배치하고 `useLayoutEffect` 안의 코드를 실행합니다.
  3. `useLayoutEffect`가 툴팁의 [높이를 계산하고](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect) 바로 다시 렌더링시킵니다.
  4. `Tooltip` 이 실제 `tooltipHeight`로 렌더링 됩니다. (따라서 툴팁이 올바른 위치에 배치됩니다.)
  5. React가 DOM에서 이를 업데이트하고 마침내 브라우저가 툴팁을 표시합니다.
  두번의 렌더링을 거치지만 실제로 보이는 것은 최종 결과뿐…! ⇒ 이 경우에는 `useEffect`가 아닌 `useLayoutEffect`가 필요하다

### useMemo : memoization

- useMemo(calculateValue, dependencies) : 재렌더링 사이에 계산 결과를 캐싱할 수 있게 해주는 hook.
  ```tsx
  import { useMemo } from "react";

  function TodoList({ todos, tab }) {
    const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
    // ...
  }
  ```
  ### 매개변수
  - `calculateValue` : 캐싱하려는 값을 계산하는 함수.
    - 인자를 받지 않는 순수 함수. 모든 타입의 값을 반환할 수 있어야 함.
    - React 초기 렌더링 중에 함수 호출
    - 다음 렌더링에서 `dependencies` 가 변경되지 않았을 시 동일한 값을 반환.
    - 변경되었다면 `calculateValue` 호출하고 결과값 반환
  - `dependencies` : `calculateValue` 코드 내에 참조된 모든 반응형 값들의 목록
  ### 반환값
  초기 렌더링 : 인자없이 `calculateValue` 호출한 결과 반환
  이후 렌더링 : 저장된 값을 반환하거나, `dependencies` 가 변경되었다면 `calculateValue`를 다시 호출한 후의 결과값
  useMemo는 성능 최적화용도로만 사용할 것 !
  - 비용이 높은 로직의 재계산 생략하기
    비싼 연산인지 알아보는 방법…
    ```tsx
    console.time("filter array");
    const visibleTodos = filterTodos(todos, tab);
    console.timeEnd("filter array");

    // 와 같이 console.time & console.timeEnd 추가해서 콘솔 찍어볼수있다.!
    ```
  - 컴포넌트 재렌더링 건너뛰기 : `memo` 로 컴포넌트를 감싸면 된다.
    `const MemoizedComponent = memo(SomeComponent, arePropsEqual?)`
    ```tsx
    import { memo } from "react";

    const List = memo(function List({ items }) {
      // ...
    });
    ```
  - 다른 Hook의 종속성 메모화
  - 함수 메모화 ⇒ but `useCallback` 사용할 것

### Hydration

프론트엔드 SSR과 관련.

서버사이드 렌더링은 웹페이지를 서버에서 미리 렌더링하여 클라이언트에게 HTML을 전달

클라이언트 측에서 자바스크립트를 통해 추가적인 기능을 활성화하는 과정이 hydration

### CSS-in-CSS, CSS-in-JS

출처 : https://choi-hyunho.com/css-in-js?source=more_series_bottom_blogs

### 기존 css 문제점

- Global namespace: 모든 스타일이 global에 선언되어 중복되지 않는 class 이름을 적용해야 하는 문제
- Dependencies: css 간의 의존관계를 관리하기 힘든 문제
- Dead Code Elimination: 기능 추가, 변경, 삭제 과정에서 불필요한 CSS를 제거하기 어려운 문제
- Minification: 클래스 이름의 최소화 문제
- Sharing Constants: JS 코드와 상태 값을 공유할 수 없는 문제
- Non-deterministic Resolution: CSS 로드 순서에 따라 스타일 우선 순위가 달라지는 문제
- Isolation: CSS와 JS가 분리된 탓에 상속에 따른 격리가 어려운 문제

### CSS-in-CSS : ex) `CSS Module`

CSS 스타일을 별도의 파일이나 HTML 마크업에 작성하는 기술

전체 페이지에 필요한 CSS를 처음부터 전부 로딩하여 style태그 생성

### CSS-in-JS : ex) `styled-component`

CSS를 자바스크립트 코드에서 작성하는 방식

해당 컴포넌트가 렌더링 될때만 스타일 태그를 생성 ⇒ 현재 사용 중인 스타일만 DOM에 포함

위에 언급한 기존 CSS 문제 해결 가능

- but 단점 있음
  - css-in-js를 사용하기 위한 별도의 라이브러리 설치 ⇒ 번들 크기 증가 , CSR 방식에서 최초 로딩시간을 지연시킴
  - SSR 사용시 성능적인 문제 ( 컴포넌트가 렌더링될때만 스타일 태그를 생성하기 때문에, 실제 페이지가 이동할때마다 내려받아야함 )
- 동작방식
  1. 브라우저 런타임에서 CSS와 컴포넌트를 이어주는 classname 동적 생성
     (CSS-in-JS에서는 각 컴포넌트를 해싱해서 classname 동적 생성 ex) `sc-cMWNzn`)
  2. css parser로 style html tag를 만들어줌
