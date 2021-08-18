# HTML 요소 크기(offsetWidth, clientWidth, scrollWidth, getBoundingClientRect)

> 출처 : https://so4869.tistory.com/26

<br/>

<br/>

1. `offsetWidth` (Height)

margin을 제외한, padding값, border값까지 계산한 값을 가져옴.(일반적으로 많이 쓰임)

```html
<style>
  * {
    margin: 0;
    box-sizing: border-box;
  }
  .asdf {
    width: 100px;
    height: 100px;
    
    margin: 30px;
    padding: 20px;
    border: 10px solid #000;
    
    overflow: scroll;
  }
</style>

<div class="asdf">글씨글씨글씨글씨글씨글씨글씨글씨</div>
<div class="result"></div>

<script>
  let div = document.querySelector('.asdf');
  let result = document.querySelector('.result');
  
  result.innerText = div.offsetWidth + ' X ' + div.offsetHieght;
</script>
```

그러면 result에는 `160 X 160` 이 찍히게 된다. 즉, margin을 제외한 padding 값과 border 값이 포함된 크기를 가져옴.

<br/>

<br/>

2. `clientWidth` (Height)

margin값과 border 값이 제외 된, padding 값까지만 적용된 내부의 실제 크기를 가져옴. border는 외부에 속한다.

```html
<style>
  * {
    margin: 0;
  }
  .asdf{
    width: 100px;
    height: 100px;

    margin: 30px;
    padding: 20px;
    border: 10px solid #20c997;

    overflow: scroll;
  }
</style>

<div class="asdf">글씨글씨글씨글씨글씨글씨글씨글씨</div>
<div class="result"></div>

<script>
  let div = document.querySelector('.asdf');
  let result = document.querySelector('.result');
  
  result.innerText = div.clientWidth + ' x ' + div.clientHeight;
</script>
```

결과는 `140 X 140` 이 찍히게 됨. 즉, margin, border 값이 빠진 padding 값만 반영되어 실제 크기만 가져와진 것임. 엄밀히 말하면 scroll bar 크기도 빠져있음.

<br/>

<br/>

3. `scrollWidth` (Height)

스크롤 영역일 때, 스크롤로 감싸여진 내용을 전체 크기를 가져옴.

```html
<style>
  * {
    margin: 0;
  }
  .asdf{
    width: 100px;
    height: 100px;

    margin: 30px;
    padding: 20px;
    border: 10px solid #20c997;

    background-color: #22b8cf;
    color: #fff;
    overflow: scroll;
  }
</style>
<div class="asdf">글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨</div>
<div class="result"></div>

<script>
  let div = document.querySelector('.asdf');
  let result = document.querySelector('.result');
  
  result.innerText = div.scrollWidth + ' x ' + div.scrollHeight;
</script>
```

결과는 `125 x 154` 이다. 스크롤을 더 크게 만들기 위해 글씨를 더 추가한 결과임. 스크롤로 가져린 전체 크기가 가져와졌다.

<br/>

<br/>

4. `getBoundingClientRect()` 

실제 랜더린 크기임.

```html
<style>
  *{
    margin: 0;
  }
  .asdf{
    width: 100px;
    height: 100px;

    margin: 30px;
    padding: 20px;
    border: 10px solid #20c997;

    background-color: #22b8cf;
    color: #fff;
    overflow: scroll;
  }
</style>

<div class="asdf">글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨글씨</div>
<div class="result"></div>

<script>
  let div = document.querySelector('.asdf');
  let result = document.querySelector('.result');
  
  result.innerText = div.getBoundingClientRect();
  console.log(div.getBoundingClientRect());
</script>
```

결과는 `[object DOMRect]` 임. 기본적으로 top, right, bottom, left, x, y의 값이 콘솔에 찍힌다. 현재 console에 찍힌 결과는 아래와 같음.

<img width="449" alt="스크린샷 2021-08-17 오후 5 44 49" src="https://user-images.githubusercontent.com/59427983/129694209-9be9c073-66f4-48c7-9c47-c4929f27c46f.png">

보면, 현재 찍힌 것은 `offsetWidth` 의 결과와 같다. **그러나 이 함수는 랜더링 된 결과를 가져온다.**

```html
<style>
  .asdf {
    (...)
    transform: scale(1.2);
  }
</style>
```

이렇게 추가했다고 하자. 즉, asdf 박스에 transform: scale(1.2);를 주었다. 1.2배로 늘렸다. result에는 offsetWidth (Height)를 출력했고, console에는 getBoundingClientRect함수의 실행결과를 출력했다.

결과는 아래와 같다.

`160 X 160` 이 찍혔고, 

<img width="446" alt="스크린샷 2021-08-17 오후 5 48 11" src="https://user-images.githubusercontent.com/59427983/129694634-201b58e6-15c4-4b9a-bd9c-0ac896395725.png">

값이 바뀌었다. 즉, getBoundingClientRect 함수 실행결과는 실제 랜더린 크기임. 1.2 배로 늘렸기 때문에 160 * 1.2 한 결과인 192이 나온 것이다.