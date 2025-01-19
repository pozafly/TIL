# Arrow Function

> 출처: https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/JavaScript#arrow-function

화살표 함수 표현식은 기존의 function 표현 방식보다 간결하게 함. 화살표 함수는 항상 익명이고, 자신의 this, arguments, super 그리고 new.target을 바인딩하지 않는다. 따라서 생성자로는 사용할 수 없음.

- 화살표 함수 도입 영향: 짧은 함수, 상위 스코프 this

## 짧은 함수

```javascript
var materials = [
  'Hydrogen',
  'HHelium',
  'Lithium',
  'Beryllim',
];

material.map(function(material) {
  return material.length;
});   // [8, 6, 7, 9]

material.map((marterial) => {
  return material.length;
});   // [8, 6, 7, 9]

material.map(({length}) => length);   // [8, 6, 7, 9]
```

기존의 function을 생략 후 => 대체하여 표현

<br/>

## 상위 스코프 this

```javascript
function Person() {
  this.age = 0;
  
  setInterval(() => {
    this.age++;   // [this]는 person 객체를 참조
  }, 1000);
}

var p = new Person();
```

일반 함수에서 this는 자기 자신을 this로 정의한다. 하지만 화살표 함수 this는 Person 의 this와 동일한 값을 가짐. setInterval로 전달된 this는 Person의 this 를 가리키며, Person 객체의 age에 접근한다.

<br/>

> `정리`
>
> 화살표 함수는 가독성을 좋게, 그리고 this의 범위를 더 쉽게 늘려준다. 가끔 헷갈릴 때가 있지만 self = this; 이렇게 this를 변수화 해서 스코프 내에서 쓰는 것 보다 훨씬 가독성이 좋아진다. 물론 bind 함수를 사용할 수도 있지만 이게 훨씬 깔끔하게 보임.
