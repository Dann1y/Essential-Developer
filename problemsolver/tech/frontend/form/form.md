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



<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

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
  endcreditsName: z.string(),
  layout: z.string(),
  textAlign: z.string(),
  creditSpeed: z.number(),
});

```
{% endcode %}

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
            render={({ field }) => (
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



1번: POST 요청을 보낼 때의 body 값 = form에서 보내야할 data이기 때문에 [**OpenAPI Generator를 사용**](https://docs.essential-dev.blog/tech/web/http/http-openapi-generator)하여 나온 interface를 FormValueType으로 선언합니다.

7번: react-hook-form의 useForm으로 formMethods를 선언합니다. 앞서 선언했던 FormValueType을 useForm에 generic으로 type을 선언해줍니다.

앞으로 useForm으로부터 나온 control로 input들을 관리하게 될텐데 name을 입력하고 value를 관리할 때 사전에 선언된 type이 있기 때문에 **code를 작성하는 단계에서 나오는 type 에러를 사전에 거를 수 있게 됩니다.**

26번: 뜬금없는 useEffect 구문은 컴포넌트가 마운트되는 시점에 **useForm의 defaultValues로는 기본 값을 설정할 수 없어** 마운트된 이후에 받아온 data로 각 input에 value를 설정하는 것입니다.

이러한 useEffect 구문을 작성하고 싶지 않다면 SSR을 통해 server의 request time에 form의 input에 들어갈 data를 pre-fetch해와서 useForm의 defaultValues로 설정하는 방법이 있습니다.





