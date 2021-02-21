# Class vs Object(OOP)

> 출처 : https://youtu.be/_DLhUBWsRtw

- class : 연관있는 Data를 한데 묶어놓음. (붕어빵 틀)

  ```javascript
  class person {
    name;
    age;
    speak();
  }
  ```

  - name, age : 속성(field)
  - speak() : 행동(method)
  - class는 ES6에서 새롭게 추가된 것임. prototype-based 임.

- object : 실제 Data를 넣어 만드는 것. (붕어빵)

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

- this.age : 메모리에 올라가 있는 age를 가져오는 것이 아니라, (1)get 함수를 불러옴
- = age; : (2)set 함수를 불러옴. = value; 도 마찬가지로 setter를 호출함.
- 따라서 `Uncaught RangeError : Maxinum call stack size exceeded` 라는 오류를 뱉음

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

