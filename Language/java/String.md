# String

java에서 String 변수를 선언햘때, 다른 것은 int, boolean 등 소문자로 시작하지만, 얘만 대문자로 시작함.

이유는 불변 `객체` 이기 때문이다.
```java
public class StringBasicMain {
  public static void main(String[] args) {
    String str1 = "hello";
    String str2 = new String("hello");
  }
}
```
객체이기 때문에 참조형임. 하지만, `new String()` 을 하지 않아도 리터럴을 넣어도 된다.

문자열은 자주 사용하기 때문에 자바 언어 측면에서 변경해주는 것임.

## 클래스 구조
```java
public final class String {
  // 문자열 보관
  private final char[] value; // 자바 9 이전
  private final byte[] value; // 자바 9 이후
  
  // 여러 메서드
  public String concat(String str) { ... }
  ...
}
```
**속성(필드)**
```java
private final char[] value;
```
String의 실제 문자열 값이 보관 됨. 데이터 자체는 `char[]` 에 보관. String 클래스는 개발자가 직접 다루기 불편한 `char[]` 를 내부에 감추고 `String` 클래스를 사용하는 개발자가 편리하게 문자열을 다룰 수 있도록 다양한 기능을 제공.

> 자바 9 이후 String 클래스는 `char[]` 대신 `byte[]` 를 사용함.
>
> 문자 하나를 표현하는 `char` 는 `2byte` 임. 하지만, 영어, 숫자는 `1byte` 로 표현이 가능. 그래서 단순 영어, 숫자로만 표현된 경우 `1byte` 를 사용하고 그렇지 않은 나머지 경우 `2byte` 인 UTF-16 인코딩을 사용함. 따라서 메모리를 더욱 효율적으로 사용할 수 있게 되었음.

<br/>

## String 클래스와 참조형

`String` 은 클래스이자 참조형임. 그래서 `+` 기호를 사용할 수 없다.
```java
String a = "hello";
String b = " java";

String result1 = a.concat(b);  // hello java
String result2 = a + b;  // hello java
```
하지만 문자열은 특수 class 이므로 java에서 이를 허용해줌.

<br/>

## String 클래스 - 비교

항상 `==` 비교가 아니라 `equals()` 비교를 해야 함.

- 동일성 (Identity): `==` 연산자를 사용해 두 객체의 참조가 동일한 객체를 가리키고 있는지 확인.
- 동등성 (Equality): `equals()` 메서드 사용해 두 객체가 논리적으로 같은지 확인.
```java
String str1 = new String("hello");
String str2 = new String("hello");
System.out.println(str1 == str2);  // false
System.out.println(str1.equals(str2));  // true

String str3 = "hello";
String str4 = "hello";
System.out.println(str3 == str4);  // true
System.out.println(str3.equals(str4));  // true
```
`new String()` 으로 만든 문자열은 `==` 비교시 참조값을 비교함. 하지만 리터럴은 그냥 값 자체를 비교해줌. 이유는 마찬가지로 String은 java에서 특별 취급이기 때문.

![[assets/images/ab202158c7741bdaaafe582a099904f6_MD5.png]]

이건 `new String()` 을 사용했을 경우 메모리 구조임. 하지만 리터럴과 같은 경우 String pool(문자열 풀)로 관리된다.

![[assets/images/0f46edd5a69f954d33978e170a0e2689_MD5.png]]

문자열 리터럴을 봤을 때, 어플리케이션이 실행되자마자 모든 리터럴을 문자열 풀에 넣어두고, 같은 값을 참조하여 사용하게끔 한다. 메모리 효율성과 성능 최적화를 위하기 때문임.

> 참고
>
> 프로그래밍에서 풀(Pool)은 자원이 모여있는 곳을 의미함. 공용 자원을 모아둔 곳을 말한ㄷ. 여러 곳에서 함께 사용할 수 있는 객체를 필요할 때마다 생성하고, 제거하는 것은 비효율적임. 대신 이렇게 문자열 풀에 필요한 `String` 인스턴스를 미리 만들어두고 여러 곳에서 재사용할 수 있다면 성능과 메모리를 더 최적화할 수 있다.
>
> 참고로 문자열 풀은 힙 영역을 사용한다. 그리고 문자열 풀에서 문자를 찾을 때는 해시 알고리즘을 사용하기 때문에 매우 빠른 속도로 원하는 `String` 인스턴스를 찾을 수 있음.

그러면 `==` 써도 되나?? 아님. `equals()` 를 그냥 사용해라.
```java
public static void main(String[] args) {
  String str1 = new String("hello");
  String str2 = new String("hello");
  System.out.println(isSame(str1, str2)); // false
  
  String str3 = "hello";
  String str4 = "hello";
  System.out.println(isSame(str3, str4)); // true
}

public ststic boolean isSame(String x, String y) {
  return x == y;
}
```
`new String()` 으로 생성했는지 리터럴로 생성했는지 `isSame()` 메서드에서는 알 길이 없음. 따라서 서로 다른 결과를 내뱉는데, 그래서 그냥 `equals()` 쓰는게 낫다.

<br/>

## 불변

