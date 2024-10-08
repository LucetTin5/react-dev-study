# [탈출구](https://ko.react.dev/learn/escape-hatches)

## Effect로 동기화하기

### 개발 중에 Effect가 두 번 실행되는 경우를 다루는 방법

#### 데이터 패칭

---
> Effect에서 데이터를 가져오는 좋은 대안은 무엇인가요? 

Effect 내부에서 fetch를 다룰 때 발생할 수 있는 다음과 같은 문제들을 해결해야 한다.

- 경쟁 상태 버그 영향을 받지않는 방법(서로 다른 두 요청이 경쟁하여 예상과 다른 순서로 도착함을 방지하는 것)
- 응답 캐싱 방법(사용자가 뒤로가기 버튼을 클릭하여 이전 화면을 즉시 볼 수 있도록하는 것)
- 서버에서 데이터를 가져오는 방법(초기 서버 렌더링 HTML에 스피너 대신 가져온 콘텐츠가 포함되도록 하는 것)
- 네트워크 워터폴을 피하는 방법(자식이 모든 부모를 기다리지 않고 데이터를 가져올 수 있도록하는 것)

문제해결?? <br/>
프레임워크별 특징 나는 모르겠고. 일괄적으로 react-query(=Tanstack/query)를 사용하는 방식을 취하겠다.

<br/>

#### 일반적인 사용 방법
<br/>
1. 설치

```bash
npm install @tanstack/react-query
```
<br/>
2. 쿼리 함수 정의

```javascript
// src/api.js
export const fetchUser = async (userId) => {
  const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
  if (!response.ok) {
    throw new Error('Network response was not ok');
  }
  return response.json();
};
```

<br/>
3. useQuery 훅 사용

```javascript
// src/UserComponent.js
import React from 'react';
import { useQuery } from '@tanstack/react-query';
import { fetchUser } from './api';

const UserComponent = ({ userId }) => {
  // useQuery 훅을 사용하여 데이터 패칭
  const { data, error, isLoading } = useQuery(['user', userId], () => fetchUser(userId));

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>An error occurred: {error.message}</div>;

  return (
    <div>
      <h1>{data.name}</h1>
      <p>Email: {data.email}</p>
      <p>Phone: {data.phone}</p>
    </div>
  );
};

export default UserComponent;
```

<br/>
4. QueryClientProvider 설정

```javascript
// src/App.js
import React from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import UserComponent from './UserComponent';

// 모든 쿼리의 상태를 관리하는 객체(캐시, 무효화, 리트라이 등의 로직을 포함).
const queryClient = new QueryClient();

const App = () => (
// Context API를 사용해 QueryClient 객체르 전역적으로 공급
  <QueryClientProvider client={queryClient}>
    <div>
      <h1>User Information</h1>
      <UserComponent userId={1} />
    </div>
  </QueryClientProvider>
);

export default App;
```

<br/>

#### 해결 방식

중에서! '경쟁 상태 유발가능성'만 해결해보도록 하겠습니다.

1. 경쟁 상태란?

[공식 홈페이지 제공 예시](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)

비동기 동작의 순서제어가 멋대로 일어나 오래된 데이터가 덮어쓰여지는 문제. <br/>
(이 부분은 codesandbox를 통해 예시 설명을 직접 하려고 했습니다 ㅠ)

<br/>

2. TanStack Query의 **'경쟁 상태'** 해결방식

TanStack Query는 queryKey를 통해 각 쿼리를 고유하게 식별하고, 이를 기반으로 쿼리의 상태를 관리하여 경쟁 상태를 방지한다.

쿼리의 식별자(queryKey)는 쿼리가 업데이트될 때의 식별 기준이 되며, 동일한 queryKey를 가진 쿼리는 같은 상태를 공유한다. <br/>
쿼리의 상태는 QueryCache에서 관리되며, 쿼리의 상태를 기반으로 데이터를 요청하거나 반환한다.

```typescript
import axios from 'axios';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// 사용자 정보를 가져오는 함수
const useUser = (userId) => {
  return useQuery(['user', userId], async () => {
    const response = await axios.get(`https://api.example.com/users/${userId}`);
    return response.data;
  });
};

// 사용자 정보 업데이트 함수
const updateUser = async (userId, newData) => {
  const response = await axios.patch(`https://api.example.com/users/${userId}`, newData);
  return response.data;
};

// 사용자 정보 업데이트 훅
const useUpdateUser = () => {
  const queryClient = useQueryClient();

  return useMutation(
    (userData) => updateUser(userData.userId, userData.newData),
    {
      // 성공 시 해당 사용자 정보를 무효화하여 최신 데이터로 갱신
      onSuccess: (data, variables) => {
        queryClient.invalidateQueries(['user', variables.userId]); // ⚠️ (클라이언트 캐시)데이터 무효화
      },
    }
  );
};
```
```typescript
import React, { useState } from 'react';

