# Flux 패턴

> [출처](https://www.tcpschool.com/react/react_redux_intro)

MVC 패턴이란, Model에 데이터를 저장하고, Controller를 사용해 Model의 데이터를 관리하는 소프트웨어 디자인 패턴의 하나다. 이런 MVC 패턴에서는 사용자가 View를 통해 데이터를 입력하면 View에서도 Model을 업데이트 할 수 있기 때문에 데이터가 양방향으로 전달될 수 있다.

하지만, 이렇게 구현된 프로젝트는 프로젝트의 규모가 커질수록 수많은 Model과 View가 생성되고, 이 때 Model의 상태에 변화가 생길 경우 Model과 View 사이에 엄청난 양의 데이터가 양방향으로 전달되게 된다. 이로 인해 데이터의 흐름을 예측하기 힘들어지고 수많은 버그 발생.

![[assets/images/db9cdcad1e5869918c62b134f751ff94_MD5.png]]

Flux 패턴은 사용자가 입력을 기반으로 Action을 생성하고, 이를 Dispatcher에 전달, Store의 데이터를 변경한 뒤 View에 반영하는 단방향 데이터 흐름을 가지는 아키텍처다.

![[assets/images/ba22290ce7ae1bede13f6adec1c8c26e_MD5.png]]
