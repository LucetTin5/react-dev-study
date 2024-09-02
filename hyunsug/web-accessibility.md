# Web Accessibility (a11y) - 웹 접근성

## Semantic HTML

[MDN - Semantics](https://developer.mozilla.org/en-US/docs/Glossary/Semantics)  
HTML 문서의 의미, 구조를 bot에 보다 정확히 전달할 수 있음

- `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`
- Headings (`<h1>` ~ `<h6>`)
- List (`<ul>`, `<ol>`, `<li>`)
- Table (`<table>`, `<th>`, `<td>`)

## ARIA (Accessible Rich Internet Applications)

[MDN - Aria](https://developer.mozilla.org/ko/docs/Web/Accessibility/ARIA)  
ARIA는 웹 콘텐츠와 웹 애플리케이션을 더 접근성 있게 만들기 위한 방법을 정의하는 규격임. 특히 동적 콘텐츠와 고급 사용자 인터페이스 컨트롤의 접근성을 향상시키는 데 중점을 둠.

### ARIA의 주요 개념

1. **Roles**: 요소의 기능을 정의함.
2. **Properties**: 요소의 특성이나 상태를 설명함.
3. **States**: 요소의 현재 상태를 나타냄.

### 주요 ARIA 속성

#### Roles

- `role="button"`: 요소를 버튼으로 정의함
- `role="navigation"`: 탐색 영역을 정의함
- `role="search"`: 검색 기능을 정의함
- `role="tablist"`, `role="tab"`, `role="tabpanel"`: 탭 인터페이스를 구성함
- `role="dialog"`: 대화 상자를 정의함

#### Properties

- `aria-label`: 요소에 대한 텍스트 설명을 제공함
- `aria-labelledby`: 다른 요소의 ID를 참조하여 레이블을 제공함
- `aria-describedby`: 추가적인 설명을 제공하는 요소를 참조함
- `aria-required`: 필수 입력 필드를 표시함

#### States

- `aria-expanded`: 확장 가능한 요소의 현재 상태를 나타냄 (true/false)
- `aria-selected`: 선택 가능한 요소의 선택 상태를 나타냄 (true/false)
- `aria-checked`: 체크박스나 라디오 버튼의 선택 상태를 나타냄
- `aria-disabled`: 요소의 비활성화 상태를 나타냄 (true/false)

### ARIA 사용 시 주의사항

1. **네이티브 HTML 요소 우선**: 가능한 한 네이티브 HTML 요소를 사용하고, 필요한 경우에만 ARIA를 추가함.

2. **적절한 역할 사용**: 요소의 실제 기능과 일치하는 ARIA 역할을 사용함.

3. **키보드 접근성 보장**: ARIA 역할을 추가할 때 해당 요소가 키보드로 접근 및 조작 가능한지 확인함.

4. **동적 업데이트**: 페이지 내용이 동적으로 변경될 때 관련 ARIA 속성도 함께 업데이트함.

5. **과도한 사용 주의**: ARIA를 과도하게 사용하면 오히려 접근성을 해칠 수 있으므로 필요한 경우에만 사용함.

6. **테스트**: 스크린 리더 및 다양한 보조 기술로 ARIA 구현을 테스트함.

ARIA를 올바르게 사용하면 동적 웹 애플리케이션의 접근성을 크게 향상시킬 수 있음. 그러나 ARIA는 기존의 HTML 시맨틱을 대체하는 것이 아니라 보완하는 것임을 항상 기억해야 함.

### React-Aria-Component, React-Aria

Adobe에서 개발한 OSS library로 aria 속성을 쉽게 사용할 수 있는 훅들을 제공 [React-Aria - Adobe](https://react-spectrum.adobe.com/react-aria/index.html)  
React-Aria-Component는 React-Aria 기반으로 제공하는 컴포넌트 모음

`Vanilla CSS`, `TailwindCSS` 등과 함께 사용하여 헤드리스 컴포넌트의 형태로 사용할 수 있음
