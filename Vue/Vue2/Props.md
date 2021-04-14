# Props

> 출처 : https://kr.vuejs.org/v2/guide/components-props.html

<br/>

## Prop 대소문자 구분 (camelCase vs Kebab-case)

HTML 어트리뷰트는 대소문자 구분이 없기 때문에 브라우저는 대문자를 소문자로 변경하여 읽는다. 그렇기 때문에 카멜 케이스(대소문자 혼용)로 prop의 이름을 정한 경우에 DOM 템플릿 안에서는 케밥 케이스(하이픈으로-연결된-구조)를 사용해야 올바르게 동작한다.

```jsx
Vue.component('blog-post', {
  // Javascript에서의 camelCase
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
});

<!-- HTML에서의 kebab-case -->
<blog-post post-title="hello!"></blog-post>
```

<br/>

## Prop 타입

props를 문자열 배열 형태로 작성된 prop 리스트를 봐왔다.

```js
props: ['title', 'likes', 'isPublished', 'commentIds', 'author']
```

prop에 특정 타입의 값을 넣고 싶은 경우가 있을 수 있다. 이 때, 다음과 같이 prop을 속성 이름과 타입을 포함하는 오브젝트로 선언함으로써 타입이 지정된 prop의 리스트를 구현할 수 있다.

```js
props: {
 title: String,
 likes: Number,
 isPublished: Boolean,
 commentIds: Array,
 author: Object,
 callback: Function,
 contactsPromise: Promise  // or nay other consturctor
}
```

이는 컴포넌트를 읽기 좋게 문서화할 뿐 아니라 브라우저의 자바스크립트 콘솔에서도 잘못된 타입이 전달된 경우 경고를 띄워줄 수 있도록 해준다.

<br/>

## 정적 & 동적 prop 전달하기

```html
<blog-post title="My journey with vue"></blog-post>
```

혹은 아래와 같이 `v-bind` 를 이용해 동적인 prop을 전달할 수도 있다.

```html
<!-- 변수에 담긴 값을 동적으로 할당 -->
<blog-post v-bind:title="post.title"></blog-post>

<!-- 복잡한 표현식의 값을 동적으로 할당 -->
<blog-post
  v-bind:title="post.title + ' by ' + post.author.name"
></blog-post>
```

위 두 가지 경우는 모두 문자열 형태의 변수를 전달하지만 실제로는 모든 타입의 변수가 prop으로 전달될 수 있다.

### 논리 자료형(Boolean) 전달

```html
<!-- 값이 없는 prop은 `true` 를 전달. -->
<blog-post is-published></blog-post>

<!-- `false`는 정적인 값이지만, Vue에서 해당 값이 논리 자료형이라는 것을 알 수 있도록 하기 위해 -->
<!-- v-bind를 이용해 문자열이 아닌 JavaScript 표현식이라는 것을 알려줌. -->
<blog-post v-bind:is-published="false"></blog-post>

<!-- 변수의 값을 동적으로 할당할 수도 있다. -->
<blog-post v-bind:is-published="post.isPublished"></blog-post>
```

### 배열(Array) 전달

```html
<!-- 해당 배열은 정적인 값이지만, Vue에서 해당 값이 배열이라는 것을 알 수 있도록 하기 위해 -->
<!-- v-bind를 이용해 문자열이 아닌 JavaScript 표현식이라는 것을 알려줌. -->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>

<!-- 변수의 값을 동적으로 할당할 수도 있다. -->
<blog-post v-bind:comment-ids="post.commentIds"></blog-post>
```

### 오브젝트(Object) 전달

```html
<!-- 해당 오브젝트는 정적인 값이지만, Vue에서 해당 값이 배열이라는 것을 알 수 있도록 하기 위해 -->
<!-- v-bind를 이용해 문자열이 아닌 JavaScript 표현식이라는 것을 알려줌. -->
<blog-post
  v-bind:author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
></blog-post>

<!-- 변수의 값을 동적으로 할당할 수도 있다. -->
<blog-post v-bind:author="post.author"></blog-post>
```

### 오브젝트의 속성(Properties) 전달

