# Let's remove Effect

## Select
```tsx
function Select({ value, setValue }){
  const [isOpen, setIsOpen] = useState<boolean>(false);
  
  const selectOption = (option: string) => {
    setValue(option);
    setIsOpen(false);
  };

  useEffect(() => {
    setIsOpen(false);
  }, [value]);
  
  return (      
    <OptionContainer $isOpen={isOpen}>
      {options?.map((item, idx) => (
        <OptionWrapper key={idx} onClick={() => selectOption(item)}>
          {item}
        </OptionWrapper>
      ))}
    </OptionContainer>
  );
}
```

setValue -> 리렌더링 -> value 변경으로 setIsOpen -> 리렌더링

selectOption 호출 시 setIsOpen(false) 되기 때문에 삭제 가능

```tsx
function Select({ value, setValue }){
  const [isOpen, setIsOpen] = useState<boolean>(false);
  
  const selectOption = (option: string) => {
    setValue(option);
    setIsOpen(false);
  };

  // 제거
  
  return (      
    <OptionContainer $isOpen={isOpen}>
      {options?.map((item, idx) => (
        <OptionWrapper key={idx} onClick={() => selectOption(item)}>
          {item}
        </OptionWrapper>
      ))}
    </OptionContainer>
  );
}
```

## WatchInput
```tsx
function WatchInput({ errorMessage }){
  const [inputState, setInputState] = useState<InputStateType>('default');

  useEffect(() => {
    if (errorMessage) {
      setInputState('error');
    } else {
      setInputState('focused');
    }
  }, [errorMessage]);

  
  const styled = {
    default: {
      borderColor: '#E5E7EB',
    },
    focused: {
      borderColor: '#76A9FA',
    },
    error: {
      borderColor: '#E02424',
    },
  };

  const inputFocusHandler = () => {
    if (errorMessage) {
      return;
    }

    setInputState('focused');
  };

  const inputBlurHandler = () => {
    if (errorMessage) {
      return;
    }

    setInputState('default');
  };
}
```

첫 렌더링 이후 리렌더링 발생


```tsx
function WatchInput({ errorMessage }){
    const [inputState, setInputState] = useState<InputStateType>(errorMessage ? 'error' : 'default');

  // 제거
  
  const styled = {
    ...
  };

  const inputFocusHandler = () => {
    ...
  };

  const inputBlurHandler = () => {
    ...
  };
}
```

## RegisterInputs
```tsx
function RegisterInputs(){
  const [cardType, setCardType] = useState("일반카드");

  useEffect(() => {
    setValue("cardType_3", cardType);
    setValue("customerIdentityNumber_4", "");
  }, [cardType]);
  
  return(
  	  ...
      <Input
        name={"cardType_3"}
        control={control}
        label={"카드유형"}
        defaultValue="일반카드"
        render={(props = {}) => <Select options={["일반카드", "법인카드"]} value={cardType} setValue={setCardType} />}
      />
  );
}
```

첫 렌더링 이후 cardType 변경 시 setValue로 리렌더링

```tsx
function RegisterInputs(){
  const [cardType, setCardType] = useState("일반카드");

  const handleCardTypeChange = () => {
    setValue('cardType_3', cardType);
    setValue('customerIdentityNumber_4', '');
  };
  
  return(
  	  ...
      <Input
        name={"cardType_3"}
        control={control}
        label={"카드유형"}
        defaultValue="일반카드"
        render={(props = {}) => 
            <Select 
              options={["일반카드", "법인카드"]} 
              value={cardType} 
              setValue={setCardType} 
              handleChange={handleCardTypeChange}
         	 />
         }
      />
  );
}


function Select({ options, placeholder, value, setValue, handleChange = () => {} }: SelectProps) {
  const selectOption = (option: string) => {
    setValue(option);
    setIsOpen(false);
    handleChange();
  };
}
```

handleChange로 이벤트 변경 발생 시에만 실행


## Card
```tsx
function Card(){
  const { outputCardNo, outputState, outputName } = billkeyInfo;
  const [cardState, setCardState] = useState<CardStateType>('unabled');

  useEffect(() => {
    setCardState(outputState === 1 ? 'abled' : 'unabled');
  }, [outputState]);
}
```

첫 렌더링 이후로 setCardState로 리렌더링

```tsx
function Card(){
  const { outputCardNo, outputState, outputName } = billkeyInfo;
  const cardState = outputState === 1 ? 'abled' : 'unabled';
}
```
state로 표현


