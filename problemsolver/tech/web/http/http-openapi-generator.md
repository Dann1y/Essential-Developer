---
description: 2023.04.26.
---

# HTTP 통신, OpenAPI Generator로 추상화하기

OpenAPI Generator를 통해 HTTP 통신을 추상화한 경험에 대해서 공유하겠습니다.



## 기존에 발생한 문제점

react 생태계에서 data fetch를 하는 다양한 방법들이 있지만 몇 가지 문제점들이 있습니다.

#### 1. API server에서 어떤 값이 넘어올까?

구체적인 문서를 작성하지 않는 한 기존에는 API server에서 어떤 값이 넘어오는지 확인하려면 postman, API 문서 등 확인해야할 것이 너무 많았고 IDE에서 유추되지 않으니 휴먼 에러가 발생할 확률이 높았습니다.

최근에는 BE 엔지니어가 swagger 문서를 작성하여 API 문서의 문제를 해결하였지만 여전히 FE에서 사용할 때의 문제점은 개선되지 않았습니다.

#### 2. API에서 받아오는 data의 타입은 무엇일까?

typescript를 사용하면서 얻을 수 있는 장점은 개발 환경에서 타입을 사전에 확인하여 개발자의 의도대로 소프트웨어가 동작하도록 하는 것입니다. 그러나 data fetch 해오는 값을 개발 단계에서 타입을 확인하려면 swagger 문서를 보고 DTO를 작성하거나 아니면 휴먼 에러를 겪는 수 밖에 없었습니다.

#### 3. 너무 많은 API를 어떻게 다 유지하고 따라잡을까?

간단한 To do list 정도는 분량이 적으니 쉽게 유지보수가 가능합니다.

그러나 API 갯수가 50\~100개 이상이 된다면 이것을 어떻게 전부 유지하고 관리할 수 있을까요? 만약 API에서 넘어오는 값의 type이 변경되거나 구조가 변경되면 DTO 수정, 로직 수정 등 고려해야할 것이 많고 이는 기술 부채로 이어질 확률이 높습니다.





## 어떻게 이 문제를 해결할 수 있을까?

{% embed url="https://openapi-generator.tech/#try" %}

위 문제를 OpenAPI Generator를 활용하여 획기적으로 해결할 수 있습니다.

OpenAPI Generator를 적용하기 위해선 사전에 선행되어야할 것들이 있습니다.



1. swagger 문서를 기준으로 generate 할 것이기 때문에 swagger 문서가 작성, 유지보수 되어야합니다.
2. Typescript를 사용해야 개발 단계에서 타입 분석이 가능하고 에러를 줄일 수 있습니다.
3. FE <-> BE 간의 원활한 소통이 되어야합니다.
   1. swagger 단계에서 type에 문제가 생긴다면 generate 된 api.ts 코드에서도 문제가 발생하여 FE 작업에 영향을 주기 때문입니다.





## 어떻게 적용했을까?

선행 작업만 된다면 적용하는 것은 어렵지 않습니다.



### Installation

```sh
yarn add -D @openapitools/openapi-generator-cli
```

`@openapitools/openapi-generator-cli`

라는 패키지를 설치합니다. monorepo를 사용하기 때문에 이 또한 공용 package 폴더로 분리하여 서비스 디렉토리에서 관리하지 않도록 했습니다.



```json
"build-api": "openapi-generator-cli generate -i [Swagger Link] -g typescript-axios -o ./api
```

package.json에 `build-api` 라는 scripts를 추가하여 커맨드를 입력하면 generate가 동작하도록 했습니다.

* [**generate -i**](https://openapi-generator.tech/docs/usage#generate)
  * `[(-i | --input-spec )]`
  * 어떤 file을 포함할건지 작성
  * swagger url 뒤에 `-json` 을 붙이면 json 형식으로 보여지는데 이 링크를 \[Swagger Link]에 넣으면 됩니다.
* [**generate -g**](https://openapi-generator.tech/docs/usage#generate)
  * `[(-g | --generator-name )]`&#x20;
  * generator name을 작성합니다.
* [**generator -o**](https://openapi-generator.tech/docs/usage#generate)
  * `[(-o | --output )] [(-p | --additional-properties )...]`
  * 어떤 경로로 나올건지 output 경로를 작성합니다.



이렇게 설정하고 커맨드를 입력하면 (yarn build-api) 자동으로 generate된 API들의 Factory들이 나오는데 이 안에 API가 함수로 추상화 되어있습니다. 이를 활용하기 위해서 약간의 가공이 필요합니다.

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>generate를 하게 되면 자동으로 생성</p></figcaption></figure>

### Setting

{% code title="client.ts" %}
```typescript
import axios from "axios";
import { Configuration } from "./configuration";
import { BlogFactory } from "./api";

class APIClientClass {
  public client = axios.create({
    withCredentials: true
  });
  private readonly configuration = new Configuration();

  readonly Blog = BlogFactory(
    this.configuration,
    API_ADDRESS,
    this.client
  );
}

export const APIClient = new APIClientClass();
```
{% endcode %}

generate된 api.ts안에 Factory가 있는데 APIClientClass 안에 사용할 프로퍼티를 선언해줍니다.



### 사용할 때

이제 모든 준비가 끝났고 사용하기만 하면 됩니다. 저는 사용할 때 주로 SWR 커스텀 훅 ts file 안에서 처리했기 때문에 실제 컴포넌트 단계에서 사용할 일은 많이 없었습니다.

[useSWRMutation에서 사용한 방법은 이 글을 참고하시면 도움이 됩니다.](https://docs.essential-dev.blog/problemsolver/tech/frontend/swr/useswrmutation-mutate)



```typescript
APIClient.Blog.blogControllerCreate(arg);
```

너무 간결해졌습니다. APIClient를 가져와 Blog안에 만들어진 blogControllerCreate라는 함수를 사용하기만 하면e됩니다.

axios, fetch 코드를 작성한 긴 함수를 더 이상 컴포넌트 안에서 관리하지 않도록 서로의 관심사를 분리했습니다.



## 주의할 점

* OpenAPI generator는 Java로 작성이 되어있기 때문에 커맨드 입력 시 Java 관련 에러가 뜬다면 Java(JDK)를 설치해주어야합니다.



## 마무리

* OpenAPI generator + swr을 활용해서 명령적인 fetch과정을 선언적으로 변경하여 각각의 관심사를 분리할 수 있었고
* 더욱 강력한 타입 관리로 data fetching 할 때 타입 문제를 겪지 않고
* api에서 넘어오는 값 유추할 수 있고
* API가 추가되거나 변경되면 그냥 커맨드 한번 입력하면 끝.



Form을 관리하는 글에서 어떻게 Form의 data들의 타입을 더욱 강력하게 관리했는지 공유하겠습니다.

오늘도 글을 읽어주셔서 감사합니다.

좋은 하루 되세요!
