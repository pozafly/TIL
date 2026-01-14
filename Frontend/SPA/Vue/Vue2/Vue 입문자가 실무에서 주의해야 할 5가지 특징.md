# Vue 입문자가 실무에서 주의해야 할 5가지 특징

> 출처: https://www.youtube.com/watch?v=Z9OGUU6G8vM

장기효님 발표 정리.

<br/>

## 목차

1. 반응성 - 왜 내 화면은 다시 그려지지 않는 걸까?
2. DOM 조작 - 오래된 습관 버리기
3. 라이프사이클 - 나는 인스턴스를 얼마나 알고 있나?
4. ref 속성 - 만들다가 만 ref 속성
5. computed 속성 - 간결한 템플릿의 완성

<br/>

<br/>

## 1. 반응성 - 왜 내 화면은 다시 그려지지 않는 걸까?

### 뷰의 반응성이란?

- 데이터의 변화에 따라 화면이 **다시 그려지는 뷰의 성질**
```js
var vm = new Vue({
  data: {
    count: 0
  }
});
vm.count += 1;  // count 값이 증가하면 화면에 표시된 count 값도 증가
```
<br/>

### 반응성은 언제 설정될까?

인스턴스가 *생성될 때* data의 속성들을 **초기화**.

라이프사이클의 관점에서 보면,

![[assets/images/aa49dad7b3ab1816e808def6ed675b11_MD5.png]]

이벤트와 라이프 사이클이 beforeCreate 이전에 초기화 되고, created 메서드 전에 반응성이 주입이 된다. 따라서, created 전에 반응성이 주입이 되어 그 때부터 Vue 의 반응성을 다룰 수 있게 된다.

<br/>

### 반응성에 대해 알아야 할 점

- 생성하는 시점에 없었던 data는 반응하지 않는다.
  ```js
  var vm = new Vue({
    data: {
      user: {
        name: 'Captain'
      }
    }
  });
  vm.user.age += 1;  // age 값이 변하더라도 화면은 갱신되지 않는다.
  ```
  스크립트의 값은 변했지만, 화면의 DOM은 갱신되지 않는다.

  체크박스, 토글 등 화면에서만 필요한 UI 상탯값을 다룰 때 주의해야 한다. UI 상탯값 같은 경우, Database에서 관리하지 않기 때문에 데이터를 가져왔을 때 객체에 속성이 없을 가능성이 많다.

  체크박스가 하나 있다고 가정하고, script 상으로 checked를 true로 두는 button이 있다고 하자.
  ```js
  data: {
    some: {
      (...)
      // checked: false,
    }
  }
  ```
  checked가 초기화가 되어있지 않다고 했을 때 버튼을 눌러도 아무 동작을 하지 않게 된다. 하지만 `$set()` 메서드를 사용하면 객체 안에 속성을 명시해줄 수 있게 도와준다.
  ```js
  fetch('...')
    .then(data => {
      this.users = data;
      this.$set(this.users[0], 'checked', false);
    });
  ```
  이렇게 설정해주면 api로 데이터를 받아온 직후, users 라는 객체에 checked: false 라는 값을 집어넣어주고, ui를 클릭했을 때, checked 값이 변하면서 vue의 반응성이 이루어지게 되는 것이다.

- 백엔드에서 불러온 데이터에 임의 값을 추가하여 사용하는 경우

  이 경우도 위와 비슷하다. 백엔드로부터 어떤 객체를 받아왔다고 가정하면, 그 객체에 속성을 추가해야할 경우, 기존에 정의되어 있지 않은 속성에 대해서는 Vue가 반응하지 않는다. this.$set() 메서드를 사용하거나, 따로 함수를 만들어두고 속성을 추가하는 작업을 해주어야 한다.

- 뷰엑스의 state도 data와 동일하게 취급
  ```js
  state: {
    user: { name: 'Captain' }
  },
  mutations: {
    // 생성하는 시점에 없었던 데이터는 반응성이 없음
    setUserAge: function(state) { state.user.age = 23; },  // X
    // 객체 속성을 임의로 추가 또는 삭제하는 경우 뷰에서 감지하지 못함
    deleteName: function(state) { delete state.user.name; }  // X
  }
  ```
  하지만, Vue 3.0에서는 괜찮다.

  Object.defineProperty() 에서 Proxy 기반으로 변화했기 때문이다.
  ```js
  var obj = {};
  
  // Vue 2.x
  Object.defineProperty(obj, 'str', { .. });
  
  // Vue 3.x
  new Proxy(obj, { .. });
  ```
<br/>

<br/>

## 2. DOM 조작 - 오래된 습관 버리기

### (기존) 화면 조작을 위한 DOM 요소 제어 방법

