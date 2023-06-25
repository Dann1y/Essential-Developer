# Frontend 성능 최적화

[https://www.youtube.com/live/BEwv4to9OWY?feature=share](https://www.youtube.com/live/BEwv4to9OWY?feature=share)

## 1. 성능 최적화를 해야하는 이유

* 웹 서비스 초기 로딩 속도가 길어질수록 유저 이탈률이 함께 증가한다.
  * 이탈률이 줄어들 수록 고객 구매 횟수가 증가한다.
* UX에 가장 큰 영향을 미치는게 로딩 속도이다.
  * 이 부분은 나도 동의한다. Mancity 웹페이지는 너무 느려서 굿즈나 유니폼을 볼 때 너무 답답하다.
* Frontend만 해야하는게 아니라 타 팀과도 함께 협업하면서 최적화를 해야한다.



### 1-1. Core Web Vital

* LCP, FID, CLS
* 사용자 관점에서 정의된 지표
* **LCP**
  * Largest Contentful Paint
  * 첫 화면에서 사용자에게 보이는 가장 큰 컨텐츠가 얼마나 빠르게 보이는가?
  * 보통은 image
* **FID**
  * First Input Delay
  * 첫 화면에서 사용자가 첫 Input을 상호작용할 때 얼마나 빠르게 반응하는가?
* **CLS**
  * Cumalative Layout shift
  * 사용자의 화면에 shift가 얼마나 덜 일어나고 안정적인가?



