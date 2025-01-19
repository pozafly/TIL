# Position

> [출처](https://velog.io/@iamhayoung/CSS-Position-Display-Float%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)

<br/>

## Position

```css
.box {
  position: 값;
  /* 아래의 값은 position의 값이 static일 때 적용되지 않음 */
  /* position의 값을 기준으로 구체적인 상하좌우의 위치를 숫자로 지정 가능 */
  top: 1px;
  bottom: 1px;
  left: 1px;
  right: 1px;
}
```

position은 이름 그대로 위치를 결정하는 속성. 요소의 위치를 어떤 기준으로, 어디에 배치시킬 것인지 표현할 수 있다.

| value    | description                                                  |
| -------- | ------------------------------------------------------------ |
| static   | default값. 기준 위치를 설정하지 않음.                        |
| relative | 현재 위치를 기준으로 상대 위치를 지정.                       |
| absolute | 부모 위치를 기준으로 절대 위치를 지정.                       |
| fixed    | 윈도우(브라우저 창)를 기준으로 절대 위치를 지정해 요소를 그 위치에 고정(fix)시킴 |
| sticky   | 지정된 위치에 뷰포트가 도달했을 때, 요소가 그 위치에 고정(fix)됨 |

### Relative

현재 위치를 기준으로 상대 위치를 지정.

```css
.box.two {
  position: relative;
  top: 30px;
  left: 50px;
}
```

![position relative](https://user-images.githubusercontent.com/59427983/119773194-3a5a5780-befb-11eb-84fa-10b55250e09d.png)

> 특징 정리
>
> - top, left, right, bottom 의 지정이 가능.
> - z-index의 지정이 가능.
> - relative의 기준점은 요소 자기 자신이 원래 배치되었던 위치.
> - `absolute` 의 기준 위치가 됨.

<br/>

### Absolute

absolute는 relative나 fixed가 지정된 부모 위치를 기준으로 자신의 위치를 지정할 수 있게 해준다.

```css
.parent {
  position: relative;
}

.box3 {
  position: absolute;
  left: 200px;
  bottom: 100px;
}
```

![position absolute](https://user-images.githubusercontent.com/59427983/119773675-f1ef6980-befb-11eb-86f8-604f703e3bae.png)

> 특징 정리
>
> - 부모 요소(=기준 위치)에 relative또는 fixed를 지정해야 함.
> - 자식 요소(=이동시키고 싶은 요소)에 absolute를 지정.
> - 자식 요소를 top, left, right, bottom 등으로 구체적 위치를 조절할 수 있다.

<br/>

### Fixed

브라우저 화면(윈도우)을 기준으로 요소를 정해진 위치에 고정할 수 있다.

```css
.btn-top {
  position: fixed;
  ...
  right: 20px;
  bottom: 60px;
  ...
}
```

![스크린샷 2021-05-27 오후 3 00 49](https://user-images.githubusercontent.com/59427983/119773932-55799700-befc-11eb-84e4-55ac5e496b16.png)

화살표는 스크롤 해봐도 고정되어 있다. fixed는 브라우저 화면이 기준점이 된다. 이렇게 탑 이동 버튼 뿐 아니라 header를 페이지 상단에 고정시킬 때도 빈번하게 사용된다.

> 특징 정리
>
> - 브라우저 화면(윈도우) 전체가 기준점이 됨.
> - 스크롤 해도 요소가 지정된 위치에 계속 고정되어 있다.
> - top, bottom, left, right 지정 가능
> - z-index 지정이 가능.

<br/>

### Sticky

지정된 기준점 top, left, left, right 등으로 설정해둔 위치)에 도달했을 때, 그 기준점에 요소를 고정(fix) 시켜줌.

```css
<p>스크롤 해보시라우😸</p>
<p>스크롤 해보시라우😸</p>
<div class="menu">menu</div>
<p>스크롤 해보시라우😸</p>
<p>스크롤 해보시라우😸</p>

(...)

.menu {
  position: -webkit-sticky; /* Safari 호환을 위한 Vendor prefix */
  position: sticky;
  top: 0;
}
```

![스크린샷 2021-05-27 오후 3 04 41](https://user-images.githubusercontent.com/59427983/119774268-e05a9180-befc-11eb-8a27-53fbbff740a4.png)

기준점은 `top: 0` 이다. 브라우저(윈도우)의 최상단 지점이다. 즉, menu가 top: 0 이 되면 거기서 멈춰있다.

> 특징 정리
>
> - 브라우저 화면(윈도우) 전체가 기준점.
> - 기준점에 도달했을 때 요소가 그 위치(기준점)에 고정(fix)됨.
> - top, bottom, right, left 지정 가능.
> - z-index 지정 가능.
