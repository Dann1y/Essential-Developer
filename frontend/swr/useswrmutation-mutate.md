---
description: 2023.04.26.
---

# useSWRMutation - Mutate 추상화하기

useSWRMutation으로 Mutate를 추상화한 경험에 대해서 공유하겠습니다.





## 기존에 발생한 문제점

Next.js에서 Data fetching을 하면서 GET 요청에 대해서는 useSWR로 추상화하여 코드 수를 줄이고 역할을 분리했습니다. 그러나 나머지 POST, DELETE, PUT 과 같은 mutate 요청에 대해서는 여전히 매번 긴 코드를 작성해야했습니다.

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption><p>mutate의 의미</p></figcaption></figure>

[**OpenAPI generator가 적용**](https://docs.essential-dev.blog/tech/web/http/http-openapi-generator)되어있어 사용성에서 큰 문제는 없었지만 여러 mutate 요청을 하나의 컴포넌트에서 하다보면 코드가 길어질 수 밖에 없었습니다.

따라서 이러한 문제점을 찾기 위한 방법을 생각해보았습니다.



## 해결 방법

{% embed url="https://swr.vercel.app/ko/docs/mutation#useswrmutation" %}

**useSWRMutation**이 있다는 사실은 전부터 알고 있기는 했지만 나왔을 당시 팀에 적용하자고 의견을 냈을 때는 당장 필요하지 않아서 적용하지 않았었습니다. 하지만 더 좋은 방법인 것을 알았기에 먼저 적용 후 PR을 올려서 팀원 분들의 의견을 확인했습니다. [**(내가 한 행동의 근거)**](https://docs.essential-dev.blog/growthmoment/undefined/7-or-ep02#6.)&#x20;

```tsx
import useSWRMutation from 'swr/mutation'
 
// Fetcher 구현.
// 추가 인수는 두 번째 매개변수의 `arg` 속성을 통해 전달됩니다.
// 아래 예제에서 `arg`는 `'my_token'`이 됩니다.
async function updateUser(url, { arg }: { arg: string }) {
  await fetch(url, {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${arg}`
    }
  })
}
 
function Profile() {
  // useSWR + mutate 같은 API를 사용하지만, 자동으로 요청을 시작하지는 않습니다.
  const { trigger } = useSWRMutation('/api/user', updateUser, options?)
 
  return <button onClick={() => {
    // 특정 인수를 사용하여 `updateUser`를 트리거 합니다.
    trigger('my_token')
  }}>Update User</button>
}
```

SWR 문서에 있는 예시 코드입니다. 여기서 확인할 수 있는 점은

1. hook을 사용하는 구조가 기존 useSWR과 동일해서 적용하는데 어려움이 없다.
2. fetcher로 넘어오는 parameters
   1. 첫 번째 인자 = key (`/api/user`)
   2. 두 번째 인자 = option (`{arg}`)

{% code title="mutation/index.d.ts" %}
```typescript
type FetcherOptions<ExtraArg = unknown> = Readonly<{
    arg: ExtraArg;
}>;

infer Arg | null | undefined | false ? (key: Arg, options: FetcherOptions<ExtraArg>)
```
{% endcode %}

설치된 swr 모듈의 index.d.ts에 들어가보면 이를 확인할 수 있습니다. (options에 지금은 arg 하나만 있는데 나중에 업데이트 되면서 무언가 추가가 되려고 따로 빼놓은건가?) 기존 useSWR에서는 parameter가 **`arg: Arg`** 로 하나였는데 여기선 분리가 되어있어서 초반에 약간 어려움을 겪었습니다.



return되는 초기 data는 undefined이고 trigger를 실행함으로써 mutate를 할 수 있습니다. mutate를 어떻게하는지 **명령형 코드는 fetcher로 추상화**하고 코드에서는 오로지 **hook을 선언하고 trigger를 실행**하면 된다.

복잡한 fetch 과정을 단 2가지로 추상화했습니다.



## 적용한 방법

과거에 사용한 useSWR과 마찬가지로 useSWRMutation을 이용해서 custom hook을 생성하여 추상화했습니다.

예시로 블로그 글을 생성하는 POST 요청을 추상화하는 useMutateBlogCreate hook을 만들어보겠습니다.



### useMutateBlogCreate.ts

{% code lineNumbers="true" %}
```typescript
type BlogCreationRequestBody {
  name: string;
  type: string;
  id: number;
}  

import useSWRMutation from 'swr/mutation';
import { APIClient, BlogCreationRequestBody } from '@api';

const fetcher = async (
  key: string,
  { arg }: { arg: BlogCreationRequestBody },
) => {
  const {
    data: {
      data: { linkId },
    },
  } = await APIClient.Blog.blogControllerCreate(arg);

  return linkId;
};

export const useMutateBlogCreate = () => {
  return useSWRMutation('/blog/create', fetcher);
};

```
{% endcode %}

#### fetcher

* 2번: OpenAPI generator로 생성된 APIClient 객체와 POST 요청을 수행할 때 필요한 body의 타입 import
* 6번 : options 안에 arg를 가져오고 arg의 타입을 선언
* 12번 : `blogControllerCreate()` 에 arg를 넣어 POST 요청을 하고 반환하는 linkId를 가져와 return

#### customMutateHook

* 17번 : useSWRMutation hook을 return하는 useMutateCreate hook을 작성
* 18번 : key와 fetcher를 할당하여 useSWRMutation을 사용

### 사용할 때

{% code lineNumbers="true" %}
```tsx
const { trigger: addTrigger, isMutating: isAddMutating } =
    useMutateSlotMachineCreate();

const onSubmit = async () => {
    try {
      const linkId = await addTrigger({
        name: '블로그 글',
        type: 'blog',
        id: 1
      })
      
      if(linkId) {
         toast.success();
       }
    } catch (err) {
      // error handling
    }
  };
```
{% endcode %}

* `useMutateSlotMachineCreate` hook을 통해 사용할 trigger와 isMutating을 가져옵니다.
  * isMutating은 request status가 (pending)일 때 Loading UI를 표시하는 곳에 사용하여 사용자에게 웹사이트가 멈추지 않고 통신중이라는 의미를 전달할 수 있습니다.
* onSubmit이 발생할 때 trigger를 실행함으로써 POST 요청을 보내고 반환값인 `linkId`를 받아와서 이후 처리



## 마무리

useSWRMutation으로 mutate에 대해서 추상화를 하니 실제로 사용할 곳에서 **오직 Mutate만 역할을 하도록** 설계할 수 있었습니다. 기존의 레거시 코드들이 많이 남아있지만 앞으로 작성할 코드는 useSWRMutation으로 **관심사를 분리**할 수 있습니다.



글을 읽어주셔서 감사합니다.

좋은 하루 되세요!

##
