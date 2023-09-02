---
description: 2023.09.02.
---

# 왜 Next는 Babel에서 SWC로 Compiler를 변경했는가?

## 1. 문제 배경

### 느린 빌드 속도

* Javascript를 transpile하고 빌드하는데 시간이 오래 걸립니다.
* 빌드가 느려진 이유
  * 빌드 툴(V8 엔진을 통해 실행)이 CPU를 많이 활용합니다.
  * Javascript의 event loop은 CPU 위주의 작업에 적합하지 않은 모델입니다.
    * 싱글 스레드이기 때문에 데이터 의존성이 없는 병렬처리가 가능한 작업도 동기적으로 실행해서 그렇습니다.

### Babel의 한계

* Javascript 기반이고
* CPU 위주의 작업인데 1개의 CPU 코어만 사용하며
* Parsing, Code Generation은 더욱 오래걸립니다.
* Javascript V8 엔진의 동작에 대한 내용은
  * [javascript-language-engine-timing.md](../../javascript/javascript-language-engine-timing.md "mention")

***

## 2. 해결 방법

### SWC (Speedy Web Compiler)

* 이름에서 유추할 수 있듯이 아주 빠른 웹 Compiler입니다.
* Rust로 작성된 이유는
  * 병렬처리로 인한 성능 증가가 핵심 + 여러 개발 편의성



### 기능

* Transform
  * 비동기
  * 멀티 스레드
  * Babel을 대체
* Minify
  * 비동기
  * 멀티 스레드
  * terser(parser, mangler, compressor)를 대체
* Compile, Bundling, Minification 등 다양한 작업을 수행합니다.



***

## 3. 간략한 나의 경험

이전에는 babelrc 같은 config 파일을 다뤘어야했는데 (난 이걸 진짜 싫어했다.) Next 환경에서 주로 사용하는 지금은 그냥 `next.config.js` 파일에서 어떻게 수정해야할지 값 유추도 가능하기 때문에 Babel에 비해 설정하기가 편해졌습니다. `ex) styledComponents`



빌드 속도는.. 솔직히 최근 SWC만 사용하고 익숙해져서 그런지 잘 못 느끼는 중입니다. 다시 Babel을 적용해야 역체감을 느낄 수 있을 것 같은데 일단 빠른건 확실한 것 같습니다.



***

## 마무리

* 기존 Javascript 기반의 Babel이 수행하던 느린 Parsing, Transpiling을
* 멀티 스레드로 인한 병렬처리 + 강력한 성능을 자랑하는 Rust 기반의 SWC가 등장하여&#x20;
* 이를 대체했더니
* 성능이 너무 좋아져서 안 쓸 이유가 없기 때문에 바꿨다.

***

## 참고 & 출처

* [https://youtu.be/4RJxyGJQe4o?si=2DvLmE5i8pPm1EN0](https://youtu.be/4RJxyGJQe4o?si=2DvLmE5i8pPm1EN0)
* [https://nextjs.org/docs/architecture/nextjs-compiler](https://nextjs.org/docs/architecture/nextjs-compiler)
