## [useCallback](https://ko.react.dev/reference/react/useCallback)

### 최적화가 유의미한 성능이 보일 수 있는 시점

cpt 참고

<br/>
<br/>

### 메모이제이션 함수로 인해 발생할 수 있는 에러

어디가 문제일까요?

```typescript
export default function App() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);

  const increment = useCallback(() => {
    setCount((c) => c + step);
  }, []);

  return (
    <div>
      <h1>Count: {count}</h1>
      <input
        type="number"
        value={step}
        onChange={(e) => setStep(parseInt(e.target.value))}
      />
      <Counter onIncrement={increment} />
    </div>
  );
}
```

```typescript
import React, { useState, useCallback } from "react";

function Counter({ onIncrement }) {
  return <button onClick={onIncrement}>Increment</button>;
}
```

<br/>
<br/>
