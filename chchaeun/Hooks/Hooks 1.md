useActionState
> 실험
- 폼 액션의 결과를 기반으로 state를 업데이트할 수 있도록 제공하는 Hook 
```js
const [state, formAction] = useActionState(fn, initialState, permalink?);
```

### 매개변수
- fn: 폼이 제출되거나 버튼을 눌렀을 때 호출될 함수, 첫 번째 인수로 폼의 이전 state를 전달, state는 초기에 전달한 initialState이고, 이후에는 이전 실행의 반환값
- initialState: 초기 state로 설정하고자 하는 값, 직렬화할 수 있는 값
- permalink: 폼이 수정하는 고유의 url이 포함된 문자열
### 반환값
1. 현재 state
2. form 컴포넌트의 action prop에 전달하거나 폼 내부 button 컴포넌트의 formAction prop에 전달할 수 있는 새로운 액션

-> 함수형 프로그래밍 도움

## useCallback
```js
const cachedFn = useCallback(fn, dependencies);
```

### 매개변수
- 리렌더링 간에 함수 정의 캐싱
- fn: 캐싱할 함수값, 어떤 인자나 반환값도 가질 수 잇음
- dependencies: fn 내에서 참조되는 모든 반응형 값의 목록
### 반환값
- 최초 렌더링에서는 전달한 fn 함수 그대로 반환
- 후속 렌더링에서는 이미 저장해두었던 fn 함수 그래도 반환하거나 현재 렌더링 중에 전달한 fn 함수 그대로 반환
### 용법
- 함수 동등성 보장
  - 함수를 다른 컴포넌트로 전달할 때
  - memo와 같이 사용

## useContext
```js
const value = useContext(SomeContext);
```
- 컴포넌트 최상위 수준에서 호출하여 context를 읽고 구독

### 매개변수
- SomeContext: createContext로 생성한 context
### 반환값
- useContext는 호출하는 컴포넌트에 대한 context값 반환
- 트리에서 호출하는 컴포넌트 위의 가장 가까운 SomeContext.Provider에 전달된 value로 결정

## useDebugValue
- React DevTools에서 커스텀 훅에 라벨을 추가할 수 있게 해주는 React hook
```js
useDebugValue(value, format?)
```

### 매개변수
- value: React DevTools에 표시하고 싶은 값
- format?: 포맷팅 함수, value를 인자로 포맷팅 함수를 호출하여 반환된 값을 표시

## useDeferredValue
```js
const deferredValue = useDeferredValue(value)
```
- UI 일부 업데이트를 지연시킬 수 있는 React Hook
### 매개변수
- value: 지연시키려는 값
- initialValue? (Canary): 처음 렌더가 실행될 때 사용될 값
### 반환값
- currentValue: 초기 렌더링 중에는 사용자가 제공한 값, 업데이트가 발생하면 먼저 이전 값으로 리렌더링 시도하여 반환값이 이전값과 일치하도록 하고, 백그라운드에서 새값으로 리렌더링 시도하여 반환값이 업데이트된 새 값과 일치하도록
### 내부 동작
1. react는 새로운 query로 리렌더링하지만 값은 deferredQuery를 사용, 결과목록에 전달하는 deferredQuery값은 지연된 query값
2. 백그라운드에서 react는 query와 deferredQuery를 모두 ab로 업데이트한 상태로 리렌더링을 시도

# 함수형 프로그래밍
- 좋은 설계를 유지한다 -> 코드 전반에 걸쳐 일관적인 원칙과 규칙으로 작성 돼야 한다.
- 이러한 원칙과 방법이 되는 관점을 **패러다임**이라고 부른다.
- 함수형 프로그래밍 패러다임
  - 데이터를 함수로 연결하는 것을 중심으로 사고하고 프로그래밍 하는 것
- 자바스크립트
  - 함수형 패러다임을 기반으로 하며 객체지향의 문법을 쓰는 독특한 언어
  - 멀티 패러다임
  - 이 둘의 패러다임을 잘 구분해서 섞어 쓸 수 있어야 좋은 코드를 작성할 수 있다
  - 함수형으로 사고하는 패러다임을 잘 이해할 필요가 있다
### 함수형 프로그래밍 용어 다시 쓰기
1. 순수함수: 코드를 액션과 계산, 데이터로 분리하자. 특히 액션에서 계산을 분리하는 코드를 작성하자.
2. 불변성: 카피온라이트와 방어적복사를 이용하여 불변성을 유지하자.
3. 선언적 패턴: 계층형 설계와 추상화 벽을 이용하여 무엇과 어떻게를 구분하여 좋은 설계를 유지하자.

