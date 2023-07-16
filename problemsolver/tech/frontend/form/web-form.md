---
description: 2023.05.02.
---

# Web에서의 Form이란?

Web에서 Form을 제작하는 것은 총 3단계로 나눌 수 있습니다.

* Structure web form
  * [How to structure a web form](https://developer.mozilla.org/en-US/docs/Learn/Forms/How\_to\_structure\_a\_web\_form)
* Validation
  * [Client-side form validation](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form\_validation)
* Send Data (Submit)
  * [Sending form data](https://developer.mozilla.org/en-US/docs/Learn/Forms/Sending\_and\_retrieving\_form\_data)
  * [Sending form through JS](https://developer.mozilla.org/en-US/docs/Learn/Forms/Sending\_forms\_through\_JavaScript)



Form을 구축하고, 올바른 값인지 검증하며, 검증된 값을 보내는 3단계라고 할 수 있습니다.



## 1. Structure web form

Form을 구축하는 것은 크게 2가지로 나눌 수 있습니다.

1. **`<form>`** 태그로 글로벌 구조를 형성
2. 상황에 맞게 **HTML 위젯** input, textarea ...etc 구성



### I. Form 구조 형성

Web에서 Form을 다루기 위해서는 form을 선언해야하는데, 이 때 공식적으로 폼을 정의하는 요소로 이 \<form> 태그를 사용합니다. form을 다루며 사용하게 되는 브라우저 플러그인이나 기술을 사용할 때 form을 쉽게 찾을 수 있기 때문입니다.



### II. HTML 위젯 구성

사용자로부터 어떤 데이터를 받을지에 따라서 원하는 위젯을 사용할 수 있습니다.

위젯을 구성할 때는 label + input 형식으로 정의할 수 있습니다.&#x20;

#### [Label](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label) 구성 방식

<pre class="language-tsx"><code class="lang-tsx"><strong>//명시적
</strong><strong>&#x3C;label for="cash>캐시&#x3C;/label>
</strong>&#x3C;input type="text" name="cash" />

//암시적
&#x3C;label>
    &#x3C;input type="text" name="cash" />
&#x3C;/label>
</code></pre>

label을 input과 연관시키는 이유가 무엇일까요?

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>google form</p></figcaption></figure>

* label의 텍스트가 현재 Input에 무엇을 입력해야하는지 알기 쉽습니다.
* label을 클릭하여 Input을 focus 할 수 있습니다. 모바일 디바이스처럼 터치 스크린에서는 더욱 큰 장점이 됩니다.





## 2. Validation

서버에 데이터를 전송하기 전에, 필요한 값을 전부 입력했는지, 올바른 형식으로 입력했는지 등 검증하는 과정이 필요합니다. 이것을 **client-side form validation** 이라고 합니다. (전송한 이후에 서버에서 검증하는 것은 **server-side validation**이라고 합니다.)

서버에 올바른 값을 보내기 위함도 있지만 Form을 입력하는 사용자로 하여금 올바른 값을 입력했는지 명시해주기 위함도 있습니다.&#x20;

<figure><img src="../../../../.gitbook/assets/image (13).png" alt=""><figcaption><p>google form</p></figcaption></figure>

현재 잔액을 입력할 때 부족하지 않은지, 최소 or 최댓값을 충족하는지, 숫자로 입력했는지를 고려할 수 있습니다.

Validation을 수행할 수 있는 방법은 built-in form validation, javascript로 총 2가지가 있습니다.



### I. Built-in form validation

HTML의 form validation 기능을 활용하는 것입니다.

```tsx
  <input
    name="cash"
    type="number"
    inputMode="numeric"
    pattern="[0-9]*"
    maxLength={2}
    max={23}
    min={0}
  />
```

예를 들어 input 태그 안에서 type을 지정하고 최소, 최댓값을 지정하는 것처럼 사용할 수 있습니다.

HTML validation의 성능은 JS보다 좋지만, 커스텀이 JS를 활용하는 것만큼 다양하진 않습니다. 따라서 기본적인 validation은 HTML built-in validation으로 진행하고 구체적인 조건이 필요한 경우에는 JS를 활용해 커스텀합니다.



### II. Javascript

HTML의 validation 코드 자체가 Javascript로 작성되었기 때문에 JS를 사용해 커스텀이 가능하지만 작성해야하는 양이 늘어날 수 있습니다. 따라서 라이브러리를 사용해 커스텀하는 것이 도움이 됩니다.



### + controlled vs uncontrolled

validation을 통해 input value를 검증할 때의 시점에 따라 제어 혹은 비제어 컴포넌트를 사용할지 정할 수 있습니다.

예를 들어 사용자의 입력이 있을 때마다 컴포넌트가 re-render 되는 제어 컴포넌트는 실시간으로 값을 검증할 수 있지만 비제어 컴포넌트는 값을 검증하는 trigger를 하기 전까지는 값을 알 수 없기 때문에 실시간으로 값 검증이 불가능합니다.



## 3. Send Data (Submit)

Form의 data가 client-side validation을 마쳤다면, 폼을 submit 하기 위한 준비가 되었다는 뜻입니다.

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption><p><a href="https://developer.mozilla.org/en-US/docs/Learn/Forms/Sending_and_retrieving_form_data#clientserver_architecture">https://developer.mozilla.org/en-US/docs/Learn/Forms/Sending_and_retrieving_form_data#clientserver_architecture</a></p></figcaption></figure>

data를 서버에 HTTP 프로토콜을 사용하여 보낼 때 HTML form은 정말 **유저 친화적**입니다.

form 태그 안에 action과 method(GET, POST)를 활용해 data를 보낼 수 있습니다. 이 때 요청에 대해서는 network탭에서 확인할 수 있습니다.

GET요청은 network 탭에서 보낸 값을 확인할 수 있는데, 만약 비밀번호처럼 보안이 중요한 데이터를 보낼 경우 GET요청에는 함께 보내면 안됩니다. 또한, 많은 양의 데이터를 보낼 경우 url size limit이 있기 때문에 POST요청으로 보내야합니다.



저는 action과 method를 활용하여 사용하기 보다는 Javascript를 활용해 보내는 것을 선호합니다.

React를 활용한 SPA로 Web을 만들다보면 위 method를 사용해서 새로운 document를 만들어 **full page load가 일어날 필요가 없기 때문**입니다.

따라서 **Modern UI**는 유저로부터 **input 값을 얻기 위해 HTML form을 활용**하고 데이터를 보낼 때는 Application이 background에서 데이터를 제어하고 비동기적으로 제출하여 UI 업데이트가 필요한 부분만 변경합니다.

React를 활용한다면 form의 `onSubmit` 이벤트를 활용하여 비동기적으로 data를 request하는 함수를 작성함으로써 적용할 수 있습니다. 이 때 사용자는 **full page load를 겪지 않고** **필요한 UI만 업데이트**되기 때문에 사용자 경험이 증가할 수 있습니다.



다음 시간에는 Form을 응용하여 관리한 사례에 대해서 더욱 구체적으로 경험을 서술해보겠습니다.
