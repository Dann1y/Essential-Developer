---
description: 2023.07.16.
---

# 모노레포 업그레이드 & 관리: 효율적인 운영을 위한 실전 가이드



[https://github.com/vercel/turbo/issues/693](https://github.com/vercel/turbo/issues/693)생각의 과정

1. 서론
   1. 발표 계기
2. 기존에 사용해온 방식
3. 모노레포를 사용하면서 겪은 문제
   1. node\_modules 유령 의존성
   2. 프로젝트의 노쇠화 (버전 관리)&#x20;
      1. 의존성 버전 관리
      2. storybook 버전 업그레이드
4. 해결 방법
5. 결과

***

## 1. 서론

모노레포를 세팅하는 것에 대한 정보는 많이 있지만, 프로덕션 레벨에서 오랜 기간 사용 후 겪은 문제를 해결하는 내용은 많이 없었고, 다른 분들도 문제를 겪었을 때 해결하는 것을 돕기 위하여 정리를 하게 되었습니다.

***

## 2. 기존에 사용해온 방식

{% embed url="https://turbo.build/repo" %}

기존에 트윕 Frontend팀은 Turborepo를 사용해 모노레포를 구성했습니다.

병렬 실행, 쉬운 환경 설정, 빌드 캐싱에 강점이 있다고 생각해서 선택했고, 초기에는 만족을 하며 사용했습니다.



<figure><img src="../../../.gitbook/assets/image (5).png" alt="" width="563"><figcaption><p><a href="https://turbo.build/repo/docs/handbook/migrating-to-a-monorepo">https://turbo.build/repo/docs/handbook/migrating-to-a-monorepo</a><br></p></figcaption></figure>

`apps` directory에는 실제 트윕을 운영하기 위한 서비스를 구성했고,\
`packages` directory에는 internal package pattern을 활용하여

* apps
  * donation
  * widgets
  * ...etc
* packages
  * ui
  * shared
  * assets
  * ...etc

위와 같은 폴더 구조로 모노레포를 사용해왔습니다.

사용하면서도 이 구조에 대한 불편함은 없었고, 서비스를 개발하면서 필요한 ui나 함수를 적재적소에 사용할 수 있어서 좋았습니다.

***

## 3. 모노레포를 사용하면서 겪은 문제

그러나 초기에 모노레포를 initialize 했을 때는 괜찮았지만 시간이 지나며 여러 문제점을 발견하게 되었습니다.



### 3-1. node\_modules 유령 의존성

이미 여러 모노레포 발표들에서 node\_modules에 대한 문제점들을 많이 다뤄주셨는데요, 저희 역시 이 문제를 겪게 되었습니다.

<figure><img src="../../../.gitbook/assets/Screenshot 2023-07-16 at 9.06.14 PM.png" alt=""><figcaption></figcaption></figure>

가장 대표적인 예시로는 작업자가 실수로 A 서비스에서만 사용되어야할 패키지를 루트 혹은 다른 B서비스의  package.json에 명시해도 실제로는 root의 node\_modules를 참조하기 때문에 A서비스의 package.json 명단에는 없지만 import하고 사용이 가능했습니다. 이로 인해 의존성 관리가 점점 느슨해져 관리의 복잡도가 올라갔습니다.



또한, CI/CD 파이프라인에서 프로젝트를 빌드할 때 yarn install로 의존성 설치하는 시간이 굉장히 길었습니다. 의존성 설치 시간을 줄이는 사례를 찾아보며 node\_modules에 더 이상 의존하지 않아야겠다고 생각했습니다.



### 3-2. 프로젝트의 노쇠화 (버전 관리)

시간이 지남에 따라&#x20;

***

## 4. 해결 방법

<figure><img src="../../../.gitbook/assets/image (7).png" alt="" width="563"><figcaption><p><a href="https://github.com/vercel/turbo/issues/693">https://github.com/vercel/turbo/issues/693</a></p></figcaption></figure>



***

## 5. 결과

***

