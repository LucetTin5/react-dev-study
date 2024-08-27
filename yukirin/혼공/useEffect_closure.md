# 각각의 렌더링은 고유한 Effect를 갖는다
이 [예제](https://codesandbox.io/s/vlrxty?file=/src/App.js&utm_medium=sandpack) 솔직히 이해하기 어려웠다 -_-
> 우선 예제를 이해하기 위해 유념해야 될 대전제는 React에서 **Effect는 렌더링 후 실행된다**는 것이다. 유념하고 스타뜨

<img width="825" alt="스크린샷 2024-08-10 오전 8 39 55" src="https://github.com/user-attachments/assets/95ff9752-8ba9-4902-b06e-089508eedca8">

## 예제 분석하기
### state 업데이트
- 사용자가 입력란에 타이핑할 때마다 `setText` 함수가 호출됨
- 이로 인해 `text`가 업데이트 되고, `Playground` 컴포넌트가 리렌더링
### Effect의 동작
- dependency가 `text`이므로, `text`가 업데이트 될 때마다 `useEffect` 실행
- `useEffect` 내부에서는 `setTimeout`을 사용하여 3초 후 `console.log('⏰ ' + text)` 하는 타이머 설정
- 타이머가 설정되면 cleanup 함수에서 `clearTimeout`을 호출하여 이전 타이머 취소

<br />

## 타이핑 시 콘솔 흐름
가정: 사용자가 input 창에 "abc"를 빠르게 입력하는 상황

|입력 "a"|입력 "ab"|입력 "abc"|3초 후|
|:--:|:--:|:--:|:--:|
| 🔵 스케줄 로그 "a" | 🟡 취소 로그 "a", 🔵 스케줄 로그 "ab" | 🟡 취소 로그 "ab", 🔵 스케줄 로그 "abc" | ⏰ "abcd" |
| 타이머 실행 예정 | 이전 타이머 취소, 새 타이머 설정 | 이전 타이머 취소, 새 타이머 설정 | 타이머 실행 |

<br />

## "ab"가 "a" 타이머를 취소하는 원리
"ab"가 "a" 입력 때 설정된 타이머를 취소할 수 있는 건 `useEffect`와 [closure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) 메커니즘 덕분이다.
### closure란
- 자바스크립트에서 함수가 자신이 선언된 scope 밖에서 호출되더라도 접근할 수 있는 기능
- 클로저를 사용하면 함수가 자신이 선언된 시점의 변수에 접근할 수 있음
```javascript
function outerFunction() {
  const outerVariable = 'I am outside!';
  function innerFunction() {
    console.log(outerVariable); // 외부 변수에 접근 가능
  }
  return innerFunction;
}

const myFunction = outerFunction();
myFunction(); // "I am outside!" 출력
```
### setTimeout과 closure
- `setTimeout`은 closure를 통해 콜백 함수가 선언되는 시점의 `text` 값을 기억
- 즉, 3초 후의 `text` 값이 아닌 타이머가 설정될 당시의 `text` 값을 사용
### cleanup과 타이머 취소
```javascript
return () => {
  console.log('🟡 취소 로그 "' + text);
  clearTimeout(timeoutId);
};
```
- 위 코드에서 `clearTimeout(timeoutId)`가 호출되면 그 시점에 설정된 타이머가 취소
- 이 cleanup 함수는 `text`의 이전 상태를 기억하고 있으므로 이전 상태 타이머 취소

<br />

## 예제와 함께 정리
### 타이핑 시 closure 흐름
`useEffect`의 dependency 변화 시 이전 Effect 정리를 위해 cleanup이 먼저 실행되고 나머지 코드가 실행된다.
|입력 "a"|입력 "ab"|입력 "abc"|3초 후|
|:--:|:--:|:--:|:--:|
| 🔵 스케줄 로그 "a" | 🟡 취소 로그 "a", 🔵 스케줄 로그 "ab" | 🟡 취소 로그 "ab", 🔵 스케줄 로그 "abc" | ⏰ "abcd" |
| 타이머 실행 예정 | 이전 타이머 취소, 새 타이머 설정 | 이전 타이머 취소, 새 타이머 설정 | 타이머 실행 |
| "a" 기억 | "a" 취소, "ab" 기억 | "ab" 취소, "abc" 기억 | "abc" 타이머 |
- "ab"를 입력하면 `text`가 "ab"로 업데이트 되므로 리렌더링 후 `useEffect`가 실행
- `useEffect`는 클린업 함수 실행하고 closure를 통해 "a"에 접근하여 "a" 타이머 취소
- "ab" 상태의 text를 기억하는 새로운 타이머가 closure를 통해 설정

<br />

## Stale closure 문제
아래와 같은 두 `useEffect` 문이 있다면 개발자는 콘솔에 1.5초마다 0, 1, 2, 3, ... 차례대로 찍힐 것을 기대한다. ([데모](https://codesandbox.io/s/trap-of-closure-oz4vtf?from-embed))
```javascript
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1);
  }, 1500);
}, []);

useEffect(() => {
  const timer = setInterval(() => {
    console.log(count);
  }, 1500);
}, []);
```
그러나 콘솔을 보면 0만 무한히 찍히는 것을 확인할 수 있다. ![image](https://github.com/user-attachments/assets/c5c2cdbe-7159-42ad-94d6-31127d521674)
- dependency가 빈배열이므로 최초 한번 실행이 되며, `count`는 최초의 것만을 기억
- 이것 역시 closure 특성 때문이며, 이러한 문제를 Stale closure 문제라고 지칭함
### dependency에 상태 추가
따라서 최신 상태를 얻기 위해서는 리렌더링 할 때마다 `useEffect`를 실행하도록 해야 하고, `count`를 dependency에 추가한다.
```javascript
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1);
  }, 1500);
}, [count]);

useEffect(() => {
  const timer = setInterval(() => {
    console.log(count);
  }, 1500);
}, [count]);
```
![image](https://github.com/user-attachments/assets/e9b2ab9c-36a0-45da-ba28-c6baddcf0b16)
- count가 증가하면서 콘솔에 찍히기는 하지만 여전히 기대하는 동작은 아님
- 각각의 Effect가 타임 인터벌을 따로 생성하기 때문에 여러 타이머가 따로 동작하므로
### 이전 타임 인터벌 정리
Effect에서 새로운 타이머가 생성될 때 이전 타임 인터벌을 정리하면 원하는 방식으로 동작한다.
```javascript
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1);
  }, 1500);
  return () => clearInterval(timer);
}, [count]);

useEffect(() => {
  const timer = setInterval(() => {
    console.log(count);
  }, 1500);
  return () => clearInterval(timer);
}, [count]);
```
