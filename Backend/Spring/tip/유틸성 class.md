# class

```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public final class PageLimitCalculator {

}
```

- `@NoArgsConstructor(access = AccessLevel.PRIVATE)`
	- 목적: 인스턴스 생성을 막기 위함
	- java는 생성자가 하나도 없으면 컴파일러가 기본 생성자를 public으로 생성함.
	- 명시적으로 private생성자를 만들어줘야 `new` 키워드로 인스턴스를 만들 수 없음.
- `final`
	- 목적: 상속을 막기 위해
	- 유틸 클래스는 상속을 위한 클래스가 아니고, 정적 메서드만 제공하는 용도임.
	- 따라서 불필요한 상속을 막기 위해 final 키워드를 붙임.

## 예시

```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public final class PageLimitCalculator {
    public static int calculateLimit(int page, int size) {
        return page * size;
    }
}
```

- 인스턴스화 불가능 (`new PageLimitCalculator()` 금지)
- 상속 불가능 (`extends PageLimitCalculator` 금지)
- static method만 주로 사용함.
