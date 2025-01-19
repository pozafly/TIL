# nextTick

> 출처: https://kr.vuejs.org/v2/api/index.html#Vue-nextTick

- Vue.nextTick([callback, context])
- 사용방법:
  - 다음 DOM 업데이트 사이클 이후 실행하는 콜백을 연기한다. DOM 업데이트를 기다리기 위해 일부 데이터를 변경한 직후 사용해야 한다.

```js
// 데이터를 변경한다.
vm.msg = 'Hello'
// 아직 DOM 업데이트가 되지 않았다
Vue.nextTick(function() {
  // DOM이 업데이트 되었다.
})

// promise와 함께 쓰기
Vue.nextTick()
  .then(function() {
  // DOM이 업데이트 되었다.
})
```

<br/>

## 라이프 사이클과 nextTick

> 출처: https://iam-song.tistory.com/48

라이프 사이클은 다음과 같은 순서임.

```
created(vue 객체 생성) > mounted(vue 객체를 dom에 붙인 시점) > destroyed(vue 객체 삭제)
```

따라서 아래와 같은 코드는 이렇게 찍힌다.

```js
created() {
  console.log('vue component 생성', this.$el, this._isMounted)
},

mounted() {
  console.log('vue component dom에 그려짐', this.$el, this._isMounted)
},

destroyed() {
  console.log('vue component 삭제', this.$el, this._isMounted)
}
```

결과 로그를 통해 알 수 있다.

```html
vue component 생성 undifined false
vue component dom에 그려짐 > <div>...</div> true
vue component 삭제 > <div>...</div> true
```

`created` 시점에는vue 객체가 생성만 하고 `mounted` 시점에는 vue 객체의 html이 실제 dom에 그려져 컴포넌트의 dom이 찍힌다. 그런데, 만약 created 시점에 vue의 dom을 접근하고 싶다면 `nextTick` 을 사용하면 된다.

```js
created() {
  console.log('vue component 생성', this.$el, this._isMounted)
  this.$nextTick(() => {
    console.log('in nextTick', this.$el, this._isMounted); 
  })
},

mounted() {
  console.log('vue component dom에 그려짐', this.$el, this._isMounted)
},

destroyed() {
  console.log('vue component 삭제', this.$el, this._isMounted)
}
```

결과는,

```html
vue component 생성 undifined false
vue component dom에 그려짐 > <div>...</div> true
vue component in nextTick > <div>...</div> true  // 🔥
vue component 삭제 > <div>...</div> true
```

- 🔥: 이 부분. mounted 후에 찍힌다.

즉, nextTick은 DOM 업데이트가 된 후에 콜백으로 선언된 함수가 실행하도록 돕는 메서드임. 이 메서드를 통해 dom이 그려지기 전의 시점에서도 에러 없이 dom을 핸들링할 수 있다.
