---
description: 2023.07.20.
---

# ES Modules

{% embed url="https://www.youtube.com/watch?v=mee1QbvaO10&ab_channel=FEConfKorea" %}

{% hint style="info" %}
Monorepo에서 packages의 tree-shaking을 다루면서 es module에 대해 실제 사례에 기반한 추가지식을 얻고 싶어 이 영상을 보고 정리한 글입니다.
{% endhint %}

## CommonJS

* 과거에 script태그로 외부 라이브러리를 가져올 때는 변수명이 겹치거나 수 많은 라이브러리를 가져오는데 코드가 길어지는 어려움이 있었다.
* CommonJS를 통해 작은 모듈 단위로 가져와 개발을 할 수 있게 되었다.
* 문제점
  * CommonJS가 지원되지 않는 runtime에는 사용할 수 없다. (Node.js 같은 환경에서 사용 가능)
  * 정적 분석이 어려움
    * 삼항 연산자
  * 비동기 모듈 정의 불가능
    * DB connect
  * require 함수 재정의



## TSC Transpiling

* TS나 Babel을 사용하면 우리 코드는 CommonJS로 변환이 된다.
  * import -> require('')



## ES Modules (ECMAScript Modules)

* import { hey } from './hey.js'
* 정적 분석 용이
* 쉬운 비동기 모듈 & 비동기로 동작
  * top level await
* ESM = "언어 표준"



## 에러 메시지 원인

* 동기 <-> 함수 호출
  * 비동기에서 동기 호출 O
  * 동기에서 비동기 호출 (어려움)
* ESM -> CommonJS 사용 가능
* CommonJS ->ESM 사용하기 어려움
  * `require() of ES Module`
* ESM 도입으로 해결



* package.json
  * type: module -> ESM 방식으로 동작
* .cjs -> 항상 require()
* .mjs -> 항상 import

## ESM으로 옮기기 어려운 두 가지 이유

* 가짜 ESM
* 성숙하지 않은 생태계



* 가짜 import vs 진짜 import
  * `import { Component } from './Mycomponent'`
    * 가짜 (ESM 표준 상으로는 틀림)
  * `import { Component } from './Mycomponent.js'`
    * 파일의 확장자가 있음
* Node.js의 require은 파일을 순회하면서 우리가 요청하는 파일을 찾아내려고 한다.
  * 순회는 비용이 비싸고 웹팩 번들링 속도가 느려짐
  * import하는 파일은 확장자가 명시 되어있어야한다.



* 성숙하지 않은 생태계
  * Typescript 생태계
  * `import { add } from './add.ts'`&#x20;
    * 틀린 사용법
  * typescript도 js로 import를 해야한다. (add.js)
  * cjs, cts
  * mjs, mts
* subpath import 문제
  * await import('next/app') X
  * await import('next/app.js') O



* Jest, TS Node, Yarn
  * require를 바꿈, CommonJS에 의존
  * Yarn PnP
  * Loaders API



## ESM Migration

* type: "module"
* import에 확장자 추가
* require문을 import export문으로 변경
* \_\_\_dirname
*
