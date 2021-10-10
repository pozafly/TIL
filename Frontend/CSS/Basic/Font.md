# Font

## line-height

글 사이의 간격을 띄운다. 행간을 조정한다고 함. line-height는 줄의 높이를 의미하는 것이고 이를 이용해 행간을 조정할 수 있음.

![image](https://user-images.githubusercontent.com/59427983/136688255-c29cf0b5-5bf9-439a-8876-47a715c16427.png)

<br/>

<br/>

## vertical-align

텍스트를 수직 정렬 할 수 있다. 📌 주의점은 block 요소가 아닌, **inline** 또는 **inline-block** 요소에서만 사용할 수 있다. 따라서 display 속성이 변하지 않은 div, p와 같은 block 요소에는 적용되지 않는다.

대부분 부모 요소에 상대적으로 정렬된다.

<br/>

<br/>

## text-align

수평정렬에 사용된다. 정렬하고자 하는 inline의 상위 block 요소에 선언해줘야 한다. 즉, 상위에 해줘야 한다는 것임.

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
