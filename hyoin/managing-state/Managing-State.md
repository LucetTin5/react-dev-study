[State 관리하기 - 노션 링크](https://cooing-dust-8b6.notion.site/Managing-State-de564dfccfeb4ddd9cb99b07d6b972e0?pvs=4)

전역 상태관리 라이브러리 구조(reducer, context 등으로 어떻게 구성되어있는지) 조사해보기

리코일 조사햇슴다.. reducer는 안쓰지만 context는 씀

### 리코일 루트에서 context 생성

[**Recoil**](https://github.com/facebookexperimental/Recoil/tree/c1b97f3a0117cad76cbc6ab3cb06d89a9ce717af)/[packages](https://github.com/facebookexperimental/Recoil/tree/c1b97f3a0117cad76cbc6ab3cb06d89a9ce717af/packages)/[recoil](https://github.com/facebookexperimental/Recoil/tree/c1b97f3a0117cad76cbc6ab3cb06d89a9ce717af/packages/recoil)/[core](https://github.com/facebookexperimental/Recoil/tree/c1b97f3a0117cad76cbc6ab3cb06d89a9ce717af/packages/recoil/core)/**Recoil_RecoilRoot.js**

https://github.com/facebookexperimental/Recoil/blob/c1b97f3a0117cad76cbc6ab3cb06d89a9ce717af/packages/recoil/core/Recoil_RecoilRoot.js

```tsx
/// line 74
const defaultStore: Store = Object.freeze({
  storeID: getNextStoreID(),
  getState: notInAContext,
  replaceState: notInAContext,
  getGraph: notInAContext,
  subscribeToTransactions: notInAContext,
  addTransactionMetadata: notInAContext,
});

/// line 121
const AppContext = React.createContext<StoreRef>({current: defaultStore});
const useStoreRef = (): StoreRef => useContext(AppContext);

...

/// line 362
function RecoilRoot_INTERNAL(...){
	...

	/// line 518
  return (
    <AppContext.Provider value={storeRef}>
      <Batcher setNotifyBatcherOfChange={setNotifyBatcherOfChange} />
      <Suspense fallback={<RecoilSuspenseWarning />}>{children}</Suspense>
    </AppContext.Provider>
  );
}

/// line 552
function RecoilRoot(props: Props): React.Node {
  const {override, ...propsExceptOverride} = props;

  const ancestorStoreRef = useStoreRef();
  if (override === false && ancestorStoreRef.current !== defaultStore) {
    // If ancestorStoreRef.current !== defaultStore, it means that this
    // RecoilRoot is not nested within another.
    return props.children;
  }

  return <RecoilRoot_INTERNAL {...propsExceptOverride} />;
}

/// line 569
module.exports = {
  RecoilRoot,
  useStoreRef,
  useRecoilStoreID,
  notifyComponents_FOR_TESTING: notifyComponents,
  sendEndOfBatchNotifications_FOR_TESTING: sendEndOfBatchNotifications,
};
```

RecoilRoot 컴포넌트를 호출 시 RecoilRoot_INTERNAL 컴포넌트 호출 ⇒ AppContext context 생성 & provide

### useRecoilState , useRecoilValue

```tsx
/// line 262

function useRecoilValueLoadable<T>(recoilValue: RecoilValue<T>): Loadable<T> {
  if (__DEV__) {
    validateRecoilValue(recoilValue, "useRecoilValueLoadable");
  }
  if (!recoilValuesUsed.current.has(recoilValue.key)) {
    recoilValuesUsed.current = setByAddingToSet(recoilValuesUsed.current, recoilValue.key);
  }
  // TODO Restore optimization to memoize lookup
  const storeState = storeRef.current.getState();
  return getRecoilValueAsLoadable(
    storeRef.current,
    recoilValue,
    reactMode().early ? storeState.nextTree ?? storeState.currentTree : storeState.currentTree
  );
}

function useRecoilValue<T>(recoilValue: RecoilValue<T>): T {
  if (__DEV__) {
    validateRecoilValue(recoilValue, "useRecoilValue");
  }
  const loadable = useRecoilValueLoadable(recoilValue);
  return handleLoadable(loadable, recoilValue, storeRef);
}

function useRecoilState<T>(recoilState: RecoilState<T>): [T, SetterOrUpdater<T>] {
  if (__DEV__) {
    validateRecoilValue(recoilState, "useRecoilState");
  }
  return [useRecoilValue(recoilState), useSetRecoilState(recoilState)];
}
```

### setRecoilValue

```tsx
function queueOrPerformStateUpdate(store: Store, action: Action<mixed>): void {
  if (batchStack.length) {
    const actionsByStore = batchStack[batchStack.length - 1];
    /// 최신 배치스택 map에서 인자로 받은 store에 대한 action을 찾음
    let actions = actionsByStore.get(store);
    /// actions array, 존재하지 않으면 defaultValue 넣어줌
    if (!actions) {
      actionsByStore.set(store, (actions = []));
    }

    actions.push(action);
    ///actions array에 push
  } else {
    applyActionsToStore(store, [action]);
  }
}

...
const batchStack: Array<Map<Store, Array<Action<mixed>>>> = [];
...

function setRecoilValue<T>(
  store: Store,
  recoilValue: AbstractRecoilValue<T>,
  valueOrUpdater: T | DefaultValue | (T => T | DefaultValue),
): void {
  queueOrPerformStateUpdate(store, {
    type: 'set',
    recoilValue,
    valueOrUpdater,
  });
}
```

### State를 사용해 Input 다루기

- 선언형 UI / 명령형 UI :
  선언형 프로그래밍 : UI를 세세하게 조작(명령형)하지 않고, 각각의 시각적 state를 통해 UI를 묘사
  1. 컴포넌트의 다양한 시각적 state 확인
  2. 무엇이 state 변화를 트리거하는지 알아내기

     ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/398391fa-6c95-4271-8669-27c8262c6ad0/de0c21c4-70ee-4300-be47-5f08704b2f24/Untitled.png)

  3. 메모리의 state를 useState로 표현하기
  4. 불필요한 state 변수를 제거하기

     > state가 역설을 일으키지는 않나요?
     >
     > 다른 state 변수에 이미 같은 정보가 담겨있진 않나요?
     >
     > 다른 변수를 뒤집었을 때 같은 정보를 얻을 수 있진 않나요?

  5. state 설정을 위해 이벤트 핸들러 연결하기

### State 구조 선택하기

- State 구조화 원칙
  1. 연관된 State 그룹화 : 두 개의 변수가 항상 함께 변경될 경우 단일 State로 관리
  2. State 모순 피하기
  3. 불필요한 State 피하기
  4. State 중복 피하기
  5. 깊게 중첩된 State 피하기

### 컴포넌트 간 State 공유하기

- 부모 컴포넌트에서 state를 선언하고 props로 전달할 경우 컴포넌트 간 state를 공유할 수 있다.
- 컴포넌트를 props로 부터 제어할지 state로 부터 비제어할지 고려하면 유용하다.

### State를 보존하고 초기화하기

- React는 UI 안에 있는 컴포넌트 구조로 **렌더 트리**를 만듦

  이 때 React가 UI 트리 상의 위치와 state를 연결해줌

  React가 컴포넌트를 제거할 때 state도 같이 제거함

  ⇒ **React는 JSX 마크업에서가 아닌 UI 트리에서의 위치에 관심이 있다**

- 같은 자리의 **같은 컴포넌트**는 state를 보존

  같은 자리의 **다른 컴포넌트**는 state를 초기화

- key값을 이용해 state를 초기화할 수 있음 (다른 key일 경우 다른 컴포넌트)

  하지만 key값이 전역적으로 유일하지 않고, 부모 안에서의 자리만 명시

- 컴포넌트를 제거했을 때 state를 보존하고 싶다면?
  - css 상에서만 안보이게 하기
  - state를 상위로 올리고 부모 컴포넌트에서 관리 ⇒ 제일 일반적
  - state가 아닌 다른 저장소를 이용 (ex) 로컬, 세션, 쿠키…

### State 로직을 reducer로 작성하기

- 변수와 여러 핸들러들을 한번에 관리 ⇒ reducer 함수로 옮겨서 관리
  1. state를 설정하는 것에서 action을 dispatch 함수로 전달하는 것으로 **변경**

     dispatch 함수에 넣어준 객체 : action. type에는 어떤 일이 발생했는 지 description을 작성

     ```jsx
     function handleDeleteTask(taskId) {
       dispatch(
         // "action" 객체:
         {
           type: "deleted",
           id: taskId,
         }
       );
     }
     ```

  2. reducer 함수 **작성**

     reducer함수 : state에 관한 로직을 넣는 곳

     현재 state값, action객체 두개의 인자를 받고 다음 state 값을 반환

     if else, switch문 등을 통해 각 action type에 따른 return을 정의해준다.

     ```jsx
     function tasksReducer(tasks, action) {
       switch (action.type) {
         case "added": {
           return [
             ...tasks,
             {
               id: action.id,
               text: action.text,
               done: false,
             },
           ];
         }
         case "changed": {
           return tasks.map((t) => {
             if (t.id === action.task.id) {
               return action.task;
             } else {
               return t;
             }
           });
         }
         case "deleted": {
           return tasks.filter((t) => t.id !== action.id);
         }
         default: {
           throw Error("Unknown action: " + action.type);
         }
       }
     }
     ```

  3. 컴포넌트에서 reducer **사용**

     initial state를 인자로 받는 것은 동일하나, reducer함수를 인자로 받는 것이 차이

     또한 return값이 setState가 아닌, dispatch(액션을 reducer에 전달하는 함수) 함수인 것이 차이

     ```jsx
     const [tasks, setTasks] = useState(initialTasks);
     /// useState 사용시
     const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
     /// useReducer 사용시
     ```
- useState와 useReducer 비슷하게 사용 but 차이점이 있다.
  (차이점이 아니라 React피셜 useReducer 장점)
  - 코드크기
  - 가독성
  - 디버깅
  - 테스팅
  - 개인적인 취향
  - 근데 이거 리액트에서 Reducer 를 너무 밀어주는거 아닌가

### Context를 사용해 데이터를 깊게 전달하기

- 전역으로 상태를 관리 & props drilling 방지
- Props Drilling
  ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/398391fa-6c95-4271-8669-27c8262c6ad0/a8d45206-5548-445c-9079-839a32dc65e2/Untitled.png)
  상위 컴포넌트로 끌어올린 state를 props로 전달 & 전달하며 사용해야 하는 현상
  ⇒ useContext 훅을 통해 해소 가능
- context
  ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/398391fa-6c95-4271-8669-27c8262c6ad0/297ddc06-9873-48b7-8c0c-bae3eaf4f3d5/Untitled.png)
  1. Context를 생성 `createContext()` , defaultValue를 인자로 받음

     `export const LevelContext = createContext(1);`

  2. 데이터가 필요한 컴포넌트에서 context 사용 `useContext()` , 1. 에서 생성한 context를 이용

     `const level = useContext(LevelContext);`

  3. 데이터를 지정하는 컴포넌트에서 context를 제공(context Provider이용)

     ```tsx
     <LevelContext.Provider value={level}>{children}</LevelContext.Provider>
     ```

###