- 특정 DOM을 *검색*해서 제어하는 방법
  ```js
  // 네이티브 자바스크립트
  document.querySelector('#app');
  
  // 제이쿼리 라이브러리
  $('#app');
  ```
- 사용자의 입력 *이벤트*를 기반으로 한 DOM 요소 제어
  ```js
  // 버튼 요소 검색
  var btn = document.querySelector('#btn');
  
  // 사용자의 클릭 이벤트를 기반으로 가장 가까운 태그 요소를 찾아 제거
  btn.addEventListener('click', function(event) {
    event.target.closest('.tag1').remove();
  });
  ```
<br/>

### (Vue 방식) Ref 속성을 활용한 DOM 요소 제어

- 뷰에서 제공하는 *ref 속성*
  ```jsx
  <!-- HTML 태그에 ref 속성 추가 -->
  <div ref="hello">Hello Ref</div>
  
  // 인스턴스에서 접근 가능한 ref 속성
  this.$refs.hello;  // div 엘리먼트 정보
  ```
- 뷰 *디렉티브*에서 제공되는 정보를 최대한 활용
  ```html
  <ul>
    <li v-for="(item, index) in items">
      <span v-bine:id="index">{{ item }}</span>
    </li>
  </ul>
  ```
<br/>

### DOM 제어 사고 전환이 필요한 실제 사례
```jsx
<ul>
  <li @click="removeItem">
    <span>메뉴 1</span>
    <div class="child hide">메뉴 설명</div>
  </li>
  <li @click="removeItem">
    <span>메뉴 2</span>
    <div class="child hide">메뉴 설명</div>
  </li>
  ...
</ul>

...

methods: {
  removeItem(event) {
    event.target.lastChild.classList.toggle('hide');
  }
}
```
이걸 줄일 수 있다.
```jsx
<ul>
  <li v-for="(item, index) in items" @click="removeItem(index)">
    <span>{{ item }}</span>
    <div class="child hide" ref="listItem">
      메뉴 설명
    </div>
  </li>
</ul>

...
data: {
  items: ['메뉴 1', '메뉴 2', '메뉴 3'],
},
methods: {
  removeItem(index) {
    this.$ref.listItem[index].classList.toggle('hide');
  }
}
```
<br/>

<br/>

## 3. 인스턴스 라이프 사이클 - 나는 인스턴스를 얼마나 알고 있나?

뷰 인스턴스가 생성되고 소멸되기까지의 생애 주기

![[assets/images/588d5f09d9f7082f3c0e23ee79556133_MD5.png]]

이미지 출처: Do it! Vue.js 입문(이지스퍼블리싱)

- 뷰의 템플릿 속성 - 인스턴스, 컴포넌트의 표현부를 정의하는 속성
  ```jsx
  // 인스턴스 옵션 속성
  new Vue({
    data: { str: 'Hello World' },
    template: '<div>{{ str }}</div>'
  });
  
  <!-- 싱글 파일 컴포넌트 -->
  <template>
    <div>{{ str }}</div>
  </template>
  ```
- 뷰 템플릿 속성의 정체 - 실제 DOM 엘리먼트가 아니라 **Virtual DOM(자바스크립트 객체)**
  ```jsx
  <!-- 사용자가 작성한 코드 -->
  <template>
    <div>{{ str }}</div>
  </template>
  
  // 라이브러리 내부적으로 변환한 모습
  function render() {
    with(this) {
      return _c('div', [_v(_s(str))]);
    }
  }
  ```
<br/>

### 템플릿 속성이 실제로 유효한 시점

: 인스턴스가 화면에 *부착(mounted)* 되고 난 후

<br/>

### 인스턴스 부착 시점을 이해하지 못한 사례 1
```jsx
<!-- 템플릿 속성 -->
<template>
  <canvas id="myChart"></canvas>
</template>

// 인스턴스 옵션
new Vue({
  created: function() {
    var ctx = document.querySelector('#myChart');   // null
    new Chart(ctx, chartOptions);
  }
})
```
created 라이프 사이클이므로 DOM 조작이 불가능함.

<br/>

### 인스턴스 부착 시점을 이해하지 못한 사례 2
```jsx
<!-- 템플릿 속성 -->
<template>
  <canvas id="myChart"></canvas>
</template>

// 인스턴스 옵션
new Vue({
  created: function() {
    this.$nextTick(function() {  // 업데이트 시점 혼란 야기 및 코드 복잡도 증가.
      var ctx = document.querySelector('#myChart');
      new Chart(ctx, chartOptions);      
    });
  }
})
```
nextTick을 사용했다. 업데이트 시점 혼란 야기 및 코드 복잡도 증가. 따라서 `mounted` 에서 사용할 수 있는 것은 거기서 하되, 반드시 필요한 곳 최종적으로 선택할 부분만 사용해야 한다. 즉, 위 인스턴스 옵션엔 mounted에 작성해주면 된다.

