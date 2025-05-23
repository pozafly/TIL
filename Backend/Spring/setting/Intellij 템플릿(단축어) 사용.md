# Intellij 템플릿(단축어) 사용

Intellij에서 템플릿 기능을 사용하면 snippets를 사용할 수 있음.

<img width="862" alt="Image" src="https://github.com/user-attachments/assets/ca92e38b-7629-43f0-b3ef-a56f9e25a19f" />

세팅에서 라이브 템플릿을 찾음. `+` 버튼을 누른다.

<img width="991" alt="Image" src="https://github.com/user-attachments/assets/e35c89ca-af81-4d9e-8e74-91096741ed12" />

템플릿 그룹을 먼저 하나 만들어주고, 이름(약어)을 만들어준다.

그리고 거기에 라이브 템플릿을 다시 하나 만든다.

```java
@Test
@DisplayName("$NAME$")
void $NAME2$() {
    // given
    $END$
    // when
    
    // then
    
}
```

이런식으로 코드를 넣어준다.

<img width="531" alt="Image" src="https://github.com/user-attachments/assets/b622fe13-53e0-41c6-bdae-7400834663ee" />

적용 가능한 컨텍스트 밑 정의를 누르고,

<img width="1009" alt="Image" src="https://github.com/user-attachments/assets/1e580008-a64e-4cea-b315-d71226dbe97d" />

이렇게 Java를 선택해주면 된다.

이제 편집기에서 [약어] + tab 키를 누르면 자동 완성이 된다.
