# Class(OOP)

> 출처: https://youtu.be/_DLhUBWsRtw

- class: 연관있는 Data를 한데 묶어놓음. (붕어빵 틀)

  ```javascript
  class person {
    name;
    age;
    speak();
  }
  ```

  - name, age: 속성(field)
  - speak(): 행동(method)
  - class는 ES6에서 새롭게 추가된 것임. prototype-based 임.
- object: 실제 Data를 넣어 만드는 것. (붕어빵)

<br/>

## 예제

```javascript
// 1. class 선언
class person {
  // 생성자
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  // 메서드
  speak() {
    console.log(`${this.name}: hello!`);
  }
}

// 2. Object 생성
const ellie = new Person('ellie', 20);
console.log(ellie.name);  // ellli
console.log(ellie.age);   // 20
ellie.speak();   // ellie: hello!
```

<br/>

## Getter and Setter

```javascript
class User {
  constructor(firstName, lastName, age) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;  
  }
  
  get age() {  // (1)
    return this.age;
  }
  
  set age(value) {  // (2)
    this.age = value;
  }
}

const user1 = new User('Steve', 'Job', -1);
console.log(user1.age);
```

- this.age: 메모리에 올라가 있는 age를 가져오는 것이 아니라, (1)get 함수를 불러옴
- = age;: (2)set 함수를 불러옴. = value; 도 마찬가지로 setter를 호출함.
- 따라서 `Uncaught RangeError: Maxinum call stack size exceeded` 라는 오류를 뱉음

```javascript
get age() {
  return this._age;
}

set age(value) {
  this._age = value;
}
```

변수 이름을 바꿔주어야 함. 다시 보면,

```javascript
class User {
  constructor(firstName, lastName, age) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;  
  }
  
  get age() {  // (1)
    return this._age;
  }
  
  set age(value) {  // (2)
    // if (value < 0) {
    //  throw Error('age can not be negative');
    // }
    this._age = value < 0 ? 0 : value;
  }
}

const user1 = new User('Steve', 'Job', -1);
console.log(user1.age);
```

따라서, user1.age를 사용할 때는 내부적으로 getter, setter를 이용하기 때문임.

<br/>

## Public & Private

```javascript
class Experiment {
  publicField = 2;
  #privateField = 0;
}
const experiment = new Experiment();
console.log(experiment.publicField);   // 2
console.log(experiment.privateField);  // undefined
```

- 최근에 추가된 스펙임. 사파리에서도 아직 지원하지 않음.
- `#` 을 붙여주면 외부에서 변경할 수도, 읽어올 수도 없음. 오직 class 내부에서만 사용가능한 private 변수.

<br/>

## Static

```javascript
class Article {
  static publisher = 'Dream Coding';
  constructor(articleNumber) {
    this.articleNumber = articleNumber;
  }
  
  static printPublisher() {
    console.log(Article.publisher);
  }
}
```

- 이녀석도 최근에 만들어짐.
- Class로 인해 각기 다른 Object가 만들어졌다고 했을 때, Object와는 상관없는 Class만의 지정된 값이나 메서드가 필요할 때는 static으로 선언해 줄 수 있다.

```javascript
const article1 = new Article(1);
const article2 = new Article(2);
console.log(article1.publisher);  // undefined
```

즉 여기서 봤을 때는 값이 지정되지 않았다는 것인데,

```javascript
console.log(Article.publisher);   // Dream Coding
Article.publisher();  // Dream Coding
```

Article 이라는 class에 지정되어 있기 때문에 Object로 호출이 아니라 Class 자체로 호출 해주어야 함.

<br/>

## 상속 & 다형성

```javascript
class Shape {
  constructor(width, height, color) {
    this.width = width;
    this.height = height;
    this.color = color;
  }
  draw() {
    console.log(`drawing ${this.color} color of`);
  }
  getArea() {
    return width * this.height;
  }
}

class Rectangle extends Shape {}
class Triangle extends Shape {
  // 오버라이딩
  draw() {
    super.draw();
    console.log('🔺')
  }
  getArea() {
    return (this.width * this.height) / 2;
  }
}

const rectangle = new Rectangle(20, 20, 'blue');
rectangle.draw();   // drawing blue color of
console.log(rectangle.getArea());  // 400

const triangle = new Triangle(20, 20, 'red');
triangle.drqw();   // drawing blue color of 한줄 띄우고 🔺
console.log(triangle.getArea());  // 200
```

핵심은 extends 다. Shap이라는 class를 Rectangle 에서 상속받아 사용한것. 오버라이딩 해서 사용 가능. java의 상속, 다형성과 똑같음.

<br/>

## instanceOf

```javascript
console.log(rectangle instanceof Rectangle); // true
console.log(triangle instanceof Rectangle);  // false
console.log(triangle instanceof Triangle);   // true
console.log(triangle instanceof Shape);      // true
console.log(triangle instanceof Object);     // true
```

- 하위객체 instanceof 상위객체. -> 하위객체는 상위객체의 instance인가? (즉, 하위객체는 상위객체에 의해서 만들어진 상속된 객체인가?)
- boolean 값 리턴함.

<br/>

> `정리`
>
> OOP의 개념은 Java를 공부하면서 많이 배웠던 내용이다. Javascript 버전으로 보니 개념적인 부분은 같아서 재미있게 봤음.
>
> 이동욱 님께서 유튜브에서 RxJava? 가 Javascript를 따라하는 부분이 생기고, Javascript가 Java를 따라하는 부분이 생긴다고 했다. 서로 좋은 부분은 가져다 쓰면서 언어 자체가 진화하는. 그런 느낌이 아닐까? public, private & static. 새로 생기는 개념들.
>
> 신기.
