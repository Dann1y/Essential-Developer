---
description: 2022.04.26.
---

# useSWRInfinite - Intersection Observer와 무한 스크롤 구현하기

useSWRInfinite + Intersection Observer로 무한 스크롤을 구현한 경험을 서술해보겠습니다.



## 무한 스크롤을 적용해야하는 이유는 무엇인가(문제점)?

{% embed url="https://www.pinterest.co.kr/bur_iq/erling-braut-haaland/" %}
Haaland
{% endembed %}

Pinterest는 많은 양의 data(사진, 태그, 계정 정보 등)을 스크롤을 하며 보여줘야합니다.\
홀란드를 Pinterest에서 검색한다고 했을 때 홀란드에 대한 Pinterest 게시글이 극단적으로 잡았을 때 1만개가 있다고 하면 1만개의 data를 어떻게 받아와야할까요?

만약 10만명의 유저가 홀란드를 검색한다면 1만 게시글을 10만명에게 한번에 보내준다면 동시에 다수의 요청으로 인한Server의 부하도 생길 것이고 직접 사용자가 조회하지 않는 data도 보내지기 때문에 비효율적일 것입니다.



그렇다면 이러한 비효율을 어떻게하면 Client(Frontend)에서 개선할 수 있을까요?



## 문제 해결 방법

{% embed url="https://swr.vercel.app/ko/docs/pagination#useswrinfinite" %}

Table(데이터 표)형식에서는 api를 요청할 때 query에 page를 함께 보내서 어떤 page를 fetch할건지 state로 조절이 가능하겠지만, 위와 같은 scroll 형식의 data에서는 이를 적용하기에는 부적합합니다.



따라서 문제 해결을 위한 접근 방식은 다음과 같습니다.

1. 초기에 10만개의 data를 모두 불러오지 않고 현재 유저에게 보여줄 꼭 필요한 n개의 data만 가져온다.
2. 유저가 초기에 있는 data를 모두 봤다면 다음에 이어서 data를 가져온다.
   1. 그렇다면 모두 봤다는 것은 어떻게 판별할 것인가?



꼭 필요한 n개의 data fetch => useSWRInfinite

모두 봤다는 것에 대한 판별 => Intesection Observer

로 해결하였습니다.&#x20;



#### useSWRInfinite

{% embed url="https://swr.vercel.app/ko/docs/pagination#useswrinfinite" %}

**`특정 지점에 도달하면 추가로 n개의 data를 fetch하여 유저에게 보여준다`**의 개념도 결국 Table에서 버튼을 누르지 않았다뿐이지 동작 방식으로는 pagination이라고 할 수 있습니다.

이 때 **useSWRInfinite**를 사용하여 무한 스크롤에 대한 pagination을 쉽게 구현할 수 있습니다.



useSWRInfinite의 return값은 useSWR + 가져올 page의 수인 `size`와 page의 수를 정하는 `setSize` 가 추가됩니다.



#### Intersection Observer

{% embed url="https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API" %}







## 적용 방법





&#x20;
