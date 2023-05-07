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



그렇다면 이러한 비효율을 어떻게하면 Client(Frontend)

{% file src="../../../.gitbook/assets/Screen Recording 2023-05-07 at 9.11.21 PM.mov" %}

에서 개선할 수 있을까요?



&#x20;
