---
description: 2023.04.28.
---

# 웹 안에 또 다른 웹, iframe과 실시간 소통을 하며 겪은 문제

Iframe과 통신하며 겪은 문제를 풀어나가며 겪은 고민과 해결 과정을 공유하겠습니다.



## 문제

* 실시간 iframe 통신
* Origin
  * X-Frame-Options
  * Content-Security-Policy



### 1. iframe이 불러와져야 뭐라도 하지..

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption><p><a href="https://stackoverflow.com/questions/66463781/how-do-you-stop-an-iframe-from-saying-site-refused-to-connect">https://stackoverflow.com/questions/66463781/how-do-you-stop-an-iframe-from-saying-site-refused-to-connect</a></p></figcaption></figure>

iframe을 불러올 때 iframe의 도메인이 연결을 거부하는 상황이 있었습니다.

X-Frame-Options가 sameorigin이었음에도 위 문제가 발생하여 iframe을 불러올 수 없었습니다.



[**X-Frame-Options**](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/X-Frame-Options)가 왜 있을까요?

* **DDOS 공격 방지**&#x20;
  * 개발의 편의성 때문에 보안을 잊을 때가 있지만 개발자들이 보안을 잊지 않게 하기 위해서 이러한 옵션을 만들었습니다.
  * 예를 들어 iframe 태그를 극단적으로 몇십만개를 만들고 계속 로드를 반복하면 무수히 많은 request가 해당 도메인으로 갈 것이고 이는 DDOS 공격에 취약한 구조입니다.
  * 따라서 X-Frame-Options로 같은 도메인, 혹은 허용 자체를 하지 않음으로써 이러한 취약점을 보완할 수 있습니다.
* [**Clickjacking** ](https://en.wikipedia.org/wiki/Clickjacking)방지
  * 축구를 좋아해서 하이재킹에 대한 개념을 알고 있었기 때문에 이름만으로 무엇을 방지하는지 유추할 수 있었습니다.
  * ![](<../../.gitbook/assets/image (2) (1) (1).png>)
  * 투명 레이어 위에 다른 페이지를 로드하여 사용자에게 보이는 것과 실제 동작하는 것이 다른 것을 방지합니다.
  * 사이트 내 콘텐츠들이 다른 사이트에 포함되지 않도록 X-Frame-Options로 해결할 수 있기 때문입니다.



### 2. 실시간으로 iframe과 통신을 하려면 어떻게 해야할까?

저는 이렇게 접근했습니다.

* iframe 내에 상태를 변경하려면 어떻게 해야할까?
* 기존에 iframe 코드 내 있는 스타일을 변경하는 함수를 실행을 시키면 되는데 그것을 React 안에서 어떻게 처리해야할까?
* iframe 안에 있는 객체에 직접 접근할 수 없구나, 그렇다면 다른 방법은 없을까?





## 해결 방법

### 1. iframe refused to connect

* X-Frame-Options가 same origin이라 로컬 환경의 도메인도 같았기 때문에 무슨 문제인지 파악이 어려운 상황이었습니다. ([MDN에서 Nginx config 수정하는 방법을 안내해줌](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options#configuring\_nginx))
* X-Frame-Options 뿐만 아니라 Content-Security-Policy라는 컨텐츠 보안 정책이 또 있었고 이것을 우리 도메인에 맞게 설정하여 외부로부터 들어올 수 있는 비정상적인 접근을 막았습니다.

&#x20;**X-Frame-Options**와 사용자 에이전트가 주어진 페이지에 대해 로드할 수 있는 리소스를 제어하는[ **Content-Security-Policy**](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)라는 Http 보안 옵션에 대해서 알게 되었습니다.



### 2. postMessage

DOM 문서를 가리키는 Window 객체에는 [**postMessage**](https://developer.mozilla.org/ko/docs/Web/API/Window/postMessage) 라는 함수가 있어서 이것을 사용해 **cross-origin 간에 안전한 통신**이 가능합니다. same-origin-policy를 안전하게 우회하는 방법이 바로 이 postMessage를 이용하는 것입니다. postMessage는 모든 브라우저에서 호환이 되기 때문에 제가 찾는 가장 좋은 해결 방안이었습니다.



<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

따라서 저는 이렇게 문제를 풀었습니다.

1. iframe을 띄우는 server에 메시지를 어떤 형식으로 받을건지 정합니다.
   1. object 형식으로도 보낼 수 있고, postMessage를 여러 군데에서 사용해야했기 때문에 특정 type으로 어떤 이벤트에 대한 메시지인지 파악하고 FE에서 설정한 값들을 함께 담아 보냅니다.
2. iframe에 useRef로 ref를 할당하여 iframe에 접근합니다.
3. `iframeRef.current.contentWindow` 가 있을 때 postMessage를 하는 함수 따로 만들어 호출합니다.
4. [**공식 문서 예시**](https://developer.mozilla.org/ko/docs/Web/API/Window/postMessage#example)에도 나와 있듯이 message를 받은 곳에서 `window.addEventListener("message")`로 어떤 message를 받는지 판별하고 특정 함수를 실행합니다.&#x20;
5. 스타일에 대한 Input 값이 변경되면 이 값을 watch하는 변수를 가져오고 useEffect 구문 안에서 스타일이 변경될 때 마다 postMessage를 실행하여 실시간으로 iframe내의 스타일이 적용되는 함수를 실행하도록 했습니다.



## 마무리

iframe은 다루어볼 기회가 많이 없었는데 이번 기회를 통해 iframe과 통신하는 방법에 대해서 이해하게 되었고 무엇보다 Http의 보안정책인 X-Frame-Options와 Content-Security-Policy에 대해서 알 수 있는 좋은 경험이었습니다.



글을 읽어주셔서 감사합니다.

좋은 하루 되세요!