문자열 풀에 있는 String 인스턴스의 값이 중간에 변경되면 같은 문자열을 참고하는 다른 변수의 값도 함께 변경됨.
```java
String str1 = 'hello';
str1 = 'java';
```
이렇게 하면 hello의 값 자체가 아예 사라지고 java로 대체되는 것임. 그런데 만약 str1 을 다른 곳에서 사용하고 있었을 경우, 다른 쪽에서도 변경되기 때문에 안전하게 불변으로 둔 것임.

<br/>

## StringBuilder - 가변 String

불변인 String의 단점은 비효율성에 있음.
```java
String str = "A" + "B" + "C" + "D";
```
이거는 java에서 아래와 같이 특수 처리된다.
```java
String str = new String("A") + new String("B") + new String("C") + new String("D");
String str = new String("AB") + new String("C") + new String("D");
String str = new String("ABC") + new String("D");
String str = new String("ABCD");
```
이 경우 총 3개의 `String` 클래스가 추가로 생성된다. 문제는 중간에 만들어진 `new String("AB")`, `new String("ABC")` 는 사용되지 않고 최종적으로 만들어진 `new String("ABCD")` 만 사용된다. 이후 중간에 만들어진 녀석들은 GC 대상이 된다.

불변인 `String` 클래스는 문자를 더하거나 변경할 때마다 계속 새로운 객체를 생성해야 한다는 점.

> 참고로 실제로 문자열을 다룰 때 자바가 내부에서 최적화를 적용하긴 함.

### StringBuilder

StringBuilder는 불변이 아닌 가변임. `final` 을 사용하지 않음.
```java
StringBuilder sb = new StringBuilder();
sb.append("A");
sb.append("B");
sb.append("C");
sb.append("D");
Systme.out.println(sb); // ABCD

sb.insert(4, "java");
Systme.out.println(sb); // ABCDjava

sb.delete(4, 8);
Systme.out.println(sb); // ABCD

sb.reverse();
Systme.out.println(sb); // DCBA

String value = sb.toString(); // 실제 사용할 값
```
> 가변(Mutable) vs 불변(Immutable)
>
> - `String` 은 불변. 한 번 생성되면 그 내용을 변경할 수 없음. 따라서 문자열에 변화를 주려고 할 때마다 새로운 String 객체가 생성, 기존 객체는 버려짐. 이 과정에서 메모리와 처리 시간을 더 많이 소모.
> - `StringBuilder` 는 가변. 추가, 삭제, 수정할 수 있고, 새로운 객체를 생성하지 않음. 이로 인해 메모리 사용을 줄이고 성능을 향상시킬 수 있음. 단 사이드 이펙트를 주의해야 함.

### String 최적화

#### 문자열 리터럴 최적화

컴파일 전
```java
String helloWorld = "Hello, " + "World!";
```
컴파일 후
```java
String helloWorld = "Hello, World!";
```
리터럴은 평가 시점에 걍 합쳐버림

#### String 변수 최적화

변수의 경우 어떤 값이 들어있는지 컴파일 시점엔 알 수 없으므로 단순하게 합칠 수 없음. 이 때는 java가 알아서 StringBuilder를 사용해 합침

컴파일 전
```java
String result = str1 + str2;
```
컴파일 후
```java
String result = new StringBuilder().append(str1).append(str2).toString();
```
이렇게 자바가 최적화 해주기 때문에 간단한 경우 `StringBuilder` 를 직접 사용하지 않아도 된다.

#### 최적화가 어려운 경우
```java
public static void main(String[] args) {
    long startTime = System.currentTimeMillis();
    String result = "";
    for (int i = 0; i < 100000; i++) {
        result += "Hello Java ";
    }
    long endTime = System.currentTimeMillis();

    System.out.println("result = " + result);
    System.out.println("time = " + (endTime - startTime) + "ms");
}
```
`2570ms` 가 걸림. 오래 걸림. 이유는 반복문 안에서 런타임에 연결할 문자열의 개수와 내용이 결정되기 때문에 java가 예측할 수 없음. 최적화가 안된다. 아마도 대략 반복 횟수인 100,000 번의 `String` 객체를 생성했을 것이다.
```java
public static void main(String[] args) {
    long startTime = System.currentTimeMillis();
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 100000; i++) {
        sb.append("Hello Java ");
    }
    long endTime = System.currentTimeMillis();
    String result = sb.toString();

    System.out.println("result = " + result);
    System.out.println("time = " + (endTime - startTime) + "ms");
}
```
`StringBuilder` 를 직접 사용해주면, `3ms` 밖에 걸리지 않았음.

#### 정리

따라서 StringBuilder를 직접 사용하는 것이 더 좋은 경우

- 반복문에서 반복해서 문자를 연결할 때
- 조건문을 통해 동적으로 문자열을 조합할 때
- 복잡한 문자열의 특정 부분을 변경해야할 때
- 매우 긴 대용량 문자열을 다룰 때

> 참고: `StringBuilder` vs `StringBuffer`
>
> - `StringBuilder` 와 StringBuffer는 같은 기능을 수행. Buffer는 내부에 동가화가 되어 있어, 멀티 스레드 상황에 안전하지만 동기화 오버헤드로 인해 성능이 느리다.
> - Buffer는 멀티 쓰레드 상황에 안전하지 않지만 동기화 오버헤드가 없으므로 속도가 빠르다,
