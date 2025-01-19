# 모바일 사파리 클릭영역 background

> [출처](https://stackoverflow.com/questions/11885161/remove-grey-background-on-link-clicked-in-ios-safari-chrome-firefox)

데스크탑은 생기지 않는다. 하지만 모바일 사파리에서 사진과 같이 클릭했을 때, 배경에 색상이 들어가는 경우가 생긴다. 이는 css의 -webkit-tap-highlight-color 때문에 생기는 문제다. 여기 색상을 지정해주면 클릭했을 때 색상이 생기지 않는다.

<img width="662" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/d7d941a1-6d2d-4d5d-ba1e-9254949801d9">

이런 식으로 클릭하면 영역에 background 표시가 들어온다.

```css
-webkit-tap-highlight-color: rgba(0, 0, 0, 0);
또는
-webkit-tap-highlight-color: transparent;
```

이렇게 적용해주자.
