# Hooks pt.2

## useId

- 클라이언트와 서버 간에 안정적인 고유 ID를 생성하는 Hook

### 주요 기능

- 접근성 속성(`id`, `aria-describedby` 등)에 사용할 수 있는 유니크 ID 생성
- 서버 사이드 렌더링(SSR)과 호환

### 사용 방식

```jsx
const id = useId();
```

### 주요 특징

- 컴포넌트 내에서 여러 ID가 필요한 경우 접두사/접미사 추가 가능
- 글로벌 카운터 대신 트리 구조 기반으로 ID 생성 -> 서버-클라이언트의 트리 구조 유지 / 하이드레이션 불일치 방지

### 사용 예시

```jsx
function PasswordField() {
  const id = useId();
  return (
    <>
      <label htmlFor={id}>Password:</label>
      <input id={id} type="password" />
    </>
  );
}
```

### Array의 key값으로 _그대로_ 사용하지 말 것

- useId로 생성된 id값은 a11y(accessibility) 속성을 위한 값이며, key 값은 reconciliation 과정을 위해 존재
- 생성된 id 값을 key로 사용할 시 데이터가 변경되었어도 key 값이 동일하여 정확한 리렌더링이 일어나지 않을 수 있음

- 하지만, prefix와 같은 형태로는 권장되는 패턴

```jsx
function ItemList({ items }) {
  const idPrefix = useId();

  return (
    <ul>
      {items.map((item, index) => (
        <li key={item.id}>
          <label htmlFor={`${idPrefix}-item-${item.id}`}>{item.name}</label>
          <input
            id={`${idPrefix}-item-${item.id}`}
            type="checkbox"
            checked={item.isComplete}
          />
        </li>
      ))}
    </ul>
  );
}
```

---

## useImperativeHandle

- 정의: 부모 컴포넌트에 노출할 ref 핸들을 사용자 정의하는 Hook

### 주요 기능

- 부모 컴포넌트에 노출되는 인스턴스 값 사용자 정의
- 명령형 코드를 최소화하면서 필요한 기능 제공

### 사용 방식

```jsx
useImperativeHandle(ref, createHandle, [deps]);
```

### 주요 특징

- forwardRef와 함께 사용
- 부모 컴포넌트의 ref 접근 제어
- 컴포넌트의 내부 구현 숨기기 가능
- ref.current가 전체가 아닌 일부 메서드나 속성만 포함하게 됨

### 사용 예시

```jsx
const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    setCustomValue: (value) => {
      inputRef.current.value = value.toUpperCase();
    },
  }));
  return <input ref={inputRef} />;
});

// 부모 컴포넌트에서 사용
function Parent() {
  const fancyInputRef = useRef();

  useEffect(() => {
    fancyInputRef.current.focus();
    fancyInputRef.current.setCustomValue('hello');
  }, []);

  return <FancyInput ref={fancyInputRef} />;
}
```

---

## useInsertionEffect

- 정의: DOM 변경 전 동기적으로 실행되는 Hook

### 주요 기능

- CSS-in-JS 라이브러리 작성자를 위한 것이라 안내되고 있어 사용될 일은 거의 없을 것으로 보임
- loyoutEffect 이전에 Element를 DOM에 주입

### 사용 방식

```jsx
useInsertionEffect(didUpdate);
```

### 주요 특징

- 매우 특수한 사용 사례를 위한 Hook
- DOM 변경 전 실행되어 레이아웃 계산에 영향 주지 않음
- 일반적으로 CSS-in-JS 라이브러리 구현에만 사용 권장

### 사용 예시 (CSS-in-JS 라이브러리 내부)

```jsx
function useCSS(rule) {
  useInsertionEffect(() => {
    if (!isInserted.has(rule)) {
      isInserted.add(rule);
      document.head.appendChild(createStyleNode(rule));
    }
  });
  return rule;
}
```

---

## useLayoutEffect

- 정의: DOM 변경 후, 브라우저가 **화면을 그리기 전에** 동기적으로 실행되는 Hook

### 주요 기능

- DOM을 동기적으로 읽고 쓰는 작업 수행
- 레이아웃 측정 및 DOM 기반 애니메이션 구현

### 사용 방식

```jsx
useLayoutEffect(didUpdate);
```

### 주요 특징

- useEffect와 유사하나 실행 시점이 다름
- 동기적 실행으로 성능에 영향 줄 수 있음
- 필요한 경우에만 사용 권장

### 사용 예시

```jsx
function Tooltip() {
  const ref = useRef(null);
  useLayoutEffect(() => {
    const { width, height } = ref.current.getBoundingClientRect();
    // 툴팁 위치 조정
  }, []);
  return <div ref={ref}>Tooltip content</div>;
}
```

### 성능저하의 원인이 되는 이유는 무엇일까?

- 동기적 실행: DOM 업데이트 직후 화면이 그려지기 전에 동기적으로 실행되기에 브라우저의 페인팅 과정을 차단하게 될 수 있음
- 렌더링 지연: layoutEffect 내부 작업이 복잡하거나 오래 걸리는 작업의 경우, 화면 업데이트에 지연이 발생할 수 있음
- 레이아웃의 연속적인 재계산: layoutEffect에서 DOM이 변경되면, 레이아웃 쉬프트가 발생하여 이 과정이 반복된다면 성능저하를 일으킬 수 있음

---