<br/>

<br/>

## 4. Ref 속성 - 만들다가 만 ref 속성

### Ref 속성이란?

- 특정 **DOM** 엘리먼트나 하위 **컴포넌트**를 가리키기 위해 사용
- **DOM 엘리먼트**에 사용하는 경우 **DOM 정보**를 접근
- **하위 컴포넌트**에 지정하는 경우 컴포넌트 **인스턴스 정보** 접근
- **v-for** 디렉티브에 사용하는 경우 **Array** 형태로 정보 제공

즉, **특정 DOM 요소를 조작하고 싶을 때 사용하는 속성**

<br/>

### Ref 속성 사용할 때 주의할 점 1

- ref 속성은 템플릿 코드를 render 함수로 변환하고 나서 생성
- 따라서 ref에 접근할 수 있는 최초의 시점은 `mounted` 라이프사이클 훅.
  ```jsx
  <p ref="pTag">Hello</p>
  
  created: function() {
    this.$refs.pTag;  // undifined
  },
  mounted: function() {
    this.$refs.pTag;  // <p>Hello</p>
  }
  ```
<br/>

### Ref 속성 사용할 때 주의할 점 2

- `v-if` 디렉티브와 사용하는 경우 화면에 해당 영역이 그려지기 전까진 DOM 요소 접근 불가.
  ```jsx
  <div v-if="isUser">
    <p ref="w3c">W3C</p>
  </div>
  
  new Vue({
    data: { isUser: false },
    mounted: function() {
      this.$refs.w3c;   // undifined
    }
  })
  ```
  따라서 v-if 대신 `v-show` 를 사용하면 된다. v-show는 DOM은 그려지지만, 화면상 보이지않게 처리하는 것이기 때문이다.

<br/>

### Ref 속성 사용할 때 주의할 점 3

- 하위 컴포넌트의 내용을 접근할 순 있지만 남용하면 안된다.
  ```jsx
  <div id="app">
    <TodoList ref="list"></TodoList>
  </div>
  
  new Vue({
    el: '#app',
    methods: {  // 상위 컴포넌트에서 불필요하게 하위 컴포넌트를 제어하는 코드
      fetchItems: function() { this.$refs.list.fetchTodos(); }
    }
  });
  ```
  `this.$refs.list` 는 하위 컴포넌트인 TodoList가 되고, TodoList 컴포넌트의 fetchTodos() 라는 메서드를 실행하는 코드다. 이렇게 되면 좋지 않은게 이미, props, emit으로 컴포넌트 간 통신 규약이 있기 때문에 그걸 사용하는게 좋다.
  ```jsx
  <div id="app">
    <TodoList></TodoList>
  </div>
  
  // 하위 컴포넌트의 라이프 사이클 훅을 이용
  var TodoList = {
    methods: {
      fetchTodos: function() { ... }
    },
    created: function() { this.fetchTodos(); }
  };
  ```
<br/>

<br/>

## 5. Computed 속성 - 간결한 템플릿의 완성

- **간결**하고 **직관적인** 템플릿 표현식을 위해 뷰에서 제공하는 속성
```jsx
<p>{{ 'hello' + str + '!!' }}</p>  <!-- 템플릿 표현식만 이용하는 경우 -->
<p>{{ greetingStr }}</p>  <!-- computed 속성을 활용하는 경우 -->

new Vue({
  data: { str: 'world' },
  computed: {
    greetingStr: function() {
      return 'hello' + this.str + '!!';
    }
  }
});
```
### Computed 속성 활용처 1

- 조건에 따라 HTML *클래스*를 **추가, 변경**할 때
```jsx
<li v-bind:class="{ disabled: isLastPage }"></li>

computed: {
  isLastPage: function() {
    var lastPageCondition = 
        this.paginationInfo.current_page >= this.paginationInfo.last_page;
    var nothingFetched = Object.keys(this.paginationInfo).length === 0;
    return lastPageCondition || nothingFetched;
  }
}
```
<br/>

### Computed 속성 활용처 2

- 스토어(Vuex)의 state 값을 접근할 때
```jsx
<div>
  <p>{{ this.$store.state.module1.str }}</p>
  <p>{{ module1Str }}</p>
</div>

new Vue({
  computed: {
    module1Str: function() {
      return this.$store.state.module1.str;
    }
  }
});
```
<br/>

### Computed 속성 활용처 3

- Vue i18n 과 같은 **다국어 라이브러리**에도 활용 가능
```jsx
<p>{{ 'userPage.common.filter.input.label' }}</p>
<p>{{ inputLabel }}</p>

computed: {
  inputLabel: function() {
    return $t('userPage.common.filter.input.label');
  }
}
```