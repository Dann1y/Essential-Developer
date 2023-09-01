---
description: 2023.09.01.
---

# Babel과 Polyfill

{% hint style="info" %}
SWC가 해주다보니 잊고 있던 개념을 정리하기 위한 글입니다..
{% endhint %}

## 1. Babel이란?

### Babel 개념

[https://babeljs.io/docs/](https://babeljs.io/docs/)

Babel은 ES6+ 코드를 구형 브라우저 환경에서 호환할 수 있도록 변경하기 위해서 주로 사용합니다.&#x20;

* Syntax를 변경하거나
* 타겟 환경에서 제공하지 않는 폴리필을 제공하거나 (core-js)
* source 코드를 변경합니다.

```javascript
// ES6 arrow function
[1,2,3].map(v => v + 1);

// ES5 환경
[1,2,3].map(function(v) {
    return + 1;
});
```

### Babel이 주로 쓰이는 곳

저는 주로 React 환경에서 개발을 하기 때문에 React 기준으로 설명을 드리겠습니다.

```jsx
// JSX
const Button = ({index, children}) => {
  return (
    <div index={index}>
      {children}
    </div>
  )
}

// 다음과 같이 변환
const Button = ({ id, children }) => {
  return React.createElement(
    "div",
    {
      idndex: index,
    },
    children
  );
}
```

최근에는 Babel을 다룰 일이 없다보니 잊고 있던 내용이 있습니다.\
JSX 구문이 실제로는 `React.createElement` 함수 내부에 type, props, children 순으로 들어가서 render되는 것은 이해를 하고 있었는데 브라우저가 이해할 수 있도록 transpile 하는 과정을 Babel이 한다는 사실을 잊고 있었습니다.

### Transpile 하는 이유

JSX 코드를 있는 그대로 script 태그에서 실행시키게 되면 JSX의 `<` 이 부등호를 이해를 할 수 없기 때문에 SyntaxError를 발생시키게 됩니다.

따라서 브라우저가 실행할 수 있도록 JSX에서 Javascript로 변경해야할 필요가 있습니다.

### 따라서

* Transpile
  * JSX 같은 형식의 코드를 브라우저가 읽을 수 있는 Transpiler의 역할을 하며
* Polyfill
  * 구형 브라우저가 지원하지 않는 ES6+ 의 문법을 ES5 형식으로 Polyfill을 제공하기도 합니다.
  * core js, polyfill.io (사용자의 브라우저에 따라 Polyfill script 제공)



## Polyfill이란?

[https://developer.mozilla.org/ko/docs/Glossary/Polyfill](https://developer.mozilla.org/ko/docs/Glossary/Polyfill)

이왕 Babel 정리한 김에 헷갈릴 수 있는 Polyfill까지 한번에 정리하겠습니다.

Polyfill은 지원하지 않는 이전 브라우저에서 최신 기능을 제공할 때 필요한 코드입니다.



## 둘의 차이점은?
