[UI 표현하기 - 노션 링크](https://cooing-dust-8b6.notion.site/describing-the-ui-0ab8404f19a345ddb119b4f1d5af80cb)

### 첫 번째 컴포넌트

- 컴포넌트 내에 다른 컴포넌트를 중첩해 정의하면 안된다. 렌더링 될때마다 내부의 컴포넌트가 새로 정의되며 만들어지기 때문 (”**볼드모트**”가 느린이유 )

### 컴포넌트 import 및 export하기

| Syntax  | Export 구문                         | Import 구문                           |
| ------- | ----------------------------------- | ------------------------------------- |
| Default | export default function Button() {} | import Button from './button.js';     |
| Named   | export function Button() {}         | import { Button } from './button.js'; |

### JSX로 마크업 작성하기

- JSX는 확장된 문법, React는 JS 라이브러리로 별개의 개념
- 하나의 루트 엘리먼트를 return해야함. <div> or <>로 감싸 하나의 객체로 만들어줘야 한다.
  ( JSX 내부적으로는 일반 JavaScript 객체로 변환 )
- 어트리뷰트는 camelCase로 작성되지만 `aria-*` , `data-*` prefix의 경우는 대시를 포함해서 사용

### 중괄호가 있는 JSX 안에서 자바스크립트 사용하기

- 중괄호 `{ }` 사이에 JS를 사용할 수 있음
  ```tsx
  const name = "Gregorio Y. Zara";
  return <h1>{name}'s To Do List</h1>;
  ```
- 중괄호 사용하는 위치
  - JSX 태그 내에 문자 `<div> {name} </div>`
  - 태그의 어트리뷰트 `<img src={avatar}>`
- 이중 중괄호는 특별한 문법이 아닌 중괄호 사이에 JS 객체가 들어가 있는 형태 `{{name:'myName'}}`
- 인라인 style 프로퍼티는 camelCase로 작성됨. `style={{backgroundColor: 'blue'}}`

### 컴포넌트에 props 전달하기

- <img> 태그에 전달할 수 있는 props들은 미리 정의되어있다. ex) src, alt, width, height, className … (ReactDOM이 HTML 표준을 준수하고 있음)
- props 사용 예시

```tsx
/// ES6 구조 분해 할당 이용
/// default value 설정 가능
function myComp({ props1, props2 = "default" }) {
  return (
    <div>
      {props1}, {props2}
    </div>
  );
}

/// 동일한 의미
function myComp(props) {
  let props1 = props.props1;
  let props2 = props.props2 ? props.props2 : "default";

  return (
    <div>
      {props1}, {props2}
    </div>
  );
}
```

- 자체 컴포넌트를 props로 전달

```tsx
function Parent({ children }) {
  return <div className="parent">{children}</div>;
}

export default function GrandPa() {
  return (
    <Parent>
      <div> 이것은 children </div>
    </Parent>
  );
}
```

- props는 읽기 전용으로, 렌더링 마다 새로운 버전의 props를 받음. 변경할 수 없다.

### 조건부 렌더링

- 조건문 `if(){} else{}`
- 삼항 연산자 `condition? ifTrue : ifFalse`
- 논리 연산자 `&&`
  - && 의 왼쪽 요소를 JS가 자동으로 boolean으로 변환하지만 `0` 일 경우 아무것도 반환하지 않는 것이 아닌 `0`을 렌더링하게됨. 숫자를 넣지 말것.

### 리스트 렌더링

- js `Array.map()` 사용
- js `Array.filter()` 사용
- map 호출 시 JSX 엘리먼트 내부의 key가 필요하다.
- 만약 리스트 내부의 엘리먼트가 Fragment로 감싸진 경우에는 짧은 `<></>` 구문이 아닌, 명시적인 `<Framgent key={id}></Framgent>` 프래그먼트 문법을 사용해야한다.

### 컴포넌트 순수하게 유지하기

- 순수성 Purity : **같은 입력, 같은 출력**
- React로 작성되는 모든 컴포넌트가 순수 함수일 것이라 가정

```tsx
let guest = 0;

function Cup() {
  // 나쁜 지점: 이미 존재했던 변수를 변경하고 있다!
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

- 위의 코드에서 <Cup /> 컴포넌트들이 각각 다른 결과를 출력하므로 순수하지 않음. ⇒ props로 넘겨 순수성 up :)
- StrictMode ⇒ 순수성 체크 로직. 두 번 씩 호출했을 때 결과 값이 바뀌는 경우 규칙 위반 컴포넌트

### 트리로서 UI 이해하기

- 트리로서의 UI
  - 브라우저 : HTML(DOM), CSS(CSSOM) 모델링 위한 트리구조 사용
  - 리액트도 마찬가지로 컴포넌트를 트리로 모델링해서 사용
- 렌더 트리 ⇒ 렌더링 속도 디버깅
  렌더링된 컴포넌트로 구성된 UI 트리
  root 노드는 Root 컴포넌트. 부모 - 자식 관계를 표현
  렌더 트리의 구성요소는 React 컴포넌트로만 구성 되어 있다. (HTML 요소는 트리 요소가 아님)
  리액트 앱의 단일 렌더링을 나타냄
- 모듈 의존성 트리 ⇒ 번들 크기 디버깅
  앱의 모듈 의존성( import ) 를 모델링
  root 노드는 루트 모듈 (일반적으로 루트 컴포넌트 포함 모듈)
  컴포넌트가 아닌 모듈도 트리의 구성요소
