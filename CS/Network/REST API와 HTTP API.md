REST API와 HTTP API

> 출처  : https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html

## REST API

REpresentational State Transfer이다.

- Representatinal : 표현
- State : 상태
- Transfer : 전송

즉, 자원을 이름(표현)으로 구분하여 해당 자원의 상태(정보)를 주고 받는 모든 것을 의미한다.

구체적으로는, HTTP URI를 통해 자원(Resource)을 명시하고, HTTP Method를 통해 자원에 대한 상태를 주고 받을 수 있는 것이다.

<br/>

## REST의 특징

1. Server-Client(서버-클라이언트 구조)
   - 자원이 있는 쪽이 Server, 자원을 요청하는 쪽이 Client가 된다.
   - 서로 간 의존성이 줄어든다.
2. Stateless(무상태)
   - HTTP 프로토콜은 stateless Protocol이므로, REST 역시 무상태성을 갖는다.
   - Client의 context를 Server에 저장하지 않는다.
     - 즉, 세션과 쿠키와 같은 context 정보를 신경쓰지 않아도 되므로 구현이 단순해진다.
   - Server는 각각의 요청을 완전히 별개의 것으로 인식하고 처리한다.
     - 각 API 서버는 Client의 요청만을 단순 처리한다.
     - 이전 요청과 다음 요청이 처리에 연관되면 안된다.
     - Server의 처리 방식에 일관성을 부여하고 부담이 줄어들며, 서비스의 자유도가 높아진다.
3. Cacheable(캐시 처리 가능)
   - 웹 표준 HTTP 프로토콜을 그대로 사용하므로 웹에서 사용하는 기존의 인프라를 그대로 활용할 수 있다.
     - 즉, HTTP가 가진 가장 강력한 특징 중 하나인 캐싱 기능을 적용할 수 있다.
     - HTTP 프로토콜 표준에서 사용하는 Last-Modified, ETag를 이용.
   - 대량의 요청을 효율적으로 처리하기 위해 캐시가 요구된다.
   - 캐시 사용을 통해 응답시간이 빨라지고 REST Server 트랜잭션이 발생하지 않기 때문에 전체 응답시간, 성능, 서버의 자원 이용률을 향상시킬 수 있다.
4. Layered System(계층화)
   - Client는 REST API Server만 호출한다.
   - REST Server는 다중 계층으로 구성될 수 있다.
     - API Server는 순수 비즈니스 로직을 수행하고 그 앞단에 보안, 로드밸런싱, 암호화, 사용자 인증 등을 추가하여 구조상의 유연성을 줄 수 있다.
     - 또한 로드 밸런싱, 공유 캐시 등을 통해 확장성과 보안성을 향상시킬 수 있다.
   - PROXY, 게이트웨이 같은 네트워크 기반의 중간 매체를 사용할 수 있다.
5. Code-On-Demand(Optional)
   - Server로 부터 스크립트를 받아 Client에서 실행한다.
   - 반드시 충족할 필요는 없다.
6. Uniform Interface(인터페이스 일관성)
   - URI로 지정한 Resource에 대한 조작을 통일되고 한정적인 인터페이스로 수행한다.
   - HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능하다.
     - 특정 언어나 기술에 종속되지 않는다.

<br/>

> 단, HTTP API와 REST API의 정확한 차이점으로는 REST API는 아래 4가지 가이드 원칙을 지켜야 한다.
>
> 1. 자원의 식별
> 2. 메시지를 통한 리소스 조작
> 3. 자기 서술적 메시지
> 4. 애플리케이션의 상태에 대한 엔진으로서 하이퍼미디어(HATEOAS)

<br/>

<br/>

## Uniform Interface

Uniform Interface는 URI로 지정된 리소스에 대한 조작을 통일하고 한정된 인터페이스로 수행하는 아키텍쳐 스타일이다. Uniform Interface에 제약조건이 있다. 4가지. 이는 방금 위의 인용문과 동일하다.

1. Resource-Based
   - 자원의 식별
2. Manipulation Of Resources Through Representations
   - 메시지를 통해 리소스 조작
3. self-descriptive messages
   - 자기 서술적 메시지. -> 메시지는 스스로 설명해야한다.
4. hypermedia as the engine of application state(HATEOAS)
   - 클라이언트와 서버가 서로 소통하지 않고도 메시지를 해석함으로써 독립적인 진화를 할 수 있음.(서버에서 메시지가 바뀌어도 클라이언트의 메시지를 보고 해석이 가능)
   - 다시 말하면, 애플리케이션 상태는 Hyperlink를 통해 전이 되어야 한다.

<br/>

### self-descriptive messages

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F9EbhF%2FbtqvfkAVGAL%2FoskaONyGuZi8rhM0kUscsK%2Fimg.png)

- 이 사진을 보고도 어떤 메시지인지 알아야 한다는 말이다.
- Content-Type을 보고 대괄호, 중괄호 의미를 파악할 수 는 있지만, op와 path의 의미를 전혀 알 수 없음.
- 위 사진은 self-descriptive 하지 않은 메시지다.

그러면 어떻게 해결할 수 있을까?

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqcLki%2Fbtqvb6KQwOG%2FROAwAG7B1u7Zo6ZR4NBV1K%2Fimg.png)

- Content-Type이 application/json-patch+json 이므로, 미디어 타입에 대한 명세를 정의한 문서가 되었다. https://tools.ietf.org/html/rfc6902 로 찾아가서 op와 path의 의미를 해석할 수 있음

---

### hypermedia as the engine of application state(HATEOAS)

- 클라이언트와 서버가 서로 소통하지 않고도 메시지를 해석함으로써 독립적인 진화를 할 수 있음.(서버에서 메시지가 바뀌어도 클라이언트의 메시지를 보고 해석이 가능)
- 다시 말하면, 애플리케이션 상태는 Hyperlink를 통해 전이 되어야 한다.

예를 들면 HTML의 a 태그의 href, HTTP의 Link 헤더

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FZEpqL%2FbtqveHi2fl7%2Fkb7JC6XafqGdfHlito7Ly0%2Fimg.png)

a 태그 안에 있는 하이퍼 링크를 통해 다른 상태로 전이할 수 있음. -> 즉, HATEOAS 만족.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fm9vAg%2FbtqvdlOjicE%2FSvZOfGuq8KYHjFX43i41q1%2Fimg.png)

Link 헤더를 통해 다른 리소스와 하이퍼링크로 연결 시킬 수 있다. 즉, 다른 상태로 전이 가능. -> HATEOAS 만족.

HATEOAS를 지켜야 하는 이유는, 서버에서 링크를 동적으로 변경할 수 있음. 클라이언트는 서버에서 받아온 링크를 가지고 다음 상태로 바뀔 수 있기 때문에 서버에서 링크를 바꾸는 것을 신경쓰지 않고 개발할 수 있다.

## 이유

위 제약 조건을 지키는 이유는 `독립적 진화` 때문이다.

- 서버와 클라이언트가 각각 독립적으로 진화
- 서버의 기능이 변경되어도 클라이언트를 업데이트할 필요 없다.
- REST를 만든 계기는 어떻게 하면 웹을 깨뜨리지 않고 HTTP를 진보시킬 수 있을까에서 나왔다.

[출처](https://dingue.tistory.com/11)



[링크](https://wonit.tistory.com/454) -> 여기 꼭 읽어보삼.



