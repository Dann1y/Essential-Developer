---
description: 2023.07.16.
---

# 궁극의 Monorepo 업그레이드

생각의 과정

1. 서론
   1. 발표 계기
2. 기존에 사용해온 방식
3. 모노레포를 사용하면서 겪은 문제
   1. node\_modules 유령 의존성
   2. 번들 사이즈
   3. 프로젝트의 노쇠화 (버전 관리)&#x20;
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



### 3-2. 버전 관리



### 3-3. 프로젝트의 노쇠화 (버전 관리)

***

## 4. 해결 방법



***

## 5. 결과

***

