# FSD 비판

> [출처](https://medium.com/@junep/fsd-feature-sliced-design%EC%97%90-%EB%8C%80%ED%95%B4-11a7b88d5c9e)

[기능 분할 설계 FSD](https://emewjin.github.io/feature-sliced-design/?source=post_page-----11a7b88d5c9e--------------------------------)

## 시작

FSD에 보기 전 짚어야 할 부분은 '견고함' 이다. 어떤 구조든 견고할 수록 무너지기 쉽다. 빠듯한 계획은 아주 작은 약속 불이행으로 깨진다. 반면 느슨한 계획은 일정 규모 이상의 예측 실패에도 유연하게 대응할 수 있다. 이런 시각으로는 FSD는 다소 견고하게 느껴진다.

FSD는 각 레이어가 다루는 부분이 모호하지 않다. 어떤 대상이 레이어를 벗어나 다른 레이어에도 포함될 수 있는 여지가 크게 존재하지 않는다. 예를 들어, entities에 속해야 하는 무언가가 features에 속하기는 어려워 보인다. 즉, 가능할 수도 있지만 다소 억지스러운 측면이 있음.

만약 'A 레이어에도 속할 수 있지만, B레이어에 두기로 했다'는 결정을 했다면 FSD를 깨뜨리고 있는게 아닌지, FSD를 사용하면서 이전의 방법으로 돌아간건 아닌지 고민해봐야 한다.

<br/>

## 서문

> The **layers** are standardized across all projects and vertically arranged. Modules on one layer can only interact with modules from the layers strictly below. There are currently seven of them (bottom to top):

`vertically arranged`라는 건 vertical slice architecture와 비슷하다.

- [The Problem with Clean Architecture: Vertical Slices](https://medium.com/design-microservices-architecture-with-patterns/the-problem-with-clean-architecture-vertical-slices-111537c0ffcb)

- [Clean Architecture vs Vertical Slice Architecture](https://dev.to/rexebin/clean-architecture-vs-vertical-slice-architecture-3mja?source=post_page-----11a7b88d5c9e--------------------------------)

이 방법은 FE 입장에서 보면 어떤 페이지 또는 기능을 수정하거나 추가할 때 해당 페이지 또는 기능만 살펴보는 걸로 가능하게 하는 것이다. 따라서 페이지 간 공유하는 컴포넌트, 로직 등을 최소화 하는게 중요하다. 마치 기능 별로 인프라까지 나누는 마이크로서비스와 일맥상통하다.

`Modules on one layer can only interact with modules from the layers strictly below.` 이 문장은 항상 옳은 문장은 아니지만, FSD 뿐 아니라 계층을 나눈다면 어떤 방법론에서든 참고해야 하는 문장이다. 만약 자신에게 인접한 계층을 통하지 않고 지나쳐서 다른 계층과 커뮤니케이션, 메시지를 주고 받는다면 계층이 깨지고 유지보수가 어려워질 수 있다. (하지만 답글에 적힌 것처럼 지나치는 계층이 열려있는지 닫혀있는지에 따라 복잡성 및 유지보수성의 난이도는 달라질 수 있다.)

많이 언급되는 [디미터 법칙](https://en.wikipedia.org/wiki/Law_of_Demeter)도 이와 관련이 있고 사실 대부분 SW 엔지니어링에서 통용되는 개념이다. 근본적으로 캡슐화를 해치기 때문임.

### shared

`detached from the specific of the project/business`. 특정 프로젝트 또는 비즈니스에서 분리되어(떨어져나와) 재사용할 수 있는 기능이라는 건 강력한 조건이라고 생각한다. 왜냐면 반대로 프로젝트나 비즈니스에 붙어 있다면 함부로 분리하면 안 된다는 의미이기도 하기 때문이다.

따라서 특정 페이지나 프로젝트에서 사용하고 있는 비즈니스 로직이 있고, 다른 프로젝트에서도 사용한다면 함부로 분리하면 안된다. 그렇게 되면 공통 로직이 거대해지고 수직 분할을 했을 때 효용이 줄어든다.

### entities

엔티티란 비즈니스 로직의 주체다. 또한 개인적으로 서구권 언어의 특징이라고 생각하기도 함. 동양권, 서구권의 가장 큰 차이 중 하나는 대상을 어떻게 바라보느냐다. 우리는 물건이 무얼 한다고 하면 이상하게 생각한다. '빨간 불이니까 (우린) 움직이면 안되' 라고 하지, '빨간 불이 우리에게 멈추라고 했어' 라고 하면 어색하다. 반면 영어는 그렇게 표현하는게 가능하다. 'The red light says stop!'

그렇게 보면 엔티티는 비즈니스 로직의 주체다. 사용자가 될 수도 있고 상품이나 주문이 될 수도 있다.  예를 들어 우리는 사용자가 주문을 취소하지만 영어식 표현으로 보면 사람이 주문에게 취소를 요청하고 주문은 요청받은 취소를 수행한다. 그렇기 때문에 주문도 엔티티가 될 수 있다.

이런 측면에서 보면 FSD에서 엔티티는 특정 프레임워크나 방법론에 속하는 게 아닌 범용적 비즈니스 로직이다. 그리고 그래야만 다른 계층과 섞이지 않는다.

[Entity](https://javascript.plainenglish.io/practical-ddd-in-typescript-entity-fc79002d0015)는 [Value Object](https://javascript.plainenglish.io/practical-ddd-in-typescript-value-object-b76bcd2d9283)와 비교해서 볼 때 더 도움이 되기도 한다. 특히 에릭 에반스의 [도메인 주도 설계](https://product.kyobobook.co.kr/detail/S000001514402)를 참고하는 게 가장 좋다.

### features

FE는 사용자의 동작을 입력 받아 데이터로 변환하는 작업을 수행하는 곳이고, 반대로 데이터를 사용자가 이해할 수 있는 형태로 출력하는 곳이다. 그런 측면에서 features는 비즈니스 로직의 값 `business value`, 즉 데이터를 UI의 상태 (리액트의 state 또는 BEM의 class selector)로 변환하거나 그런 값들을 비즈니스 값으로 변환하는 곳이다.

따라서 이벤트 리스너, 훅, 상태 관리 도구, 리액트 쿼리 등이 위치할 수 있겠다.

### widgets

사실 `compositional layer to combine entities and features into meaningful blocks` 문장이 모든 걸 설명함. 이 패턴은 [Controller, Humble Object](https://martinfowler.com/bliki/HumbleObject.html)에서도 잘 드러납니다.

즉, 엔티티에 속하지 않고 features에 속하지 않으면서 이 둘을 합쳐 활용하는 곳. 리액트에선 컴포넌트와 훅이 대표적이다. 물론 MVC 패턴에서 C(Controller)에 해당한다.

### pages

`from entities, features and widgets`. pages 부터는 절대적 계층이 조금 허물어진다는 의미로 받아들여진다. 즉, pages에서 무언가 구성하다보면 엔티티에 바로 접근할 수도 있고 피처스에도 그렇다.

이건 FE 구조의 특징으로 페이지 역시 페이지만의 고유한 기능을 담당하고 있기 때문이다.

### app

우리가 만든 어플을 전체적으로 관리하는 대상을 포함한다.

<br/>

## 마무리

공리주의를 주장하는 정치철학자 처럼 '이럴 땐 이렇게 저럴 땐 저렇게' 라며 편리하게 의견을 바꿔선 안된다. 하지만 공리주의를 선호하는 개인은 때론 개인주의적일 수 있다.

FSD는 견고한 구조를 갖는다. 마치 철학처럼 각 조직마다 유연하게 사용하겠지만 원리, 이론 등 가장 기본이 되는 곳은 그만큼 견고해야 한다.

하지만, 그럼에도 근간이 되는 방법을 크게 벗어나지 않도록 하는게 좋다. 왜냐면 우리만의 이유가 늘어난다는 건 그만한 근거가 충분해야 하고, 많은 사람이 공유하려면 그만큼 비용이 더들기 때문이다. 무엇보다 우리만의 이유를 만든다는 건 그만큼 의견 충돌이 많이 발생할 수 있고 아마 가장 큰 비용이 되지 않을까 한다.


