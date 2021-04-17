# Variable

- 변수(let) : rw(read / write) 가능함.

- 상수(const) : r(read)만 가능.



## 1. var를 쓰지 말아야 할 이유

- hoisting
- no block scope -> 블록 스코프와 관계없이 실행됨.

<br/>

## 2. let(Mutable)

- var 대신 나온 변수. 값을 재할당할 수 있고 호이스팅이 일어나지 않음.

<br/>

## 3. const(Immutable)

- security : 할당 후 절대 값 변경 불가. 따라서 해커들도 이 값을 변경할 수 없음.
- thread safety : 실행 환경에서 쓰레드가 여러 개의 변수를 조작하게 되는데 자원을 아낄 수 있음.
- reduce human mistakes : 협업 시 코드를 바꿀 수 없도록.

<br/>

## 4. Types

primitive 타입과 object 타입으로 나누어짐. primitive는 메모리에 바로 적재가 되고, object 타입은 메모리에 ref(object를 가리키고 있는 것 -> 포인터)가 적재되고 안에 있는 각각의 item들은 또 다른 메모리에 적재 됨. 따라서 만약 object를 const로 선언한다면 ref가 상수화 되는 것이고 안의 내용은 다른 메모리에 적재되므로 `변경이 가능`하다.

- primitive, single item
  - number
    - Infinity : 1 / 0
    - NegativeInfinity = -1 / 0
    - NaN(not a number) : string / 2
  - string
  - boolean
    - false : 0, null, undefined, NaN, ''
    - true : any other value
  - null
  - undefined
    - 선언은 되었지만 값이 할당되지 않음. null인지 값이 지정될 것인지 아직 지정되지 않음.
  - symbol
    - map 다른 자료에서 우선순위를 주고 싶을 때 사용함.
- object, box container(single item을 하나로 묶어서 관리하는 타입)
  - function
  - first-class function : function도 하나의 데이터 타입처럼 변수에 할당 된다는 말임. 인자로 전달 가능, return도 됨.

<br/>

## 5. Dynamic typing

선언할 때, 타입을 결정하지 않고 run time(프로그램 동작) 시 할당된 타입에 따라 타입이 결정됨.



>`정리`
>
>변수.. 용어 정리 했네. symbol 은 아직 모르겠음. const로 객체를 저장 시 안의 내용을 바꿀 수 있다는 사실은 처음 알게 되었는데 포인터 개념까지 같이 들어가 있다니. 재미있군.

