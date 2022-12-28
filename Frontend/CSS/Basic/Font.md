## line-height

글 사이의 간격을 띄운다. 행간을 조정한다고 함. line-height는 줄의 높이를 의미하는 것이고 이를 이용해 행간을 조정할 수 있음.

![image](https://user-images.githubusercontent.com/59427983/136688255-c29cf0b5-5bf9-439a-8876-47a715c16427.png)

📌 line-height는 block 속성의 element를 수직 정렬할 때도 아주 유용하게 사용함.
```html
<style>
.button {
	width: 100px;
	height: 70px;  // 🚀
}
a {
	display: block;
	line-height: 70px;  // 🚀
}
</style>

<div class="button">
    <a href="#">Click</a>
</div>
```

🚀 height, line-height 가 동일 px로 되어있고, a 태그가 block 속성이므로 line-height 로 수직 정렬을 맞춰줄 수 있다. 
즉, 글자를 감싸는 박스의 높이와 같은 크기의 line-height를 사용한 것임. 이는 ⭐️이메일 템플릿 코드를 작성할 때도 동일하게 적용된다.

<br/>
<br/>

## vertical-align

텍스트를 수직 정렬 할 수 있다. 📌 주의점은 block 요소가 아닌, **inline** 또는 **inline-block** 요소에서만 사용할 수 있다. 따라서 display 속성이 변하지 않은 div, p와 같은 block 요소에는 적용되지 않는다.

대부분 부모 요소에 상대적으로 정렬된다.

<br/>
<br/>

## text-align

수평정렬에 사용된다. 정렬하고자 하는 inline의 상위 block 요소에 선언해줘야 한다. 즉, 상위에 해줘야 한다는 것임. inline 요소에는 너비가 존재하지 않는다. 너비가 존재해야 가운데 정렬이 되겠지?
하지만 block 요소는 너비를 가지고 있으므로 수평 가운데 정렬이 가능하다.

<br/>
<br/>

## text-decoration

`<a>` 태그를 사용하면 기본적으로 밑줄이 그어져 있음. 이런 녀석들을 제어할 수 있다. none, underline, overline, line-through 같은 녀석을 사용할 수 있음.

<br/>
<br/>

## 속성-단어 관련 속성

- `white-space` : 요소 안 공백을 어떻게 처리할 지 지정
  - **normal**(default) : 공백과 개행을 무시, 필요한 경우 자동 줄바꿈
  - **nowrap** : 공백과 개행 무시, 자동 줄바꿈 일어나지 않음.
  - pre : 공백과 개행 표현, 자동 줄바꿈 일어나지 않음.
  - pre-line : 공백 무시, 개행만 표현. 필요한 경우 자동 줄바꿈 발생.
  - pre-wrap : 개행 무시, 공백만 표현. 필요한 경우 자동 줄바꿈 발생.
- `letter-space` : 자간 지정. px 단위
- `word-space` : 단어 사이 간격. px 단위
- `word-break` : 단어가 라인 끝에 나올 경우 어떻게 처리할지(중단점) 지정
  - normal(default) : 중단점은 공백이나 하이픈(-)
  - break-all : 중단점은 음절. 모든 글자가 요소를 벗어나지 않고 개행
  - keep-all : 중단점은 공백이나 하이픈(-)
- `word-wrap` : 요소를 벗어난 단어의 줄바꿈을 지정
  - normal(default) : 중단점에서 개행.
  - break-word : 모든 글자가 요소를 벗어나지 않고 강제로 개행.

<br/>
<br/>

## Font Family
font-family 속성에는 사용자 컴퓨터에 설치된 폰트를 사용한다.
- 일반적으로 한 단어로 이루어진 폰트는 따옴표를 사용하지 않음.
- 하지만 두 단어 이상으로 이루어지는 단어는 따옴표를 반드시 사용해야 함.
```css
.font-arial {
	font-family: Arial;
}
.font-roman {
	font-family: 'Times New Roman';
}
```
주의 점은, 개발하고 있는 local 컴퓨터는 폰트가 설치되어 있지만, 개발한 웹 페이지를 사용할 사용자에게는 폰트가 설치되어 있지 않을 수 있다. 따라서, 속성을 여러개 사용한다.

```css
.font-arial {
	font-family: '없는 폰트', Arial;
}
```
또, 다국어 웹 페이지를 제공할 경우 사용자에게 무슨 폰트가 있는지 일일이 확인할 수 없다. 따라서, font-family 속성의 가장 마지막 폰트에는 **Serif** 폰트(명조체), **Sans-serif** 폰트(고딕체), **Mono space** 폰트(고정폭 글꼴)를 적용한다.

> 모서리에 돌기가 있는 글자를 Serif 폰트(명조체)라고 부르고 모서리에 돌기가 없는 글자를 Sans-serif 폰트(고딕체) 라고 부름.

##