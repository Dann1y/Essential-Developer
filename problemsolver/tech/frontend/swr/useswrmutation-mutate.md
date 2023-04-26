---
description: 2023.04.26.
---

# useSWRMutation - Mutate 추상화하기

useSWRMutation으로 Mutate를 추상화한 경험에 대해서 공유하겠습니다.



## 기존에 발생한 문제점

Next.js에서 Data fetching을 하면서 GET 요청에 대해서는 useSWR로 추상화하여 코드 수를 줄이고 역할을 분리했다. 그러나 나머지 POST, DELETE, PUT 과 같은 mutate 요청에 대해서는 여전히 매번 긴 코드를 작성해야했다.

<figure><img src="../../../../.gitbook/assets/image (4).png" alt=""><figcaption><p>mutate의 의미</p></figcaption></figure>

OpenAPI generator가 적용되어있어(블로그 글 추가) 사용성에서 큰 문제는 없었지만 여러 mutate 요청을 하나의 컴포넌트에서 하다보면 코드가 길어질 수 밖에 없었다.

따라서 이러한 문제점을 찾기 위한 방법을 생각해보았다.



## 해결 방법

{% embed url="https://swr.vercel.app/ko/docs/mutation#useswrmutation" %}

useSWRMutation이 있다는 사실은 전부터 알고 있기는 했지만 나왔을 당시 팀에 적용하자고 의견을 냈을 때는 당장 필요하지 않아서 적용하지 않았었다. 하지만 더 좋은 방법인 것을 알았기에&#x20;
