# 선택자 &: vs &:: vs &.

> [출처](https://velog.io/@eunjeong/TIL-SCSS%EC%84%A0%ED%83%9D%EC%9E%90-vs-vs-)

우선 scss에서 `&` 란, 상위(부모) 선택자를 참조하여 치환함.

scss에서 선택자 중 `&:`, `&::`, `&.` 이 세가지를 언제 사용하는지에 대해서 알아보자. 아래는 예시 코드다.

```scss
input {
  &::placeholder {
    color: #fff;
  }
}

button {
  &:hover {
    boackground: #fff;
  }
}

.some {
  &.checked {
    svg {
      color: #fff;
    }
  }
}
```

<br/>

<br/>

## &:

&: 은 :hover, :checked, :not(&) 등의 `가상클래스` 또는 :after, :before 등의 `가상요소` 에 사용된다.

```scss
button {
  background: none;
  &:hover {
    background: #fff;
  }
}
```

<br/>

## &::

&::는 아래와 같이 input 태그의 `placeholder` 의 스타일을 변경할 수 있다.

```scss
input::placeholder {
  background: #fff;
}
```

<br/>

## &.

&. 은 `특정한 클래스의 스타일을 변경` 한다. 대부분의 속성이 같지만 몇가지가 다르다면 아래와 위의 예제와 같이 사용한다.

```scss
.checkbox {
  cursor: pointer;
  flex: 1;
  display: flex;
  // checked 클래스가 있다면 스타일 적용
  &.checked {
    svg {
      color: #fff;
    }
  }
}
```