## Cards
```tsx
function Cards(){

  const [billkeyState, setBillkeyState] = useState<BillkeyStateProps>('empty');

  const billkeys = useRecoilValue(billkeysAtom);

  useEffect(() => {
    switch (billkeys.length) {
      case 0:
        setBillkeyState('empty');
        break;
      case 1:
        setBillkeyState('primary');
        break;
      case 2:
        if (selected !== null) {
          setBillkeyState('select');
          break;
        }

        setBillkeyState('backup');
        break;
    }
  }, [billkeys, selected]);
}
```

```tsx
function Cards(){

  const [billkeyState, setBillkeyState] = useState<BillkeyStateProps>('empty');

  const billkeys = useRecoilValue(billkeysAtom);

  useEffect(() => {
    switch (billkeys.length) {
      case 0:
        setBillkeyState('empty');
        break;
      case 1:
        setBillkeyState('primary');
        break;
      case 2:
        if (selected !== null) {
          setBillkeyState('select');
          break;
        }

        setBillkeyState('backup');
        break;
    }
  }, [billkeys, selected]);
}
```

첫 렌더링 -> setBillkeyState로 리렌더링

```tsx
function Cards(){

  const getBillkeyState = () => {
    switch (billkeys.length) {
      case 0:
        return 'empty';
      case 1:
        return 'primary';
      case 2:
        if (selected !== null) {
          return 'select';
        }
        return 'backup';
    }
  };

  const billkeys = useRecoilValue(billkeysAtom);
  const billkeyState = useMemo(getBillkeyState, [billkeys.length, selected]);

```

첫 렌더링 시 함수로 값 가져오기, 메모이제이션으로 디펜던시 변경 시에만 새로운 값 계산

## Terms

```tsx
function Terms({ control }){
  const {
    field: { onChange },
  } = useController({
    name: 'terms_5',
    control,
    rules: {
      validate: value => (value ? undefined : '필수 약관에 동의해주세요.'),
    },
    defaultValue: false,
  });
  
  // 약관 배열
  const terms = useRecoilValue<TermResponse[]>(termsAtom);
  
  // 전체 동의 여부
  const [totalTermState, setTotalTermState] = useState(false);
  
  // 동의 여부 포함 약관 배열
  const [termState, setTermState] = useState<TermState[]>([]);

  // 초기 렌더링 후 termState 업데이트
  useEffect(() => {
    setTermState(terms.map(term => ({ ...term, opened: false, checked: false })));
  }, [terms]);

  // totalTermState 변경 시 부모에게 알려줌
  useEffect(() => {
    onChange(totalTermState);
  }, [totalTermState]);

  // 전체동의 클릭 시 termState 변경 & totalTermState 변경
  const totalTermClick = () => {
    setTotalTermState(origin => {
      setTermState(prev => prev.map(p => ({ ...p, checked: !origin })));
      return !origin;
    });
  }

  // 약관 동의 클릭 시 체크 변경 후 전체동의 변경
  const setTermChecked = (id: string, checked: boolean) => {
      setTermState(prev => {
      const newState = prev.map(p => (p.id === id ? { ...p, checked } : p));

      if (newState.find(state => !state.checked)) {
        setTotalTermState(false);
      } else if (newState.find(state => !state.checked) === undefined) {
        setTotalTermState(true);
      }

      return newState;
    });

  }
}
```

초기 렌더링 후 setTermState 리렌더링
onChange로 부모 state 변경하여 리렌더링

```tsx
function Terms(){
	
  const terms = useRecoilValue<TermResponse[]>(termsAtom);

  const [termState, setTermState] = useState<TermState[]>(
    terms.map(term => ({ ...term, opened: false, checked: false })),
  );

  const getTotalTermState = () => {
    let result;
    if (termState.find(state => !state.checked)) {
      result = false;
    } else {
      result = true;
    }
    onChange(result);

    return result;
  };

  const totalTermState = useMemo(getTotalTermState, [termState]);
  
  const totalTermClick = () => {
    setTermState(prev => prev.map(p => ({ ...p, checked: !totalTermState })));
  };

  const setTermChecked = (id: string, checked: boolean) => {
    setTermState(prev => prev.map(p => (p.id === id ? { ...p, checked } : p)));
  };

}
```

totalTermState -> termState에 의존
getTotalTermState로 변경, 메모이제이션

onChange -> totalTermState 변경 시에만 발생하면 되므로 getTotalTermState 내부 작성