## 액션, 계산, 데이터
- 프로그램 = 액션 + 계산 + 데이터

> - 액션은 호출 시점과 실행 횟수에 의존합니다.
> - 계산은 입력과 출력으로 이루어져있습니다.
> - 데이터는 이벤트에 대한 사실입니다.

```js
function App() {

  // 데이터
  const [count, setCount] = useState(0)

  // 계산
  const increase = (value) => value + 1

  // 액션
  const onClick = () => setCount(increase(count))
  
  // 선언적 패턴
  return <button onClick={onClick}>{count}</button>
}
```

- 데이터: 숫자, 화면에 보여줄 수 있음
- 액션: 버튼 클릭 시 숫자가 1 커지는 것은 언제하느냐, 몇 번 하느냐에 따라 다른 결과가 만들어짐
- 계산: 입력값을 통해 출력값을 만들어냄
-> 액션이 발생하면 미리 정의된 계산에 의해 데이터가 바뀌게 됨

```js
// 잘못 분리된 계산함수
const increase = () => {
  ...
  setState(count + 5)
}

const onClick = () => {
  ...
  increase()
}
```
- 계산은 반드시 입출력으로 이루어져야 하며, 같은 입력에 대해서는 항상 같은 출력값을 내놓아야 함
- 함수형 프로그래밍의 핵심: 액션과 계산을 확실히 분리해서 액션을 최소화하고 계산함수를 많이 만들어서 관리하는 것이 목표

```js
const increase = (count, offset) => {
  const result = count + offset // count와 offset을 명시적 입력으로 변경합니다.
  // setState와 같은 명시적 출력은 사용하지 않습니다.
  return result
}

// 암묵적인 입출력은 별도의 action에 모아줍니다.
const action = () => {
  setState(increase(count, 1)) // 외부에서 필요한 모든 입력을 넣어줍니다.
}
```
- 독립적이며 언제든 재사용이 가능하며 테스트가 용이함

## 불변성 - 카피온라이트, 방어적복사
- 배열을 만들어두고 배열의 값을 하나씩 늘려가는 형태
```js
// 1씩 증가하는 계산 함수
const increase = (value) => value + 1

// 1씩 증가하는 값을 배열에 넣는 함수 ex) [1] -> [1,2] -> [1,2,3]
const increase = (arr) => {
  const value = arr[arr.length - 1]
  arr.push(value + 1)
  return arr
}

// 이렇게 작성을 하면 어떤 문제가 있을까?
```
- 계산 X 액션 O
- pass by reference 사용했기 때문

```js
const increase = (arr) => {
  arr = arr.slice() // array를 조작하기 전에 복사해서 사용한다.
  const value = arr[arr.length - 1]
  arr.push(value + 1)
  return arr
}

// spread 표기법을 쓴다면 더 간결하게 작성할 수 있다.
const increase = (arr) => [...arr, arr[arr.length - 1]]
```
- 원본 수정 없이 출력할 수 있음

### 방어적 복사
- 해당 액션이 우리가 수정할 수 없는 라이브러리라면
```js
// 액션을 써야하지만 라이브러리 함수라서 내가 수정할 수가 없다.
import someActionLibray from "lib"

const someCalcuation = (obj, value) => {
  someActionLibray(obj, value) // obj의 값을 변경해서
  return obj // 출력한면 이 함수는 계산일까?
}
```

- 방어적 복사 혹은 깊은 복사를 통해 액션을 계산으로 만들 수 있음
```js
const someCalcuation = (obj, value) => {
  const clone = structuredClone(obj); // 완전한 clone을 만들어 낸다.
  someActionLibray(clone, value) // clone값을 변경해도 원본은 변하지 않는다.
  return clone
}
```

## 선언적 패턴과 계층형 구조
- 각 영역을 함수로 구분 짓다보면 자연스럽게 좋은 구조를 만들어 낼 수 있다.
![계층형 구조](https://velog.velcdn.com/images/teo/post/71e43de1-792b-4ae4-8805-b899d51ab735/image.png)
- 액션으로 갈 수록 코드의 형태는 무엇을 하는 것인지, 행동을 기반한 기획서에 가까운 코드
- 계산과 데이터에 가까워질 수록 데이터 중심적인 코드, 재사용성이 높고 테스트를 하기 좋은 형태
-> 계층을 뛰어 넘는 코드를 작성하지 않도록 하여 좋은 구조를 계속 유지하자.

https://velog.io/@teo/functional-programming
