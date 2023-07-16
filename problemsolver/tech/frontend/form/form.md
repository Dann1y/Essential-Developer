---
description: 2023.05.03.
---

# Form을 관리하고 최적화한 방법

[**지난 글**](https://docs.essential-dev.blog/tech/frontend/form/web-form)에서 Web에서 Form을 제작하는 것을 총 3단계로 나누어보았습니다.

**"Form을 구축하고, 검증하고, 데이터를 보낸다."**

위 3단계로 Form을 제작하며 발생했던 문제를 해결한 경험을 경험에 기반하여 작성하고 필요한 부분에는 코드를 추가하겠습니다. 제가 작성하는 코드는 전부 **예시**이며 독자분들의 **production 환경에 적용하려면 추가적인 custom이 필요**합니다.



## 1. Structure web form

### 반복되는 코드

사용자에게서 어떤 정보를 받아야할 때 Form안의 Input들로부터 정보를 입력받게 됩니다. 여러 종류의 Input들은 사용자가 어떤 값을 입력해야하는지 유추하기 쉽게 하고 입력하는 편의성을 높이지만 코드에서의 반복 작업이 늘어날 수 있습니다.

Input이 많아지는만큼 Form의 코드 수도 증가합니다. 증가한 코드 수는 안 그래도 복잡한 Input을 더욱 파악하기 어렵게 만듭니다. 따라서 이를 해결하기 위해서 **중복되는 역할을 추상화**한다는 개념으로 접근해보았습니다.

<figure><img src="../../../../.gitbook/assets/image (5) (2).png" alt=""><figcaption><p>google form example</p></figcaption></figure>

이 HTML 위젯은 label과 input, 크게 2가지로 볼 수 있습니다.

label을 부모로 두고 input을 children으로 받아서 사용할 때 편의성을 늘릴 것입니다. 두 가지가 분리 되어있다면 매번 label과 input을 각각 작성하고 이를 감싸는 엘리먼트를 하나 더 추가해야하는 번거로움이 있기 때문입니다.

따라서 각각 고려해야할 것은 다음과 같습니다.

* label
  * input을 감싸고 input의 name을 label의 for로 사용
  * 위젯의 상태 (error인지 success인지)
  * help message
* input
  * 각 역할(select, textarea, input type="text" ...etc)에 맞는 input을 컴포넌트로 추상화

<pre class="language-tsx" data-title="InputWidget으로 추상화" data-line-numbers><code class="lang-tsx">export const InputWidget = ({ options ,...props }: InputWidgetProps) => {
  const { name, value, onChange, onBlur, status } = props;
  
  return (
    &#x3C;FormLabel {...props}>
<strong>      &#x3C;Input
</strong>        name={name}
        onChange={onChange}
        value={value}
        status={status}
        {...options}
      />
    &#x3C;/FormLabel>
  );
<strong>};
</strong></code></pre>

1번: options prop은 input에 필요한 prop을 받고 나머지 props는 FormLabel에 사용합니다.

2번: Input에 사용할 prop만 props에서 가져와 Input에 내려줍니다.



{% code title="InputWidget 사용할 때" lineNumbers="true" %}
```tsx
<InputWidget
  name="cash"
  required
  label="후원할 캐시"
  option={{
    type: 'number',
    min: 1
  }}
  helpMessage="후원할 캐시를 입력해주세요"
  onChange={onChange}
/>
```
{% endcode %}

label + input = widget 코드가 정말 간결해졌습니다.

HTML 위젯을 구성할 때 label, input을 작성하고 이것을 매번 반복하는 문제를 겪을뻔 했지만 중복되는 작업을 추상화함으로써 이 문제를 해결했습니다. 앞으로 InputWidget을 사용할 때 오직 필요한 값만 넣으면 동일한 UI를 기대할 수 있습니다.



## 2. Value Management & Validation

그렇다면 앞서 작성한 InputWidget에 대한 validation은 어떻게 해야할까요?

[**client-side form validation**](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form\_validation) 에 나온 것처럼 HTML built-in validation을 사용할 수도 있고 Javascript로 custom validation을 만들어서 사용할 수도 있습니다.

하지만 이 때 <mark style="color:red;">**문제가 발생**</mark>합니다. 간결했던 InputWidget 코드가 validation이 추가되면서 관심사가 1개 더 늘어나게 되고 **validation의 조건이 복잡해질 수록 코드도 더 복잡**해지며 이런 Input이 여러가지가 생긴다면 굉장히 관리하기 힘들 것입니다.



뿐만 아니라, 사용자의 입력을 받을 때마다 validation 조건과 비교하려면 controlled component 형식으로 관리를 해야하는데 이 때 **input의 value들은 어떻게 관리**를 해야할까요? input이 늘어날 때마다 useState 구문을 추가해야하는걸까요?



이러한 문제점을 해결하기 위해 빠르게 적용이 가능한 라이브러리를 사용하는 것을 고려했고 선택한 라이브러리는 **react-hook-form +** **zod**입니다.

{% embed url="https://react-hook-form.com/" %}
react-hook-form
{% endembed %}

{% embed url="https://zod.dev/" %}
zod validation
{% endembed %}



Input의 value와 관련된 관리(onChange, value, name, ref ..etc)는 react-hook-form

모든 validation은 zod로 위임하여 관심사를 분리하겠습니다.



블로그 설정을 변경하는 코드를 예시로 작성해보겠습니다.

{% code title="validationSchema.ts" lineNumbers="true" %}
```typescript
import { SetEndcreditsWidgetRequestBody } from '@twip-fe/shared'
import { ZodObject, z } from 'zod';

type SetBlogConfigKey = keyof SetBlogConfigRequestBody;

export const validationSchema: ZodObject<
  Record<SetBlogConfigKey, z.ZodTypeAny>
> = z.object({
  title: z.string(),
  description: z.string(),
  id: z.number()
});

```
{% endcode %}



4번: POST 요청을 보낼 때의 body 값 = form에서 보내야할 data이기 때문에 [**OpenAPI Generator를 사용**](https://docs.essential-dev.blog/tech/web/http/http-openapi-generator)하여 나온 interface의 key 값만 가져와 validationSchema를 선언할 때 발생하는 오타, 빠진 부분을 사전에 파악하여 휴먼에러를 줄일 수 있습니다.

zod 사용법은 불친절한 공식문서를 참고하시게 될텐데 가장 많이 쓰이는 3가지만 공유드리겠습니다.

* string, number처럼 기본적인 type을 validation합니다.
* refine - min, max처럼 간단하지 않은 조건에 대해서 value로 커스텀이 필요하다면 사용합니다.
* superRefine - 여러 input에 의존하는 validation과 함께 context에 접근하여 각 상황마다 `ctx.addIssue` 로 에러를 지정할 때 사용합니다.



{% code title="BlogConfigSetupForm" lineNumbers="true" %}
```tsx
export type BlogConfigFormValueType = SetBlogConfigRequestBody;

export const BlogConfigSetupForm = () => {
  const { data, isLoading, mutate } = useGetBlogConfig();
  const { trigger, isMutating } = useMutateBlogConfig();
  
  const formMethods = useForm<BlogFormValueType>({
    resolver: zodResolver(validationSchema),
    mode: 'all',
  });
  
  const {
    reset,
    control,
    handleSubmit,
  } = formMethods;
  
  const onSubmit = async (values: BlogConfigFormValueType) => {
    try {
      // 비동기 fetching codes
      const result = await trigger(values);
      
      if(result.status === 200) {
        //success handling
      }
    } catch (e) {
      // error handling
    }
  };
  
 useEffect(() => {
    if (data) {
      reset(data);
    }
  }, [data, reset]);

  return (
    <FormProvider {...formMethods}>
      <form ref={formRef} onSubmit={handleSubmit(onSubmit)}>
          <BlogConfigDefaultSection control={control} />
          
          //예시에서 보기 쉽도록 Controller를 작성했습니다.
          <Controller
            name="title"
            control={control}
            render={({ field }) => ( // field: { name, onBlur, onChange, ref, value }
              <InputFormItem
                {...field}
                label="제목"
                option={{
                  placeholder:
                    '1~30글자 사이로 작성해주세요.',
                }}
              />
            )}
          />
          
          <CTAButton />
      </form>
    </FormProvider>
  )
}
```
{% endcode %}

react-hook-form 공식문서의 코드를 응용하여 예시 코드를 작성해보았습니다.

* **1번**: POST 요청을 보낼 때의 body 값 = form에서 보내야할 data이기 때문에 [**OpenAPI Generator를 사용**](https://docs.essential-dev.blog/tech/web/http/http-openapi-generator)하여 나온 interface를 FormValueType으로 선언합니다.\

* **7번**: react-hook-form의 useForm으로 formMethods를 선언합니다. 앞서 선언했던 FormValueType을 useForm에 generic으로 type을 선언해줍니다.\
  \
  앞으로 useForm으로부터 나온 control로 input들을 관리하게 될텐데 name을 입력하고 value를 관리할 때 사전에 선언된 type이 있기 때문에 **code를 작성하는 단계에서 나오는 type 에러를 사전에 거를 수 있게 됩니다.**\

* **26번**: 뜬금없는 useEffect 구문은 컴포넌트가 마운트되는 시점에 **useForm의 defaultValues로는 기본 값을 설정할 수 없어** 마운트된 이후에 받아온 data로 각 input에 value를 설정하는 것입니다.\
  \
  이러한 useEffect 구문을 작성하고 싶지 않다면 SSR을 통해 server의 request time에 form의 input에 들어갈 data를 pre-fetch해와서 useForm의 defaultValues로 설정하는 방법이 있습니다.\

*   **38번**: react-hook-form의 Controller를 활용하여 Input의 value를 관리합니다.\
    \
    가장 좋은 점은 custom이 필요하지 않은 경우 `{...fields}`로 기본 값을 넘겨만 주어도 onChange, value 관리가 되기 때문에 사용할 때 편했습니다.\
    \


    <figure><img src="../../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

    [FormProvider](https://react-hook-form.com/api/usecontroller/controller/)사용하는 경우 control은 optional이지만 앞서 말씀드린 Controller의 name의 type을 관리하기 위하여 저는 control을 사용하였습니다.\
    \
    예를 들어 controller의 name이 `title` 인데 만약 `titl` 로 오타가 난 경우 `titl is not assignable to type (FormValueType)` 와 같이 에러를 마주하여 휴먼 에러를 줄일 수 있습니다.



## 3. Submit

위 코드처럼 별도의 onSubmit 비동기 함수를 작성하여 form의 submit을 handling합니다.

이 때 POST하는 HTTP 요청에 대해서는 [**useSWRMutation을 통해 과정을 추상화**](https://docs.essential-dev.blog/tech/frontend/swr/useswrmutation-mutate)하여 Mutation에만 관심사를 갖도록 했습니다.



경험했던 문제는 **DOM 구조를 기준으로 form 내에 submit button이 없는 경우**에 버튼을 눌러도 form의 submit이 동작을 하지 않는다는 것인데, 이 때 **form에 ref를 할당**하고 form 밖의 버튼을 클릭하면 `submit` 이라는 event를 dispatch 시켜서 onSubmit event를 실행하도록한 경험이 있습니다.



## 마무리

Next + typescript 환경에서

* 컴포넌트 추상화를 통해 **중복을 제거하여 사용의 편의성을 증가**시켰고
* react-hook-form + zod를 통해 **input value management, validation, type**을 효율적으로 관리하였으며
* **OpenAPI Generator와 useSWR hook**으로 data를 GET해서 form의 기본 값을 관리하고 Form을 Submit(POST or PUT)하는 과정을 추상화하여 **코드 복잡도를 줄이고 유지보수**하기 쉽도록 하였습니다.

Form을 구축하고 관리하는 과정에 대해 저의 경험을 바탕으로 겪었던 문제들, 해결방법까지 서술해보았습니다.



글을 읽어주셔서 감사합니다!

좋은 하루 되세요.\
