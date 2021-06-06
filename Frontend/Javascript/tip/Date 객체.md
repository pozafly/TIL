# Date 객체

Date 객채는 날짜 관련 객체다. Date객체는 java에서와 같이 객체를 한개 생성 후 메서드로 현재 날짜를 불러온다.

- getFullYear() : 년도 4자리.
- getMonth() : 월 정보. 근데 1월은 0을 가져오고 2월은 1 가져옴. ... 왜냐? 외국은 1월이 없고 제니워리 기 때문이다. 따라서 우리나라 표기법은 +1 해줘야함.
- getDate() : 일 정보를 가져옴.
- getDay() : 0 = 일요일, 1 = 월요일 ...6 = 토요일



```js
var now = new Date();
document.write(now.getFullYear() + "년 ");
document.write((now.getMonth()+1) + "월 ");
document.write(now.getDate() + "일 ");

var array = ["일", "월", "화", "수", "목", "금", "토"];
var index = now.getDay();
document.write(array[index] + "요일");

[ex] var now = new Date(2019, 0, 1); //2019년 1월 1일 (month + 1) 이렇게 날짜 지정도 가능.
```