오브젝트의 모든 속성을 전달하길 원하는 경우, `v-bind:prop-name` 대신 `v-bind` 만 작성함으로써 모든 속성을 prop으로 전달할 수 있다. 예를 들어, 아래와 같은 `post` 라는 오브젝트가 있다면,

```js
post: {
  id: 1,
  title: 'My Journey with Vue'
}
```

아래와 같이 작성하는 것은

```html
<blog-post v-bind="post"></blog-post>
```

아래와 같이 동작한다.

```html
<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>
```

이렇게 하면, 일일이 `post.` 으로 시작하는 속성을 정의해주지 않아도 됌. 따라서 하위 컴포넌트에서 사용할 때는, id와 title만으로 꺼내쓸 수 있다.

<br/>

## 단방향 데이터 흐름

모든 prop들은 부모와 자식 사이에 **단방향으로 내려가는 바인딩** 형태를 취한다. 부모의 속성이 변경되면 자식 속성에게 전달되지만, 반대 방향으로 전달되지 않는다. 자식의 데이터가 부모에게 전달되는 것을 막는 것은 자식 요소가 의도치 않게 부모 요소의 상태를 변경함으로써 앱의 데이터 흐름을 이해하기 어렵게 만드는 일을 막기 위해서임.

또한, 부모 컴포넌트가 업뎃 될 때마다 자식 요소의 모든 prop들이 최신 값으로 새로 고침됨. 이는 곧 사용자가 prop을 자식 컴포넌트 안에서 수정해서는 안된다는 말이 된다. 만약 수정을 시도하는 경우, Vue가 콘솔에 경고를 표시함.

아래 두 경우가 주로 prop을 직접 변경하고 싶을 수 있는 상황의 예시임.

### 1. prop은 초기값만 전달하고, 자식 컴포넌트는 그 초기 값을 로컬 데이터 속성으로 활용하고 싶은 경우

해당 경우에는 로컬 데이터 속성을 따로 선언하고 그 속성의 초깃값으로 prop을 사용하는 것이 가장 바람직 함.

```js
props: ['initialCounter'],
data: function() {
  return {
    counter: this.initialCounter
  }
}
```

### 2. 전달된 prop의 형태를 바꾸어야 하는 경우

이 경우에는 computed 속성을 사용하는게 바람직함.

```js
props: ['size'],
computed: {
  normalizedSize: function() {
    return this.size.trim().toLowerCase()
  }
}
```

📌 JS의 오브젝트나 배열을 prop으로 전달하는 경우, 객체를 복사하는 겻이 아니라 **참조**하게 된다. 즉, 전달받은 오브젝트나 배열을 수정하게 되는 경우, 자식 요소가 부모 요소의 상태를 **변경하게 될 것**이다.

<br/>

## Prop 유효성 검사

컴포넌트는 prop의 유효성 검사를 위해 요구사항을 특정할 수 있다. 요구사항이 충족되지 않은 경우 Vue는 콘솔을 통해 경고 함. 이는 다른 사람들도 사용하는 컴포넌트를 개발하는 경우 특히 유용함.

Prop들의 유효성 검사를 위해 `prop` 값에 배열이나 문자열 대신 오브젝트를 삽입할 수 있다. 예를 들어

```js
Vue.component('my-component', {
  props: {
    // 기본 타입 체크 (`Null`이나 `undefinded`는 모든 타입을 허용함.)
    propA: Number,
    // 여러 타입 허용
    propB: [String, Number],
    // 필수 문자열
    propC: {
      type: String,
      required: true
    },
    // 기본값이 있는 숫자
    propD: {
      type: Number,
      default: 100
    },
    // 기본값이 있는 오브젝트
    propE: {
      type: Object,
      // 오브젝트나 배열은 꼭 기본값을 반환하는 팩토리 함수의 형태로 사용되어야 함.
      default: function () {
        return { message: 'hello' };
      }
    },
    // 커스텀 유효성 검사 함수
    propF: {
      validator: function (value) {
        // 값이 항상 아래 세 개의 문자열 중 하나여야 함. 
        return ['success', 'warning', 'danger'].indexOf(value) !== -1;
      }
    }
  }
});
```

📌 Prop의 유효성 검사는 컴포넌트 인스턴스가 생성되기 전에 일어난다. 즉, 인스턴스의 값 (data, computed, 등등)은 'default'나 'validator' 함수 안에 사용될 수 없다.

