# Java, JavaScript 타입 비교

둘다 기본형, 참조형 존재함.

기본형은 값 자체를 저장하고, 참조형은 주소 값을 저장한다.

## 기본형과 참조형

### 기본형

```javascript
// js
let a = 10;
let b = a; // 값이 복사됨
b = 20; // b 만 변경됨

console.log(a);  // 10
console.log(b);  // 20
```

```java
// java
int a = 10;
int b = a;  // 값이 복사됨
b = 20;

System.out.println(a);  // 10
System.out.println(b);  // 20
```

기본형 데이터는 값을 수정해도 서로 독립적 존재이다.

### 참조형

둘 다 주소를 복사하기 때문에, 하나를 수정하면 다른 변수도 영향을 받는다. '같은 객체를 참조한다'.

```javascript
// js
let obj1 = { name: 'Alice' };
let obj2 = obj1;  // obj1의 주소가 obj2에 복사됨 (같은 객체를 가리킴)
obj2.name = 'Bob';

console.log(obj1.name);  // Bob
console.log(obj2.name);  // Bob
```

```java
// java
class Person {
    String name;
}

Person p1 = new Person();
p1.name = "Alice";
Person p2 = p1;  // p1의 주소를 p2에 복사
p2.name = "Bob";

System.out.println(p1.name);  // Bob
System.out.println(p2.name);  // Bob
```

<br/>

## String을 다루는 차이

- JavaScript
  - String은 **기본형**
  - 값을 복사하기 때문에 불변성이 존재하지 않는다.
- Java
  - String은 **참조형**
  - 불변 객체(Immutable)이기 때문에 새로운 객체가 힙에 생성된다. (기존 객체 그대로 유지하고 새로운 객체를 참조함.)

```java
// js
let s1 = "Hello";
let s2 = s1;
s2 = "World";  // 새로운 값이 s2에 할당됨

console.log(s1);  // Hello (s1은 변하지 않음)
console.log(s2);  // World
```

```java
// java
String s1 = "Hello";
String s2 = s1;
s2 = "World";  // 새로운 String 객체가 생성됨

System.out.println(s1);  // Hello
System.out.println(s2);  // World
```

> 여기서 불변(Immutable) 이란, 변하지 않는 것이다. 즉, 새로운 값이 할당 되고 기존 값은 그대로 유지된다.
>
> **JavaScript**에서는 불변 **객체**란 존재하지 않는다. 객체는 모두 가변이다. 객체 안의 내용은 언제든 변할 수 있기 때문이다. 기본형은 불변이지만 참조형은 가변이다. 하지만, **Java**에서는 String만 예외적으로 불변 객체이다. 새로운 값을 만들어 새로운 주소를 넣기 때문이다.
>
> ※ 단, JavaScript에서 불변 객체를 임의적으로 만들려면 `Object.freeze()` 를 사용해 만들수는 있음.
>
> JavaScript 특히 react에서 말하는 불변성을 지킨다는 것은, 기존 객체를 버리고 새로운 객체를 만들어주는 것인데 spread 문법 등을 이용해서 만들어줄 수 있다.

즉,

- JavaScript에서는 String이 **기본형(Primitive Type)**으로 다뤄지고, **불변(Immutable)** 특성을 가짐.
- Java에서는 String이 **참조형(Reference Type)**이지만 **불변(Immutable)** 객체.

따라서 Java에서는 String 객체를 수정하는 것처럼 보여도 한 번 생성되는 절대 수정 불가능하다. 기존 객체가 남아있다. JavaScript에서는 기본형이기 때문에 String은 새로운 값을 새로 생성해서 새로 넣는다.

### 🔸 Java String 생성 동작 방식(메모리 흐름)

1. "Hello"라는 String 객체가 **String Pool**(힙 메모리의 특정 공간)에 저장됨.
2. s1과 s2는 같은 "Hello" 객체를 참조함.
3. s2 = "World";를 실행하면,
   - **새로운 String 객체 "World"**가 생성됨.
   - s2는 새로운 "World" 객체를 참조하고,
   - s1은 여전히 "Hello" 객체를 참조함.

### 🔸 JavaScript String 생성 동작 방식(메모리 흐름)

1. s1에 "Hello"라는 값이 **스택(Stack) 메모리**에 저장됨.
2. s2 = s1에서 "Hello"라는 **값 자체가 복사**되어 s2에 저장됨.
3. s2 = "World"에서 **새로운 값 "World"**가 s2에 저장됨.
4. s1은 여전히 "Hello" 값을 가지고 있음.

<br/>

## Java에서 String이 불변(Immutable) 참조형인 이유

이유는 **안정성**과 **성능 최적화** 때문이다.

### 1. 보안

String이 불변이면 수정이 불가능하므로 보안상 안전하다.

```java
String url = "https://example.com";
url.replace("https", "http");
System.out.println(url);  // https://example.com (수정되지 않음)
```

가변이라면, 중요한 정보가 의도치 않게 수정될 수 있으므로 **해킹이나 악의적인 수정**을 방지할 수 있음.

### 2. 스레드 안정성

String이 불변하면 여러 스레드에서 동시에 사용해도 안전하다.

```java
String shared = "data";

// 여러 스레드에서 동시에 사용
Thread t1 = new Thread(() -> System.out.println(shared + " A"));
Thread t2 = new Thread(() -> System.out.println(shared + " B"));

t1.start();
t2.start();
```

String이 가변이었다면, **경합 상태(Race Condition)**가 발생해 **데이터 오염**이 발생할 수 있음.

### 3. 캐싱(String Pool)으로 성능 최적화

```java
String s1 = "Hello";
String s2 = "Hello";
```

"Hello"라는 문자열은 **String Pool**에 저장.

- s1과 s2는 **동일한** "Hello" **객체를 참조**하므로 메모리가 절약.
- 만약 String이 **가변(muttable)**이라면, s1 또는 s2 중 하나를 수정할 때 다른 참조에도 영향을 줘서 문제 발생.

### 4. 해시값(Hashcode) 캐싱

Java에서 String은 **해시값(Hashcode)을 캐싱**함.

- 불변이기 때문에, 한 번 계산된 해시값을 **재사용** 가능.
- 해시값이 자주 사용되는 **Map, Set**에서 성능이 크게 향상.

### 5. 디자인의 일관성

- Java에서는 **모든 참조형(객체)**이 힙(Heap) 메모리에 저장.
- String을 **참조형으로 통일**함으로써, **객체지향 언어로서의 일관성**을 유지하고, **다양한 메서드와 기능(예: substring(), concat())**을 제공함.
