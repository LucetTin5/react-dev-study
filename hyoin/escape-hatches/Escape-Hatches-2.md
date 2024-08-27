[탈출구(useEffect) - 노션 링크](https://cooing-dust-8b6.notion.site/Escape-Hatches-useEffect-987993dbc6cb4fed9ab705d831ba0f7a?pvs=4)

### useEffect란

렌더링 자체에 의해 발생하는 부수효과를 특정

외부 시스템과 동기화할 때 사용

1. useEffect 선언 : 모든 commit 이후 실행되는 Effect 선언
    
    ```jsx
    import {useEffect} from 'react`;
    
    function Mycomponent(){
    	useEffect(()=>{});
    	return <>;
    }
    ```
    
    만약 useRef같이 DOM노드 조작하기 위해서는 렌더링 후에 조작이 필요함. 
    
    ```jsx
    import { useEffect, useRef } from 'react';
    
    function VideoPlayer({ src, isPlaying }) {
      const ref = useRef(null);
    
      useEffect(() => {
        if (isPlaying) {
          ref.current.play();
        } else {
          ref.current.pause();
        }
      });
    
      return <video ref={ref} src={src} loop playsInline />;
    }
    ```
    
    ```jsx
    const [count, setCount] = useState(0);
    useEffect(() => {
      setCount(count + 1);
    });
    ```
    
    와 같은 Effect 선언은 무한루프를 만들 수 있으니 주의
    
2. 의존성 배열 지정
    
    useEffect의 두번째 인자로 의존성 배열을 지정해야함
    
    ```jsx
    useEffect(()=>{},[]);
    ```
    
    반응형값이 useEffect에 있을 경우 의존성배열에 모두 추가해야 lint 에러가 나지않음
    
    항상 같은 객체를 얻을수 있는 것은 의존성 배열에 추가하지 않아도됨. (ex) ref )
    
    ```jsx
    useEffect(() => {
      // 모든 렌더링 후에 실행됩니다
    });
    
    useEffect(() => {
      // 마운트될 때만 실행됩니다 (컴포넌트가 나타날 때)
    }, []);
    
    useEffect(() => {
     // 마운트될 때 실행되며, *또한* 렌더링 이후에 a 또는 b 중 하나라도 변경된 경우에도 실행됩니다
    }, [a, b]);
    ```
    
    의존성배열 유무에 따라 동작이 다르다.
    
3. 필요할 경우 클린업 함수 추가
    1. connection과 같이 disconnect가 반드시 필요할 경우에는 useEffect 내의 콜백의 return에 클린업 함수를 추가해준다.
        
        ```jsx
          useEffect(() => {
            const connection = createConnection();
            connection.connect();
            return () => connection.disconnect();
          }, []);
        ```
        
    2. effect에서 어떤 것을 구독할경우에서는, 클린업함수에서 구독을 해지해야함
        
        ```jsx
        useEffect(() => {
          function handleScroll(e) {
            console.log(window.scrollX, window.scrollY);
          }
          window.addEventListener('scroll', handleScroll);
          return () => window.removeEventListener('scroll', handleScroll);
        }, []);
        ```
        
    3. 애니메이션 트리거할 경우 클린업에서는 애니메이션을 초기값으로 재설정이 필요
        
        ```jsx
        useEffect(() => {
          const node = ref.current;
          node.style.opacity = 1; // Trigger the animation
          return () => {
            node.style.opacity = 0; // Reset to the initial value
          };
        }, []);
        ```
        
    4. 데이터 페칭의 경우 클린업함수에서는 fetch를 중단하거나 결과를 무시해야함
        
        ```jsx
        useEffect(() => {
          let ignore = false;
        
          async function startFetching() {
            const json = await fetchTodos(userId);
            if (!ignore) {
              setTodos(json);
            }
          }
        
          startFetching();
        
          return () => {
            ignore = true;
          };
        }, [userId]);
        ```
        

### eventHandler를 useEffect로 다루지 않도록 주의

- 렌더링을 위해 데이터를 변환하는 것에는 Effect가 필요하지 않음
- 사용자 이벤트 처리하는데에는 Effect가 필요하지 않음

- 만약 앱 실행시 딱 한번 실행되어야하는 함수가 있을시, 최상위 변수를 이용할 것 (개발단계에서 effect 두번 실행될 때를 방지)
    
    ```jsx
    let didInit = false;
    
    function App() {
      useEffect(() => {
        if (!didInit) {
          didInit = true;
          // ✅ 앱 로드당 한 번만 실행
          loadDataFromLocalStorage();
          checkAuthToken();
        }
      }, []);
      // ...
    }
    ```
    
- 외부 저장소를 구독해야할 경우 내장 hook(`useSyncExternalStore`)으로 외부 스토어를 구독하기.
    
    ```jsx
     function useOnlineStatus() {
      // 이상적이지 않습니다: Effect에서 저장소를 수동으로 구독
      const [isOnline, setIsOnline] = useState(true);
      useEffect(() => {
        function updateState() {
          setIsOnline(navigator.onLine);
        }
    
        updateState();
    
        window.addEventListener('online', updateState);
        window.addEventListener('offline', updateState);
        return () => {
          window.removeEventListener('online', updateState);
          window.removeEventListener('offline', updateState);
        };
      }, []);
      return isOnline;
    }
    
    function ChatIndicator() {
      const isOnline = useOnlineStatus();
      // ...
    }
    ```
    
    ⇒
    
    ```jsx
    function subscribe(callback) {
      window.addEventListener('online', callback);
      window.addEventListener('offline', callback);
      return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
      };
    }
    
    function useOnlineStatus() {
      // ✅ 좋습니다: 내장 Hook으로 외부 스토어 구독하기
      return useSyncExternalStore(
        subscribe, // 동일한 함수를 전달하는 한 React는 다시 구독하지 않습니다.
        () => navigator.onLine, // 클라이언트에서 값을 얻는 방법
        () => true // 서버에서 값을 얻는 방법
      );
    }
    
    function ChatIndicator() {
      const isOnline = useOnlineStatus();
      // ...
    }
    ```
    
- 데이터를 가져와야할 경우 경쟁 조건을 수정하기 위해 정리함수를 추가해준다. (오래된 응답을 무시하게끔)
    
    ```jsx
      useEffect(() => {
        let ignore = false;
        fetchResults(query, page).then(json => {
          if (!ignore) {
            setResults(json);
          }
        });
        return () => {
          ignore = true;
        };
      }, [query, page]);
    ```
    
    custom hook으로 빼는 방법도 있다.
    
    ```jsx
    function useData(url) {
      const [data, setData] = useState(null);
      useEffect(() => {
        let ignore = false;
        fetch(url)
          .then(response => response.json())
          .then(json => {
            if (!ignore) {
              setData(json);
            }
          });
        return () => {
          ignore = true;
        };
      }, [url]);
      return data;
    ```
    

### Effect의 생명주기

- React 컴포넌트 생명주기
    - 컴포넌트가 화면에 추가될때 `mount`
    - 새로운 props나 state를 수신하면 `update`
    - 컴포넌트가 화면에서 제거되면 `unmount`
- 이펙트의 생명주기는 컴포넌트의 생명주기는 별도의 생명주기를 가짐
- 각 effect는 동기화를 시작 & 중지할 수 있는 별도 동기화 프로세스가 있음