# Bool 변수 이름 제대로 짓기

> 출처 : https://soojin.ro/blog/naming-boolean-variables

Bool 변수 작명을 위해 알아야 하는 영문법

- is 용법
- 조동사 용법
- has 용법
- 동사원형 용법

<br/>

## is 용법

is가 가장 흔하다. 뒤에 나오는 단어의 특징에 따라 세가지로 나눌 수 있다.

- is + 명사
- is + 현재 진행형(~ing)
- is + 형용사

### is + 명사

"(무엇)인가?" 라는 뜻으로 쓰인다.

```ts
declare function isDescendant(...args: any[]): boolean; // 자식 판별 함수. 자식 인가?
```

### is + 현재진행형(~ing)

"~하는 중인가?" 라는 뜻이 필요할 때 쓴다.

```ts
let isExecuting: boolean; // 작업이 실행 중인가?
let isPending: boolean; // 작업이 대기 중인가?
```

### is + 형용사

이제 헷갈릴 수 있다. 형용사도 두 종류로 나뉜다.

- 단어 자체가 형용사인 것 - opaque, readable, visible 등
- 과거분사 형태 -hidden, selected, highlighted, completed 등

과거 분사(past participle)는 간단히 말해 동사로 만든 형용사다. 동사를 과거 분사로 바꾸면 뜻이 여러가지로 바뀔 수 있는데, 일단 여기서는 수동태라고 생각하면 되겠다.

- hide(숨다) - hidden(숨겨진)
- select(선택하다) - selected(선택된)
- complete(완료하다) - completed(완료된)

보다시피 동사 뒤에 `-ed`가 붙는 형태가 가장 흔하지만, 내가 쓰려는 동사의 과거분사를 모르겠다면 사전을 찾아야 한다.

```ts
let isOpaque: boolean;
let isSelected: boolean;
let isHighlighted: boolean;
let isHidden: boolean;
```

📌 is로 시작하는 변수명을 짓다가 범하는 흔한 실수가 바로 `is + 동사원형`을 쓰는 것이다. -> 절대 쓰면 안된다.

isAuthorize, isHide, isFind 등등.

가령 `let isEdit: boolean;` 코드가 있는데, edit이라는 단어가 명사로 쓰일 수도 있어 해석의 여지는 있지만, 뜻이 명확하지 않아 일반적으로는 곧바로 해석하기 쉽지 않다. 바꿔보자. 아래와 같이 적절하게 바꿔주면 해석이 더 쉽고 빠르다.

```ts
let isEditable: boolean;
let isEditing: boolean;
let canEdit: boolean;
```

<br/>

## 조동사 용법

조동사(modal verb)는 동사를 돕는 동사란 뜻인데, can, should, will 등이 있다. 주의 할 점은 **조동사 + 동사원형**으로 사용해야 한다는 것이다.

- can : '~할 수 있는가?'
- should, will: '~해야 하는가?' 혹은 '~할 것인가?'

```ts
let canBecomeFirstResponder: boolean; // first responder가 될 수 있는가?
let shouldRefreshRefetchedObjects: boolean; // 가져온 값을 refresh 할 것인가?
```

<br/>

## has 용법

has로 시작하는 boolean 변수명은 상대적으로 빈도가 낮지만 뜻이 전혀 다르게 쓰이는 두가지가 있어 알아두면 유용하다.

- has + 명사
- has + 과거분사

### has + 명사

has 다음 명사가 나오면 '~를 가지고 있는가?' 라는 뜻이다. has는 have의 3인칭 단수인데, 3인칭 단수에 대해서 다음 파트에서 알아보자.

```ts
let hasiCloudAccount: boolean; // iCloud 계정을 가지고 있는가?
let hasVideo: boolean; // 비디오가 포함되어 있는가?
```

### has + 과거분사

모든 케이스를 통틀어 가장 덜 쓰이는 케이스. 이해 안간다면 넘어가자. 게다가 is + 과거분사와 뜻이 거의 같기 때문에 꼭 알아야할 필요도 없음.

이때의 과거분사는 아까 **is + 과거분사** 때와는 다른 의미다. has + 과거분사는 *현재완료*의 뜻을 가지는데 굳이 해석하면, '과거에 완료된 것이 현재까지 유지되고 있다'는 뜻이다. 따라서 boolean 변수로 쓰이면 '~가 유지되고 있는가?' 라고 해석된다.

```ts
let hasConnected: boolean; // 콜이 연결되어 있는가?
let hasEnded: boolean; // 콜이 끝났는가?
```

근데 isConnected, isEnded로 해도 의미가 비슷.

<br/>

## 동사원형 용법

boolean 변수짓기 끝판왕. 이것까지 잘 쓰면 원어민 개발자임. 주의할 점은 동사원형을 **3인칭 단수**로 써야한다. 3인칭 단수는 보통 동사원형 뒤에 -s나 -es가 붙는 형태다.

- supports: ~를 지원하는가?
- includes: ~를 포함하는가?
- shows: ~를 보여줄 것인가?
- allows: ~를 허용할 것인가?
- accepts: ~를 받아 주는가?
- contains: ~를 포함하고 있는가?

```ts
let supportsVideo: boolean; // 비디오를 지원하는가?
let includesCallsInRecents: boolean;
let showsBackgroundLocationIndicator: boolean;
let allowsEditing: boolean; // 편집을 허용하는가?
let acceptsFirstResponder: boolean;
```

### 3인칭 단수가 중요한 이유

코드 한 줄을 하나의 문장으로 비유하면 주어 역할을 하는 인스턴스가 3인칭 단수이기 때문에 문법적으로 꼭 써야하는 이유도 있지만, 3인칭 단수로 쓰지 않을 경우  [스위프트 API 디자인 가이드](https://swift.org/documentation/api-design-guidelines/#strive-for-fluent-usage)와의 일관성이 깨져서 코드를 읽는 사람을 혼란에 빠뜨릴 수 있다고 한다.

---

정리하자면

- is 용법
  - is + 명사
  - is + 현재진행(~ing)
  - is + 형용사
  - **~~is + 동사원형~~ (절대 쓰면 안됨)**
- 조동사 용법
  - can, should, will 등
  - **조동사 + 동사원형**
- has 용법
  - has + 명사
  - has + 과거분사 (is + 과거분사와 의미 거의 동일)
- 동사원형 용법
  - **3인칭 단수**