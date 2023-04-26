---
description: 2023.04.18.
---

# What is REST?



[https://restapilinks.com/layered-system/](https://restapilinks.com/layered-system/)REST는 6가지의 제약 조건이 있다.

* Uniform Interface
* Stateless
* Cacheable
* Client-Server
* Layered System
* Code on Demand (optional)





### Uniform Interface

Client와 Server사이에 동일한 형태의 Interface로 아키텍쳐를 간소화한다.

이 안에서 총 4가지의 원칙이 있다.

<figure><img src="../../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p><a href="https://restapilinks.com/uniform-interface/">https://restapilinks.com/uniform-interface/</a></p></figcaption></figure>

* **Resource-Based**
  * 각 리소스는 URI(resource identifier)로 요청이 동일한지 확인한다.
  * 서버가 DB를 보내는 것이 아니라 HTML, JSON 형태로 DB에 기록된 표현을 보내는 것처럼 리소스들은 개념상 Client에 return되는 표현과는 분리된다.
* **Manipulation of Resources Through Representations**
  * Client가 첨부된 metadata를 포함하여 리소스의 표현을 보유할 때,
  * 권한이 있는 경우 서버에서 리소스를 수정하거나 삭제할 수 있는 충분한 정보를 가지고 있다.
* **Self-descriptive Messages**
  * 각 메시지는 어떻게 이 메시지를 처리할지에 대한 충분한 정보가 포함되어야한다.
  * Internet media type (MIME type)
* <mark style="color:red;">**H**</mark>**ypermedia **<mark style="color:red;">**a**</mark>**s **<mark style="color:red;">**t**</mark>**he **<mark style="color:red;">**E**</mark>**ngine **<mark style="color:red;">**o**</mark>**f **<mark style="color:red;">**A**</mark>**pplication **<mark style="color:red;">**S**</mark>**tate (HATEOAS)**
  * Client - body, quiery params, request header, requested URI로 상태를 전달한다.
  * Server - body, response codes & headers로 Client에 전달한다.
  * 필요한 경우 returned body (or headers)에 링크가 포함되어있는 것을 의미한다.





### Stateless

<figure><img src="../../../../.gitbook/assets/image (3).png" alt=""><figcaption><p><a href="https://restapilinks.com/stateless/">https://restapilinks.com/stateless/</a></p></figcaption></figure>

**REST - **<mark style="color:red;">**RE**</mark>**presentational **<mark style="color:red;">**S**</mark>**tate **<mark style="color:red;">**T**</mark>**ransfer**

* **Statelessness**가 바로 REST의 핵심이다.
  * Statelessness는 Server가 session state를 관리하지 않음으로써 더 좋은 확장성을 가질 수 있다.
* request를 다루기 위해 필요한 state는 request 자체에 포함되어있다.
  * URI, query-string parameters, body, or headers



* state vs resource
  * **state (application state)**
    * Server가 request를 완성하기 위해 필요한 것
    * current session, request를 위해 데이터가 필요하다.
  * **resource (resource state)**
    * resource 대표(representation)를 정의한다
    * DB에 있는 data



* 그러나 application state가 서비스의 여러 요청에 걸쳐 있지 않도록 노력해야한다.





### Cacheable

* World Wide Web에서는 Client가 response를 cache 할 수 있다.
* 따라서 responses는 그들의 cacheable하다는 것을 정의해야한다.
* 그렇지 않으면 Client가 stale 재사용하는 것을 막아야한다.
* 잘 관리된 캐싱은 client-server간의 상호작용을 삭제함으로써 성능을 향상시킬 수 있다.





### Client-server

* 동일한 형태의 interface는 Client와 Server를 분리할 수 있다.
* 따라서 각각 독립적으로 개발이 가능하다.
* Client
  * Data 저장을 신경쓸 필요가 없다.
* Server
  * User interface, User state를 신경 쓸 필요가 없다.





### Layered system



<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption><p><a href="https://restapilinks.com/layered-system/">https://restapilinks.com/layered-system/</a></p></figcaption></figure>

* Client는 직접 end server에 연결 됐는지, 중간에 하나를 거치는지는 알 수 없다.
* load-balancing을 활성화하고, Shared-caches로 시스템 확장성을 증가시킬 수 있다.
* 또한, Layer는 보안 정책도 강화할 수 있다.

###

###

### Code on demand (optional)

* Server는 실행 로직을 Client에 전송하여 임시로 Client의 기능을 확장할 수 있다.
* ex) BE- Java applets | Client-side scripts - Javascript



**NOTE**

* 오직 Code on demand만 제약 조건 중에서 optional하다.
* 만약 서비스가 다른 제약조건들을 어긴다면, RESTful하다고 할 수 없다.
