# Page Visibility API

> [출처](https://mong-blog.tistory.com/entry/Page-Visibility-API-%EC%9C%A0%EC%A0%80%EA%B0%80-%ED%8E%98%EC%9D%B4%EC%A7%80%EB%A5%BC-%EB%B3%B4%EA%B3%A0-%EC%9E%88%EB%8A%94%EC%A7%80-%EC%95%8C%EB%A0%A4%EC%A4%98)

소켓 통신이 발생하는 페이지에서, 다른 탭을 보고 있거나 페이지를 최소화 했을 때, 소켓 통신을 비활성화 해야 한다면?

`Page Visibility API` 를 사용하면 된다.

1. 유가 페이지를 보고 있지 않을 때, 불필요한 소켓 통신 중단 가능
2. 유가 페이지를 보고 있지 않을 때, video를 멈추고, 탭으로 다시 돌아왔을 때 다시 재생 가능
3. 유가 페이지를 보고 있지 않을 때, 사운드를 줄이고 탭으로 돌아왔을 때 사운드 회복 가능

## 1. Page Visibility API란?

- 유저가 페이지를 보고 있는지를 이벤트와 속성 값으로 제공한다.
- 예를 들어, 유저가 페이지를 최소화 했거나 다른 탭을 보고 있는지를 감지할 수 있다.

### 1) 속성

#### (1). document.hidden

이 속성을 페이지가 유저에게 보이지 않을 때 `true`를 반환하고, 보일 때 `false`를 반환한다.

```js
document.addEventListener('visibilitychange', () => {
  console.log(document.hidden); // boolean
});
```

다른 탭을 보고 왔을 때 상태 변경이 일어나 `document.hidden`이 `false` 로 출력된다.

![[assets/images/b7240fb326644eb25e023b768c3aca91_MD5.gif]]

단, 사전 렌더링(`prerendering`)일 때는 사용자에게 페이지가 보여지더라도 `document.hidden`이 `true`일 수 있다.(ex. 새 브라우저 탭을 열고 url을 붙여넣어 해당 url로 이동할 때)

#### (2) document.visibilityState

이 속성은 페이지가 유저에게 보여지는 상태를 문자열로 반환한다.

| 리턴 값 | 설명                                                         |
| ------- | ------------------------------------------------------------ |
| visible | 유저가 페이지의 콘텐츠를 보고 있는 상태                      |
| hidden  | 유저가 페이지의 콘텐츠를 보고 있지 않은 상태<br />ex) 백그라운드 탭일 때, 창 최소화 했을 때, OS 잠금 모드일 때 |

```js
document.addEventListener("visibilitychange", () => {
  console.log(document.visibilityState); // 'visible' or 'hidden'
});
```

다른 탭을 보고 왔을 때, 상태 변경이 일어나 `document.visibilityState`가 `visible`로 출력된 모습

![[assets/images/a72dc470698d4329b247ea931db0ab2a_MD5.gif]]

<br/>

### 2) 이벤트

#### (1) Visibilitychange event

페이지의 콘텐츠가 숨겨지거나 표시될 때 이벤트가 발생한다.

```js
addEventLinstener('visibilitychange', (event) => {});
onvisibilitychange = (event) => {};
```

<br/>

### 3) 사용 예시 보기

`visibilitychange`와 `document.hidden`을 사용해, 페이지가 보여질 때는 오디오를 플레이하고 그렇지 않을 때는 오디오를 중단한다.

```js
const audio = document.querySelector("audio");

document.addEventListener("visibilitychange", () => {
  if (document.hidden) audio.pause();
  else audio.play();
});
```

gif에서 보다 시피 탭을 전환하거나 페이지를 최소화 했을 때 오디오가 중단되고 다시 페이지가 보여질 때 오디오가 재생된다.

![[assets/images/853091973d891907bbb7613fcdc043b1_MD5.gif]]

([예시 코드 보기](https://codepen.io/kumjungmin/pen/mdQjKKz))

<br/>

## 2. 마치며

`Page Visibility API`는 웹 페이지의 보이는 상태를 감지할 수 있는데, 불필요한 소켓통신, 비디오 재생 등을 막는데 사용할 수 있다.

특히 소켓 통신의 경우, 클라이언트와 서버 간 많은 소켓 통신을 막을 수 있는 장점이 있다. 만약 서비스에서 불필요한 트래픽을 줄이고 싶다면, 이 API를 사용하도록 하자.