const UserProfile = ({ userId }) => {
  const { data, error, isLoading } = useUser(userId);
  const { mutate: updateUser, isLoading: isUpdating } = useUpdateUser();
  const [newName, setNewName] = useState('');

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  const handleUpdate = () => {
    updateUser({ userId, newData: { name: newName } });
  };

  return (
    <div>
      <h1>User Profile</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
      <input
        type="text"
        value={newName}
        onChange={(e) => setNewName(e.target.value)}
        placeholder="New name"
      />
      <button onClick={handleUpdate} disabled={isUpdating}>
        Update
      </button>
    </div>
  );
};

export default UserProfile;
```

<br/>

3. https://github.dev/TanStack/query 실제 코드

```typescript
// QueryClient.ts
getQueryData<TQueryFnData = unknown, TTaggedQueryKey extends QueryKey = QueryKey>(queryKey: TTaggedQueryKey): TInferredQueryFnData | undefined {
  const options = this.defaultQueryOptions({ queryKey });
  return this.#queryCache.get(options.queryHash)?.state.data;
}
```
```typescript
// QueryClient.ts
setQueryData<TQueryFnData = unknown, TTaggedQueryKey extends QueryKey = QueryKey>(queryKey: TTaggedQueryKey, updater: Updater<TInferredQueryFnData | undefined, TInferredQueryFnData | undefined>, options?: SetDataOptions): TInferredQueryFnData | undefined {
  const defaultedOptions = this.defaultQueryOptions({ queryKey });
  const query = this.#queryCache.get<TInferredQueryFnData>(defaultedOptions.queryHash);
  const prevData = query?.state.data;
  const data = functionalUpdate(updater, prevData);
  if (data === undefined) {
    return undefined;
  }
  return this.#queryCache.build(this, defaultedOptions).setData(data, { ...options, manual: true });
}
```

<br/>
<br/>

### useEffect와 페인트(paint)의 실행순서는 보장할 수 없다.

화요일에 커리어리에 [useEffect와 관련된 글](https://thoughtspile.github.io/2021/11/15/unintentional-layout-effect/)을 우연찮게 봐서 관련된 내용을 정리합니다.<br/>
블로그 글을 그대로 적는 것은 의미가 없으니, 읽지 않으신 분들도 초스피드로 이해할 수 있도록 작성하겠습니다.

일반적인 흐름에서 리액트의 업데이트는 순서는 다음과 같습니다.
1. 리액트 작업: 컴포넌트 렌더링 및 조정(Reconciliation) 작업을 통해 새로운 UI를 계산, <br/>
이펙트를 스케줄링한 후 실제 DOM 업데이트를 준비.
2. layoutEffect 호출
3. 브라우저 페인트: 리액트가 제어를 브라우저에 넘기고, 브라우저가 새롭게 변경된 DOM을 화면에 렌더링(paint).
4. effect 호출: useEffect가 호출되어 비동기 작업이나 DOM과 상관없는 작업을 처리.

<img src="images/effectorder.png" width="80%"/>

useEffect가 paint(화면에 렌더링) 이전에 실행될 수 있다는 점이 문제가 될 수 있는 상황이 존재합니다.

```typescript
import React, { useEffect, useState } from 'react';

function Example() {
  const [boxHeight, setBoxHeight] = useState(0);

  useEffect(() => {
    const boxElement = document.getElementById('box');
    const height = boxElement.getBoundingClientRect().height;
    setBoxHeight(height);

    // 여기서 height를 기준으로 다른 요소의 스타일을 업데이트할 수 있습니다.
    document.getElementById('another-element').style.marginTop = `${height}px`;
  }, []);

  return (
    <div>
      <div id="box" style={{ height: '100px', backgroundColor: 'lightblue' }}>
        Box
      </div>
      <div id="another-element" style={{ backgroundColor: 'lightcoral' }}>
        Another Element
      </div>
    </div>
  );
}

export default Example;
```

 #another-element의 스타일 변경에 따라 브라우저가 이 정보를 반영하기 위해 추가적인 레이아웃 계산을 수행해야 하며, <br/> 이로 인해 화면이 깜빡이거나 레이아웃이 변경되는 동안 사용자가 불안정한 UI를 보게 될 수 있습니다.

요컨데 css따위 같은 그려진 DOM에 대한 값을 조작하는 로직이 effect 내부에 들어가 있을 때 문제가 생길 수 있다는 말입니다.

따라서 위와같은 로직은 useLayoutEffect에 작성하여 paint 이전으로의 순서를 보장받아야 정상적으로 동작할 수 있습니다.

