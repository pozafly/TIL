# Intellij 템플릿(단축어) 사용

Intellij에서 템플릿 기능을 사용하면 snippets를 사용할 수 있음.

![[assets/images/955710d51797b69c2810fb7ebf374aca_MD5.png]]

세팅에서 라이브 템플릿을 찾음. `+` 버튼을 누른다.

![[assets/images/119a8213f7035c3ce2afa6029ae8da39_MD5.png]]

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

![[assets/images/ee3c4d7846ac7eb384feb026e8e752a1_MD5.png]]

적용 가능한 컨텍스트 밑 정의를 누르고,

![[assets/images/69e3503993f3c107df9080492c47c9c0_MD5.png]]

이렇게 Java를 선택해주면 된다.

이제 편집기에서 [약어] + tab 키를 누르면 자동 완성이 된다.
