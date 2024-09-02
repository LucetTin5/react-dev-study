HTML을 사용하여 웹 페이지를 제작할 때 고려해야 하는 것 중 하나가 **웹 접근성**이다.
> 웹 접근성: 장애를 가진 사람과 장애를 가지지 않은 사람 모두가 웹 사이트를 이용할 수 있게 하는 방식

예를 들어, 시각장애인들은 직접 클릭하는 것이 어려우므로 탭 버튼을 눌러서 각 요소를 이동할 것이다. 

따라서 `<input />`, `<label />` 등에 적절한 property를 추가하여 탭을 눌렀을 때 초점이 어느 순서로 이동하는지, 초점이 잡혔을 때의 안내 메시지가 제대로 나오는지 등의 여부는 굉장히 중요히다.

이런 설정들은 HTML native 요소만으로는 처리하기 어렵기 때문에 W3C는 [WAI-ARIA](https://developer.mozilla.org/ko/docs/Web/Accessibility/ARIA)라는 걸 정의했다.

<br/>

# WAI-ARIA
Web Accessibility Initiative–Accessible Rich Internet Applications
- HTML element에 따라서 특정 환경(스크린 리더 같은 보조기기 등)에서 제대로 읽히지 않는 경우가 있다.
- WAI-ARIA는 이를 개선하기 위해 웹 애플리케이션에 **역할(Role), 속성(Property), 상태(State)** 정보를 추가한다.

### 사용의 이점
WAI-ARIA는 웹 접근성 개선 뿐만 아니라 다양한 측면에서의 장점을 갖고 있다.
- 시맨틱 마크업 강화: HTML 의미를 명시적으로 정의하여 개발자가 구조를 명확하게 표현할 수 있도록 한다.
- 검색 엔진 최적화: HTML의 의미를 명시하면 검색 엔진이 웹 페이지 구조를 잘 해석하여 검색 결과에 정확한 정보를 제공할 수 있다.
- 사용자 경험 향상: 사용자가 콘텐츠에서 필요한 정보를 찾기 위해 빠르게 접근이 가능해져 사용자 경험 향상에 도움이 된다.

<br/>

# Role
Role은 `role`을 사용하여 이름 그대로 HTML element의 역할을 정의한다.
```html
<form role="search">
    <label for="search-input">Search:</label>
    <input type="text" id="search-input" name="search" placeholder="Search...">
    <button type="submit">Submit</button>
</form>
```
- 일반적으로는 HTML native 요소만으로 이를 처리하는 것이 가장 이상적이다.
- 예시처럼 `<form />`을 검색 기능을 수행하도록 하는 다양한 설정이 필요한 경우 `role`을 사용해서 역할을 명시한다.
- Role의 종류는 매우 다양하며 MDN 문서 [WAI-ARIA Roles](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles)에서 확인 가능하다.
### 사용 시 주의할 점
`role`은 추가적인 의미와 **역할을 부여하는 보조수단**이며 HTML 기본 동작을 변경하거나 무시하면 안된다.
```html
<!-- 👍 버튼의 의미와 역할이 명확하게 전달 -->
<button type="button" onclick="alert('버튼이 클릭되었습니다.')">클릭하세요</button>

<!-- 👎 div의 원래 의미적인 역할을 갖고 있지 않음 -->
<div role="button" tabindex="0" onclick="alert('버튼이 클릭되었습니다.')">클릭하세요</div>
```
- 위 예시처럼 HTML 요소의 의미와 역할을 보존하고, HTML 표준과의 호환성을 유지하도록 해야 한다.
- - 오남용하면 안된다. HTML element 자체로 충분히 의미를 전달할 수 있다면 `role`을 사용하지 않는다.
### 태그별 내장된 암묵적인 role
HTML의 각 태그별로 암묵적으로 의미하는(갖고 있는) role이 있다. 
```html
<a href="{link}"></a>  <!-- 암묵적으로 role = link를 가지고 있다.-->
<article>...</article>  <!-- 암묵적으로 role = article 가지고 있다.-->
<fieldset>...</fieldset>  <!-- 암묵적으로 role = group 가지고 있다.-->
```
- W3C 문서 [ARIA in HTML](https://www.w3.org/TR/html-aria/)에서 내장되어 있는 role 목록을 확인할 수 있다.

<br/>

# State and Property
Property는 해당 element의 특징이나 상황을 정의하며 `aria-` 접두사를 사용하며, State는 현재 상태를 나타낸다.
```html
<!-- property state example 1 -->
<li role="checkbox" aria-checked="true">체크박스 아이템</li>

<!-- property state example 2 -->
<div role="alert" aria-live="assertive">올바르지 않은 입력입니다.</div>
```
- ARIA 속성 역시 종류가 매우 다양하며 MDN 문서 [ARIA states and properties](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes)에서 확인 가능하다.
### 사용 시 주의할 점
ARIA 속성 역시 **속성을 정의하는 보조수단**이며 HTML의 기본 동작을 우선시 해줘야 한다.
```html
<!-- 👍 일반적인 상황에서는 disabled를 우선적으로 사용 -->
<button disabled="true">비활성화된 버튼</button>

<!-- 👎 disabled를 사용할 수 있는 상황이라면 적절하지 않음 -->
<button aria-disabled="true">비활성화된 버튼</button>
```
- 위 예시에서 일반적인 상황이라면 disalbed를 우선적으로 사용하는 것이 적절하다.
- 클릭만 비활성화하고 스타일링, 포커스 제어 등에선 활성화해야 되는 특별한 경우엔 사용을 고려할 수 있다.
