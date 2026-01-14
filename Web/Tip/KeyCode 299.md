# KeyCode 229

> 출처: https://circus7.tistory.com/6

`<input type="text" />` 에서 인풋 이벤트의 keyCode를 이용해 메서드를 만들 때, keyCode 229 버그가 나올 때가 있음. 특히 모바일에서는 숫자 키에도 229 가 나옴.

<br/>

input에서 한글 자판 사용시 IME에서 메세지를 가로채기 때문에 keyCode가 229를 가리키는 것임. 그래서 **keydown**, **keyup **이벤트 대신, **input** 이벤트를 사용하는 것을 추천함.

<br/>

한글이나, 일본어, 중국어 등 컴퓨터 자판의 개수보다 더 많은 글자를 쓰는 문자를 입력할 때 입력 방식 편집기(IME)를 사용한다. 안드로이드 모바일 역시 한글을 입력할 때, IME를 사용하고 있었다.

IME는 한글을 입력할 때, **컴포징** 이라는 단계를 거친다. 예를 들어, "ㄱ" 을 입력하면 아직 글자가 완성되지 않았다고 판단하고 글자가 완성될 때까지 조합을 하는데, 이걸 컴포지션이라고 부름.

문제는 컴포징이 진행 중일 때에도 keydown 및, keyup 이벤트를 보낸다는 것이다. 컴포징이 시작되었을 때, 미완성이라는 메세지로 keyCode 229를 보내는 것임.

<br/>

아래는 textarea에서 한글 "ㅁ" 을 입력했을 때, keydown, keyup 이벤트를 console에 찍은 것임. (input text 에서도 같은 결과임.)

![[assets/images/fdee14497c37ca2d7455b98482cc8a97_MD5.png]]

![[assets/images/fb3ef360ccd00310ccf64c633e953f06_MD5.png]]

위에서, `keyCode: 229` 가 나온 것을 알 수 있음. 그리고, `key: "Process"` 를 통해 IME의 프로세스로 들어간다는 것을 확인할 수 있음.

하지만 특이하게 keydown 이벤트에는 `isComposing: false` 인데 반해 keyup 이벤트에서는 `isComposing: true` 라는 것을 확인할 수 있음. 이것은 keydown에서는 아직 컴포징이 시작되지 않았기 때문임. 이 때, "ㅏ" 를 연달아 치면 "마" 가 되면서 keydown 이벤트 역시 `isComposing: true` 가 되는 것을 확인할 수 있다.

![[assets/images/914241b5624491caee6cc0ddfa388a23_MD5.png]]

그래서 한글 입력인지 알고 싶다면 아래와 같은 로직을 통해 이벤트 로직을 짤 수 있다.

```js
input.addEventListener('keyup', (event) => {
  if(evnet.isComposing) {
    // 한글 입력 중
    // return을 입력하면 한글 입력 이벤트만 받을 수 있음.
  }
  // Do Something
});
```

하지만, keydown이나, keyup으로 input의 각 character에 즉각적으로 작동하는 이벤트는 사용하지 않는 것을 권고하고 있다. 왜냐면 keydown과 keyup 이벤트에서 컴포지팅 메세지를 보내는 것이 고쳐지지 않을 수 있기 때문임.

대신, input 이벤트를 사용하는 것을 추천하고 있다. input 이벤트는 element의 value가 변할 때(change) 마다 발화하는 이벤트임.
