# SVG
svg(Scalable Vector Graphics)는 벡터 그래픽을 표현할 때 사용하는 형식이다. HTML5 부터는 SVG 태그가 표준으로 제정되어 모든 웹 브라우저에서 사용할 수 있게 되었음.

<br/>
<br/>

## 기본
svg 태그에 width, height 속성을 반드시 입력해야 함.
```html
<svg width="700" height="500">
  <rect width="700" height="300" fill="red" />
  <circle cx="350" cy="150" r="100" fill="orange" />
</svg>
```
<img width="720" alt="image" src="https://user-images.githubusercontent.com/59427983/210130557-421875a4-7ba9-4c1d-991d-5c093ba3a6aa.png">
이미지를 사용했을 때와, SVG 태그를 사용했을 경우 차이는
- SVG는 벡터 그래픽을 표현할 때 사용하는 코드이기 때문에 이미지를 확대했을 때 일반적인 레스터 그래픽 이미지는 그림이 계단처럼 끊기는 것을 볼 수 있지만, 벡터는 끊기지 않는다.
- SVG 태그는 이벤트를 연결해 사용자와 동적으로 반응 할 수 있다. 서버에서 실시간으로 데이터를 받아 반영하는 것도 가능.

> 📌 SVG 표준
> \<svg xmlns="http://www.w3.org/2008/svg" version="1.2">\</svg>