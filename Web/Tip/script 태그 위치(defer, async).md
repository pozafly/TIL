# script 태그 위치(defer, async)

> 출처 : [script async 와 defer의 차이점 - 드림코딩](https://www.youtube.com/watch?v=tJieVCgGzhs&list=PLv2d7VI9OotTVOL4QmPfvJWPJvkmv6h-2&index=3)

HTML에서 `<script>` 태그를 만났을 때, HTML 파일 파싱을 멈추고 JavaScript 파일을 다운로드한 후 다시 HTML을 파싱한다. 그점을 보고 아래 정보를 보자.



## Head 태그에 넣었을 경우

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Document</title>
    <script src="test.js"></script> // script 태그가 head 태그 안에 위치
  </head>
  <body></body>
</html>
```

![image](https://user-images.githubusercontent.com/59427983/223432088-b0459c24-1a1d-4f63-9134-7b441abd2f3a.png)

js 파일 사이즈가 크거나, 인터넷 환경이 느릴 때 사용자가 웹사이트를 로딩하는데 시간이 오래 걸린다. 따라서 script를 head 태그에 포함하는 것은 좋은 선택은 아니다. 

<br/>

## script 태그가 body 태그의 제일 하단에 위치해 있을 때

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Document</title>
  </head>
  <body>
    <div></div>
    <script src="test.js"></script> // script 태그가 body 태그의 하단에 위치
  </body>
</html>
```

![image](https://user-images.githubusercontent.com/59427983/223432828-d5e91e0d-c272-480c-991a-d7d21306501f.png)

html이 모두 준비가 된 후에 javascript를 실행한다. 사용자가 기본적인 html 컨텐츠를 빨리 볼 수 있는 장점이 있지만, 해당 웹사이트가 **javascript에 의존적이라면 의미 있는 웹페이지를 보기까지 시간이 오래 걸린다**.

<br/>

## script 태그가 head 태그 안에 있지만, async 속성 값을 사용한 경우

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Document</title>
    <script async src="test.js"></script> // async 속성값을 사용
  </head>
  <body>
    <div></div>
  </body>
</html>
```

![image](https://user-images.githubusercontent.com/59427983/223433152-90e69cf5-1cbc-44eb-9d86-bc54b062d75f.png)

async 속성을 사용할 경우, 위 사진과 같이 html 파싱과 javascript 다운로드를 병렬로 수행한다. 다만 javascript 다운로드가 완료되면 **html 파싱을 중지하고, 다운로드된 javascript 파일을 먼저 실행**한 후, 나머지 html을 파싱한다. 

병렬로 진행되기 때문에 웹사이트를 불러오는 속도를 조금 빠르게 할 순 있지만, html이 파싱되기 전에 javascript가 동작하므로 javascript에 html과 상호작용하는 코드가 있을 경우 제대로 작동하지 않을 가능성이 있다.

<br/>

## script 태그가 head 태그 안에 있지만, defer 속성값을 사용한 경우

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Document</title>
    <script defer src="test.js"></script> // defer 속성값을 사용 
  </head>
  <body>
    <div></div>
  </body>
</html>
```

![image](https://user-images.githubusercontent.com/59427983/223433224-f944130c-07c9-4673-9254-cc6f3880aef8.png)

async와 동일하게 html 파싱중에 javascript 다운로드가 병렬적으로 진행된다. 그러나 async와 다르게 html 파싱이 모두 **종료된 후에 javascript를 실행한다.**


