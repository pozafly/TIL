# Mouse Event

마우스 이벤트를 사용해 여러가지 꾸밈 요소를 만들 수 있음.

1. html에 `<div>` 태그를 이용해 마우스 커서를 대체 할 엘리먼트를 생성.
2. css로 `<div>` 태그를 꾸며줌. 포인터를 대체 할.
3. js로 mousemove 이벤트를 먹여주고 transform: translate로 따라오게 할 수 있다.

```js
// js
const cursor = document.querySelector('.cursor_item');

const mouseFunc = (e) => {
  cursor.style.transform = `translate(${e.clientX}px, ${e.clientY}px)`;
};
window.addEventListener('mousemove', mouseFunc, false);
```

```css
// css
.cursor_item {
  position: absolute;
  width: 100px;
  height: 100px;
  background-color: red;
  top: 0;
  left: 0;
}
```

하지만 이렇게만 걸어주면 마우스 움직임이 부드럽지 않다. 여기서 `requestAnimationFrame()` 을 사용하면 된다.

📌 requestAnimationFrame(반복할 함수): 브라우저에게 수행하기 원하는 애니메이션을 알리고, 다음 리페인트가 진행되기 전 해당 애니메이션을 업데이트하는 함수를 호출하게 된다.

> 사용 이유
>
> - 백그라운드 동작 및 비활성화시 중지(성능 최적화)
> - 최대 1ms(1/1000s)로 제한되고, 1초에 60번 동작
> - 다수의 애니메이션에도 각각 타이머 값을 생성 및 참조하지 않고 내부의 동일한 타이머 참조.

※ 리플로우, 리페인트: 브라우저 상 무언가를 렌더할 때, 위치나 색을 계산하는 과정을 **리플로우**라고 하고, 계산을 수행해 브라우저에 그리는 것을 **리페인트**라고 함. 하지만, 애니메이션의 경우, 리페인트 과정이 끝나지도 않았는데 다음 좌표로 이동하라고 애니메이션을 수행하는 경우, 애니메이션이 의도한 대로 부드럽게 움직이지 않게 되는데 requestAnimationFrame() 은 이를 해결해줌.

```js
// 원래 서커 위치
let originX = 0;
let originY = 0;

// 부드럽게 움직일 마우스 커서 위치
let mouseX = 0;
let mouseY = 0;

const speed = 0.03;
const cursor = document.querySelector('.cursor_item');

const mouseFunc = (e) => {
  originX = e.clientX;
  originY = e.clientY;
};
window.addEventListener('mousemove', mouseFunc, false);

const loop = () => {
  mouseX += (originX - mouseX) * speed;
  mouseY += (originY - mouseY) * speed;
  cursor.style.transform = `translate(${mouseX}px, ${mouseY}px)`;
  window.requestAnimationFrame(loop);  // requestAnimationFrame()
};
loop();  // 재귀
```

- loop를 돌리면 requestAnimationFrame이 loop를 계속해서 다시 부름
- speed를 곱한 값을 부드럽게 움질일 `mouseX`, `mouseY` 로 주어 부드럽게 움직이게 함

만약 speed 값을 1로 주면 원래 커서를 따라오는 것과 같은 효과가 남.
