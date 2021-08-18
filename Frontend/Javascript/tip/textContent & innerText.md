# textContent & innerText

> [출처1](https://webisfree.com/2020-03-07/[%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8]-textcontent-%EA%B7%B8%EB%A6%AC%EA%B3%A0-innertext-%EC%B0%A8%EC%9D%B4%EC%A0%90-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0), [출처2](https://hianna.tistory.com/483)

엘리먼트 및 노드에 텍스트를 추가하거나 값을 가져올 수 있다.

<br/>

## 자바스크립트 textContent 프로퍼티 알아보기

```js
element.textContent = '내용';
```

텍스트를 추가할 수 있다. 일반적으로 텍스트를 엘리먼트에 추가할 경우 `innerText` 가 많이 쓰임.

<br/>

## 공통점과 차이점

### 공통점 

- 텍스트 노드를 추가함.
  - 둘 모두 텍스트를 추가한다는 공통점이 있다. 결과 역시 동일함.
- 해당 엘리먼트의 텍스트 값을 반환함
  - 둘 모두 어떤 텍스트를 가지고 있는지 알 수 있음.

### 차이점

- textContent가 먼저 만들어짐. 따라서, 브라우저 호환성이 높음. 큰 차이는 아니지만 더 가벼움.
- 값을 가져올 때 다수의 스페이스가 존재하는 경우 하나로 가져오느냐 아니면 모두 가져오느냐 차이가 있음.
  - innerText는 불필요한 공백을 제거하고 텍스트로 반환.
  - textContent는 모든 텍스트를 그대도 가져옴.

즉, 

```html
<p>   Hi   Webisfree.com   Web   Development   </p>
```

이런 태그가 있다고 하면,

```js
const innerTextMsg = document.querySelector('p').innerText;
console.log(innerTextMsg);

-> Hi Webisfree.com Web Development // 출력 결과

-----------------------------------------------------------------------

const textContentMsg = document.querySelector('p').textContent;
console.log(textContentMsg);

-> Hi   Webisfree.com   Web   Development   // 출력 결과
```

이렇게 공백을 가진 경우 달라지게 된다. 실제로 여러 브라우저에서 테스트를 해보면 결과가 많이 달라짐. IE의 경우 위와 같이 나타나지만, 크롬 버전에서는 둘 다 동일하게 결과가 나타남.

<br/>

또한, textContent는 숨겨진 텍스트 값도 함께 읽어온다.

```html
<div>
  안녕하세요?         만나서 반가워요.
  <span style='display:none'>숨겨진 텍스트</span>
</div>
```

이렇게 html이 있다고 하면,

```js
const innerTextMsg = document.querySelector('div').innerText;
console.log(innerTextMsg);

-> 안녕하세요? 만나서 반가워요. // 출력 결과

-----------------------------------------------------------------------

const textContentMsg = document.querySelector('div').textContent;
console.log(textContentMsg);

-> 안녕하세요?         만나서 반가워요.
   숨겨진 텍스트  // 출력 결과
```

이렇게 `display: none` 인 녀석도 가져온다.