### Type Checks

`type` 은 아래 있는 네이티브 생성자 중 하나가 될 수 있다.

- String
- Number
- Boolean
- Array
- Object
- Date
- Function
- Symbol

또한, `type` 에는 커스텀 생성자가 사용될 수도 있다. 확인은 `instanceof` 를 통해 이루어짐. 예를 들어, 아래와 같은 생성자 함수가 선언되어 있다면,

```js
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}
```

아래와 같이 작성함으로써

```js
Vue.component('blog-post', {
  props: {
    author: Person
  }
})
```

`author` prop이 `new Person` 으로 생성된 값인지 확인할 수 있다.

<br/>

## Prop이 아닌 속성

Prop이 아닌 속성은 컴포넌트에 전달되긴 하지만 대응되는 prop이 정의되지 않는다.

명확하게 정의된 prop을 통해 자식 컴포넌트에 정보를 전달하는 것이 권장되긴 하지만, 컴포넌트 라이브러리를 만드는 등의 경우 어떤 맥락에서 해당 컴포넌트가 사용될지를 확실히 상정할 수 없는 경우가 있음. 이 때는 컴포넌트는 임의의 속성값을 받아와 컴포넌트의 루트 엘리먼트에 추가해줄 수 있다.

예를 들어, `data-date-picker` 속성이 필요한 부트스트랩의 서드파티 컴포넌트인 `bootstrap-date-input` 을 사용하는 상황을 가정해보면, 아래와 같이 컴포넌트 인스턴스 속성을 추가해줄 수 있다.

```html
<bootstrap-date-input data-date-picker="activated"></bootstrap-date-input>
```

### 기존 속성의 대체 및 병합

아래 코드가 `bootstrap-date-input` 의 템플릿이라 해보자.

```html
<input type="date" class="form-control">
```

날짜 선택 플러그인의 테마를 설정하기 위해서 아래와 같이 특정 클래스를 작성해주어야 한다.

```html
<bootstrap-date-input
  data-date-picker="activated"
  class="date-picker-theme-dark"
></bootstrap-date-input>
```

이 경우 두 개의 각각 다른 값이 `class` 에 선언됨.

- `form-control` : 컴포넌트 템플릿으로부터 부여됨
- `date-picker-theme-dark` : 컴포넌트의 부모로부터 전달받아 부여됨

대부분 속성의 경우, 전달받은 속성이 기존에 선언된 속성을 대체 한다. 예를 들어, `type="text"` 를 `type="date"` 가 선언된 컴포넌트에 전달하는 경우에는 속성이 대체되고 문제를 일으키게 될 가능성이 생긴다. 하지만 다행히도 `class` 와 `style` 속성의 경우에는 조금 더 똑똑하게 반응함. 즉, 앞의 `form-control` 과 `date-picker-theme-dark` 의 예제와 같이 두 개의 값이 합쳐져서 적용된다.

### 속성 상속 비활성화

컴포넌트 루트 엘리먼트가 상속된 속성을 **갖지 않기를** 원하는 경우, 컴포넌트에 `inheritAttrs: false` 옵션을 줄 수 있다.

```js
Vue.component('my-component', {
  inheritAttrs: false,
  // ...
})
```

이는 컴포넌트에 전달된 속성의 이름과 값을 가지고 있는 인스턴스 속성인 `$attrs` 와 조합하면 유용하게 사용할 수 있음.

```js
{
  required: true,
  placeholder: 'Enter your username'
}
```

`inheritAttrs: false` 와 `$attrs` 를 이용하면 수동으로 전달할 속성을 선택할 수 있으며, 이는 스타일 규칙 중 *base components* 를 지키는 방법으로써 꽤 바람직하다.

```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
})
```

📌 `inheritAttrs: false` 옵션은 `style`과 `class`의 바인딩에 영향을 주지 **않는다는** 것을 기억하자!

이런 패턴을 이용하면 기본 컴포넌트의 실제 루트를 크게 신경쓰지 않고도 기본 HTML 엘리먼트에 가깝게 사용할 수 있다.

```html
<base-input
  v-model="username"
  required
  placeholder="Enter your username"
></base-input>
```

