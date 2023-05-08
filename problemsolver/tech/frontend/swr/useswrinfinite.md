---
description: 2022.04.26.
---

# useSWRInfinite - Intersection Observer와 함께 무한 스크롤 구현하기

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

**Intersection Observer**로 taget하는 요소와 viewport 사이의 상호작용 변화를 동기적으로 관찰하고, 교차될 때 실행될 callback function을 등록할 수 있습니다. 단어의 의미 그대로 교차(ex. scroll) 관찰하는 것입니다.

따라서 website는 target 요소의 교차를 지켜보기 위해 메인 스레드를 사용할 필요가 없어지고 브라우저는 intersection 관리를 최적화할 수 있습니다.



## 적용 방법

로직을 설명할 수 있는 핵심적인 몇 가지의 예시 코드를 첨부해보겠습니다.

{% code title="IntersectionObserver" lineNumbers="true" %}
```tsx
const [ref, setRef] = useState<HTMLElement | null | undefined>(null);

  const onIntersect: IntersectionObserverCallback = useCallback(
    ([entry]) => {
      if (entry.isIntersecting) {
        setSize((prev) => prev + 1);
      }
    },
    [setSize],
  );

  useEffect(() => {
    if (!ref) return;
    const observer = new IntersectionObserver(onIntersect, {
      threshold: 1,
    });
    
    observer.observe(ref);
    return () => observer && observer.disconnect();
  }, [ref, onIntersect]);

  return <div ref={setRef}></div>;
```
{% endcode %}

google + chatGPT로 쉽게 찾을 수 있는 React 코드입니다.

ref가 선언되어있는 element의 모양이 어떻든 그 element를 전부 감싸는 최소 크기의 직사각형을 기준으로 Intersection을 판별합니다.

entries의 첫 번째 요소를 entry로 선언하고 `entry.isIntersecting` 일 때 useSWRInfinite의 setSize로 size를 1 증가 시킴으로써 다음 data를 받아옵니다.

비동기적으로 동작하는 useEffect 구문 안에서 observer 인스턴스를 선언하고 `observe(ref)`로 관찰할 target 요소를 지정합니다.



{% code title="useGetInfiniteBlogPost" lineNumbers="true" %}
```tsx
// getKey 함수 정의
const getKey = (pageIndex, previousPageData) => {
  const newPageIndex = pageIndex + 1;
  if (previousPageData && !previousPageData.items.length) {
    return null;
  }
  return `/api/blog?page=${newPageIndex}`;
};

// useSWRInfinite hook 사용
const { data, error, size, setSize, isValidating } = useSWRInfinite(
  getKey,
  (url) => fetch(url).then((res) => res.json())
);

export const useGetInfiniteBlogPost = () => {
  return useSWRInfinite(getKey, fetcher);
};
```
{% endcode %}

{% code title="사용" lineNumbers="true" %}
```tsx
 const { data, mutate, size, setSize } = useGetInfiniteBlogPost();
```
{% endcode %}

chatGPT + useSWRInfinite 공식문서에 있는 코드를 참고했습니다.

useSWR과 useSWRInfinite 사용은 대부분 비슷하지만 swr key를 선언하는 방식이 다릅니다. 인덱스와 이전 페이지 데이터를 받고 페이지의 키를 반환하는 함수로 `getKey`를 선언하고 할당합니다.



### 동작 방식

동작 방식은 다음과 같습니다.

1. useSWRInfinite로 data를 GET하는 코드를 선언적으로 작성합니다.
2. Intersection Observer 함수를 만들고, data 리스트가 있을 때 ref를 추가한 target 요소를 data 리스트 최하단에 추가하여 observe합니다.
3. 유저가 data 리스트를 최하단까지 스크롤하여 `entry.isIntersecting` 이면 useSWRInfinite의 setSize를 사용해 불러올 page만큼 size를 증가시켜 data 배열에 다음 data를 추가합니다.
4. 계속 반복하다가 data를 더 이상 불러올 수 없으면 null을 반환하여 fetch를 종료합니다.



## 마무리

* useSWRInfinite로 무한 스크롤에 대한 pagination을 쉽게 할 수 있고
* Intersection Observer로 pagination의 기준을 잡을 수 있습니다.



글을 읽어주셔서 감사합니다.

