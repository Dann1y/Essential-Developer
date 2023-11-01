---
description: 2022.10.12.
---

# useSWRImmutable - global state 관리하기

<figure><img src="https://blog.kakaocdn.net/dn/5HuOr/btr4ihqNHkR/HNkjAMKfIPO06GbRQ6vBy0/img.png" alt=""><figcaption></figcaption></figure>

이번에는 useSWR로 global state를 관리했던 경험을 팀에 발표한 것을 바탕으로 이야기해 보겠습니다.

이 블로그에 작성한 코드는 모두 swr 공식문서를 바탕으로 작성된 예시코드이며 production 단계에서 사용하려면 추가적인 코드 작성 및 수정이 필요합니다.



#### useSWR을 사용하면서 global state를 관리해야 하는 상황

로그인 Dialog를 open하는 액션을 여러 컴포넌트에서 처리해야 할 때가 생겼습니다.

한 곳에서만 Dialog를 띄운다면 그 컴포넌트의 안에서 prop으로 처리해야 하지만 외부에 있을 경우 global state를 사용해야 했습니다.

&#x20;

&#x20;

<figure><img src="https://blog.kakaocdn.net/dn/HS4AI/btr4hmzyOPh/sbbKh5bPI9xxTFpqIdiHg0/img.png" alt=""><figcaption></figcaption></figure>

따라서 적용된 초기 코드는 다음과 같습니다.

1\. fallbackData와 revalidate에 대한 옵션을 모두 false로 처리했습니다.

2\. return 값에서 isLoginDialogOpen: data  , setLoginDialogOpen: mutate로 보내주었습니다.

&#x20;

&#x20;

<figure><img src="https://blog.kakaocdn.net/dn/dNG1fe/btr4iUXmqPX/hoTQGbw2sEQpqcWIjFj6xk/img.png" alt=""><figcaption></figcaption></figure>

그러나 고려해야 할 점이 생겼습니다.

다른 Dialog에서도 global state를 사용해야 할 때 초기에 작성된 위 코드는 재사용을 할 수 없습니다.

&#x20;

따라서 하나의 hook에서 재사용이 가능하도록 코드를 개선해 보았습니다.

&#x20;

<figure><img src="https://blog.kakaocdn.net/dn/4YIDE/btr4wLqTIBk/fxBBlTJUAJVYO3dULTCc90/img.png" alt=""><figcaption></figcaption></figure>

1\. 전에는 key 값이 고정이었다면 key를 받아올 수 있도록 변경하였습니다.

2\. 받아오는 key의 오타로 인한 동작 오류 방지를 위해 type으로 사전에 잘못된 값을 넘겨받는 오류를 방지하였습니다.

3\. 초기의 값을 defaultOpen으로 받아와서 useSWR의 fallbackData로 넘겨줍니다.

4\. @param으로 각 파라미터에 대한 내용을 함수 사용 시에 설명하여 각자의 IDE 환경에서도 쉽게 이해가 가능하도록 설계했습니다.

&#x20;

***

#### 여기서 좀 더 코드 길이를 줄여보자면?

<figure><img src="https://blog.kakaocdn.net/dn/Z0vKS/btr4vIBiTwY/rmvPEcwtwmleiGrENNPnN1/img.png" alt=""><figcaption></figcaption></figure>

swr에는 useSWRImmutable이 있습니다.

앞서 확인했던 revalidationeOnFocus, revalidateIfStale, revalidateOnReconnect 3가지의 값이 false가 기본으로 적용되어 있는 hook입니다.

&#x20;

<figure><img src="https://blog.kakaocdn.net/dn/5gyUp/btr4yjm64PD/SW8aR1R6rNLM5nkk0NPlL1/img.png" alt=""><figcaption><p>공식문서에서 쉽게 찾을 수 있다</p></figcaption></figure>

따라서 useSWRImmutable을 적용하면

&#x20;

<figure><img src="https://blog.kakaocdn.net/dn/NDbjv/btr4uJ8BnlH/5bzCFjaabv1NrmxunGlhM0/img.png" alt=""><figcaption></figcaption></figure>

위와 같이 useSWRImmutable을 사용하여 focus, stale, reconnect에 대한 revalidate를 false 처리한 채로 더 짧게 작성할 수 있었습니다.

&#x20;

#### 사용 예시

<figure><img src="https://blog.kakaocdn.net/dn/zqurn/btr4v8NoVbH/GvJQnGVyLnTSZoe1nRCh7K/img.png" alt=""><figcaption></figcaption></figure>

따라서 실제 사용할 때는 커스텀 hook을 사용하는 것과 같이 선언을 하고 각 key값에 대한 global state 관리를 할 수 있었습니다.

&#x20;

***

#### 사용할 때 좋았던 점

1\. 가벼운 로직이지만 global state를 관리해야 할 때 과거에는 global state 관리하는 library를 사용하였지만 이러한 단점을 hook 하나로 해결할 수 있어서 많은 코드를 작성할 시간을 아낄 수 있었습니다.

2\. SSR로 pre-fetch 한 값을 \<SWRConfig>의 fallback data로 key 값에 맞게 넘겨줄 수 있었습니다.

3\. swr의 다양한 기능과 설정들을 경험해 볼 수 있어서 이해하는데 도움이 많이 되었습니다. (점점 Vercel 생태계의 매력에 더 깊게 빠지는 느낌)

&#x20;

***

#### 마치며

useSWR로 global state 관리했던 경험에 대해서 이야기해 보았습니다.

읽어주셔서 감사합니다.

좋은 하루 보내세요!
