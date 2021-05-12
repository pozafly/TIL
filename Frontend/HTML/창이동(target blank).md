# target black

> 출처 : https://webruden.tistory.com/262

새로운 창으로 이동할 때, `<a href="https://naver.com" target="_blank">새창열기</a>` 이런식으로 target="_black" 를 쓰는 경우가 많다. 이 속성을 같이 사용할 때 같이 사용해야 하는게 있음. noopener, noreferrer, nofollow 다.

## noopener

target="_black" 만 사용했을때.

<img width="651" alt="스크린샷 2021-03-19 오전 10 11 06" src="https://user-images.githubusercontent.com/59427983/111717486-776ef180-889b-11eb-8a69-f246ecfa4b70.png">

`window.opener` 객체를 사용할 수 있다. 연결중인 페이지는 연결 페이지에 부분적으로 액세스 할 수 있는 것. 이전 페이지를 제어할 수 있는 권한이 있다는 것. 악의적인 동작을 일으킬 수 있음.

```html
<a href="https://naver.com" target="_blank" rel="noopener">새창열기</a>
```

하지만, noopener 값을 rel 속성으로 사용하면 새로 열린 페이지에서 window.opener 객체가 존재하지 않는 것을 확인할 수 있다.

<br/>

## noreferrer

```html
<a href="https://naver.com" target="_blank" rel="noreferrer">새창열기</a>
```

noopener 와 유사한 기능으로, noreferrer는 새로 열린 사이트가 window.opener 객체를 조작하지 못하게 함. 그러나 noreferrer 는 다른 페이지를 탐색할 때 브라우저가 참조 웹 페이지의 주소를 보내지 못하게 함.

쉽게 말해 noreferrer 값은 링크를 클릭할 때 참조자 정보를 숨긴다. 예를 들어, 누군가 자신의 웹 페이지에 링크를 게시하고 noreferrer 값이 포함된 링크를 클릭하면 해당 사용자가 어디에서 왔는지 알 수 있는 방법이 없음.

<br/>

## 언제 사용할까?

둘 다 사용하는게 좋다. 최신 브라우저 대부분은 noopener를 지원하지만 구형은 지원 하지 않을 수 있음. noreferrer 를 사용할 수 있음.

<br/>

## nofollow

SEO(검색 엔진 최적화)에서는 페이지에 연결되는 가치있는 링크들을 얻는 것이 중요하다. 이를 `백링크` 라고 한다. 그러나 모든 링크가 동일하게 생성되는 것은 아님.

구글에서 **페이지랭크(PageRank)** 라는 용어를 링크 수량 및 품질의 척도로 사용하는데, 링크 주스를 전달하고 싶지 않을 수 있다. **링크주스(Link Juice)**란, 웹 페이지의 링크를 타고 하르는 링크의 가치, 검색 순위를 결정하는 요소 중 하나다. 페이지를 내부적으로 링크를 생성할 때, 주스를 전달하고 싶지 않을 수 있다. 그럴 때, `rel="nofollow"` 를 추가하면 됨. 이 값을 추가하면 페이지로 전달하고 싶지 않다는 것을 검색 엔진에게 알려줌.