[상호작용성 더하기 - 노션 링크](https://cooing-dust-8b6.notion.site/Adding-Interactivity-3009ea492deb40598bcf7cc45c09a21a?pvs=4)

- 이벤트에 응답하기

  이벤트 핸들러로 전달한 함수들은 호출이 아닌 **전달**되어야 한다.

  | 함수를 전달하기 (올바른 예시)    | 함수를 호출하기 (잘못된 예시)      |
  | -------------------------------- | ---------------------------------- |
  | < button onClick={handleClick} > | < button onClick={handleClick()} > |

  handleClick() 의 경우 렌더링 과정에서 즉시 함수가 실행된다.

  JSX 내의 자바스크립트가 즉시 실행되기 때문

  인라인으로 정의하는 이벤트 핸들러를 사용하려면 **익명함수**의 형태로 전달해줘야한다.

  `<button onClick={() => alert('You clicked me!')}>`

  - 이벤트는 위쪽으로 전파된다.
    - `e.stopPropagation()` : 이벤트 핸들러가 상위태에서 실행되지 않도록 해줌
    - `e.preventDefault()` : 기본 브라우저 동작을 가진 일부 이벤트 (ex ) form submit 시 리렌더링) 가 해당 기본 동작을 실행하지 않도록 방지

- State: 컴포넌트의 기억 저장소
  `useState`
  렌더링 사이에 데이터 유지 & 새로운 데이터로 렌더링하도록 유발
  ⇒ state 변수 & 재렌더링 유발하는 state setter 함수
  훅은 컴포넌트의 최상위 수준 또는 커스텀 훅에서만 호출할 수 있다.
  동일 컴포넌트를 두 번 렌더링하면 각 복사본은 완전히 격리된 state를 가진다.
- 렌더링 그리고 커밋

  - 렌더링은 React의 컴포넌트를 호출하는 것

  - 리액트 렌더링 동작
    1 ) 렌더링 트리거
    초기렌더링 : `createRoot` ⇒ `render()`
    리렌더링 : `setSomething` 을 통해 렌더링 트리거
    2 ) 컴포넌트 렌더링
    초기렌더링 : 루트 컴포넌트 호출 / DOM 노드 생성
    리렌더링 : state 업데이트가 일어나 렌더링을 트리거한 컴포넌트를 호출 /이전 렌더링 이후 변경된 속성 계산
    3 ) 리액트가 DOM에 변경사항을 커밋
    초기렌더링 : `appendChild()` DOM API를 사용해 생성한 모든 DOM 노드를 화면에 표시
    리렌더링 : 최소한의 작업을 적용하여 DOM이 최신 렌더링 출력과 일치하도록 변경 (렌더링 간에 차이가 있는 경우만 DOM 노드 변경!)
    4 ) 브라우저 페인트(브라우저 렌더링)
    리액트가 DOM을 업데이트 한 후 브라우저가 화면을 다시 표현하는 과정

- 스냅샷으로서의 State
  렌더링은 그 시점의 스냅샷을 찍는것
  prop, 이벤트 핸들러, 로컬 변수는 모두 **렌더링 당시의 state를 사용해** 계산
  state 변수의 값은 **렌더링 내에서 절대 변경되지 않는다.**
  렌더링 하기 전에 최신 state를 읽고 싶으면?
- State 업데이트 큐
  state 변수를 설정하면 다음 렌더링이 큐에 들어감.
  React는 state 업데이트하기 전에 이벤트 핸들러의 모든 코드가 실행될 때까지 대기 (batching)
  다음 렌더링 전에 동일 state 변수를 여러번 업데이트 하려면 **업데이터 함수**를 사용하기
  - `setSomething( n => n+1 )`
    ```tsx
    <button
      onClick={() => {
        setNumber((n) => n + 1);
        setNumber((n) => n + 1);
        setNumber((n) => n + 1);
      }}
    >
      +3
    </button>
    ```
    버튼 클릭시 n ⇒ n + 1 업데이터 함수가 큐에 3번 추가됨.
    | queued update | n | returns |
    | ------------- | --- | --------- |
    | n => n + 1 | 0 | 0 + 1 = 1 |
    | n => n + 1 | 1 | 1 + 1 = 2 |
    | n => n + 1 | 2 | 2 + 1 = 3 |
    다음 렌더링에 useState를 호출하면 큐를 순회하고 결과값을 반환함
- 객체 state 업데이트하기
  state가 객체를 저장하고 있을 경우 변경이 가능하지만 렌더링 트리거가 되지 않음.
  리렌더링을 발생시키기 위해서는 새 객체를 생성해 setState에 전달해줘야함.
  - 새 객체 생성
  - spread operator 사용 `{...}`
  - 여러 필드에 단일 이벤트 핸들러 사용시
    `{...,[e.target.name] : e.target.value}` 와 같은 방법도 가능 (이벤트 타겟의 name을 설정해줘야 한다)
- 배열 state 업데이트하기

  배열을 state로 관리할 때

  `Array.filter()` , `Array.map()` 등의 새 배열을 반환하는 메서드들을 활용 하거나

  `[...]` 같은 spread operator 활용해 업데이트 하기

- state는 hooks array로 관리.. ⇒
  hooks rule 2가지

1. 컴포넌트 최상단에서 정의
2. 리액트 함수 컴포넌트에서만 사용

훅의 호출 순서가 동적으로 바뀔경우 상태를 보장해주지 않는다.
[https://velog.io/@alstnsrl98/React-hook은-왜-컴포넌트-최상단에서만-동작할까](https://velog.io/@alstnsrl98/React-hook%EC%9D%80-%EC%99%9C-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EC%B5%9C%EC%83%81%EB%8B%A8%EC%97%90%EC%84%9C%EB%A7%8C-%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C)
https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e
