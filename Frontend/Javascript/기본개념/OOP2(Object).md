# Object(OOP)

> 출처: https://youtu.be/1Lbr29tzAA8

<br/>

## 1. Literals and properites

```javascript
function print(person) {
  console.log(person.name);
  console.log(person.age);
}

const hst = { name: 'hst', age: 31 };
print(hst);

hst.hasJob = true;  // 여기 마지막에 이렇게 프로퍼티를 추가할 수 있음.
console.log(hst.hasJob);  // true

delete hst.hasJob;
console.log(hst.hasJob);  // undefined
```

- 이렇게 class를 만들지 않고, 바로 Object를 만들 수 있다.

```javascript
const obj1 = {}; // 'object literal' syntax
const obj2 = new Object();  // 'object constructor' syntax
```

- Object = { key: value };

<br/>

## 2. Computed properties(계산된 properties)

- 동적으로 key를 구현할 때 사용할 수 있음.

```javascript
console.log(hst.name);     // hst
console.log(hst['name']);  // hst
hst['hasJob'] = true;
console.log(hst.hasJob);  // true
```

- Object.properties로 오브젝트의 값에 접근 가능하며
- Object['키 이름'] 으로도 값에 접근이 가능함. 단, 키 이름은 string 타입으로 커테이션을 주어 표현해야함.
- 여기서 `Computed properties`는 배열 형태를 말한다. 그럼 어떨 때 쓰는가?
  - hst.name: 코딩하는 그 순간 그냥 값을 가져오고 싶을 때,
  - hst['name']: 어떤 key가 필요한지 모를 때, 즉 runtime시 결정될 때 사용.

```javascript
function printValue(obj, key) {
  console.log(obj.key);   // undefined
  console.log(obj[key]);  // obj.name
}
printValue(ellie, 'name');
```

<br/>

## 3. Property value shorthand

```javascript
const person1 = { name: 'bob', age: 2 };
const person2 = { name: 'steve', age: 3 };
const person3 = { name: 'dave', age: 4 };

// 여기서 person4를 만들고 싶은데 일일이 치고 싶지 않을 때,
function makePerson(name, age) {
  return {
    name,   // name: name,
    age,    // age: age,
  };
};

const person4 = makePerson('hst', 30);
console.log(person4);   // {name: "ellie", age: 30}
```

<br/>

## 4. Constructor Function

위의 코드에서 function makePerson 을 고친다면,

```javascript
function Person(name, age) {
  // this = {};
  this.name = name;
  this.age = age;
  // return this;
}
```

이런식의 생성자 함수.

<br/>

## 5. In operator

간단하게 키가 있는지 없는지 판단함.

```javascript
console.log('name' in hst);  // true
console.log('age' in hst);   // true
console.log('random' in hst);   // false
console.log(hst.random);   // undefined
```

<br/>

## 6. For.. in vs for.. of

```javascript
for (key in hst) {
  console.log(key);   // name age hasJob
}
```

- for in은 key를 가지고 옴.

```javascript
const array = [1, 2, 3, 4, 5];
for (value of array) {
  console.log(value);   // 1 2 3 4 5
}
```

- for of 은 value를 가지고 옴.

<br/>

## 7. Cloning

```javascript
const user = { name: 'hst', age: '20' };
const user2 = user;

user2.name = 'coder';
console.log(user.name);   // coder
```

- user와 user2는 ref는 다르고 같은 값을 가리키고 있다.
- 여기서 user2의 name을 변화시킨다면 같은 값을 가리키고 있는 user, user2 모두 바뀌는 것임.

이 개념은 복사가 아니다. 이제 복사하고 싶으면 어떻게 해야하나?

````javascript
// old way
const user3 = {};
for (key in user) {
  user3[key] = user[key];
}
````

```javascript
// new way
const user4 = {};
Object.assign(user4, user);

또는

const user4 = Object.assign({}, user);
```

```javascript
// another exam
const fruit1 = { color: 'red' };
const fruit2 = { color: 'blue', size: 'big' };
const mixed = Object.assign({}, fruit1, fruit2);
console.log(mixed.color);   // blue
console.log(mixed.size);    // big
```

- 뒤에 나오는 녀석(fruit2)이 앞(fruit1)에 나오는 녀석을 덮어쓴다.

<br/>

> `정리`
>
> Computed properties는 Vue의 watch가 생각난다. 동적으로 변하는 key의 형태를 string으로 넘기는데, 이 때 내부적으로 vue에서 Computed properties를 사용하지 않을까?
>
> 아직 javascript에서는 class 보다 Object 형태를 많이 쓰는데, 나중에 typeScript에서 class 형태가 많다고 한다. 자바스크립트에서 주로 Object만 다루었지 class 를 보니 신기하긴 하네.
