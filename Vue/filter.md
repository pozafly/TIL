# filter

vue에서 filter는 텍스트 형식화를 적용할 수 있게 해준다.

<br/>

## 필터 문법

- 머스테치

  ```js
  {{ 데이터 | 필터함수 }}
  ```

- v-bind

  ```html
  <div v-bind:id="데이터 | 필터함수"></div>
  ```

<br/>

## 지역 필터

```jsx
// template 선언문에,
{{ postItem.createdAt | formatDate }}

// script 선언문에,
filters: {
  formatData(value) {
    return new Date(value);
  },
},
```

이렇게 넣었을 때,

<img width="408" alt="제목 없는 그림" src="https://user-images.githubusercontent.com/59427983/111904305-93d08100-8a89-11eb-9607-9f21979391ae.png">

이런 식으로 나오게 된다.

<br/>

## 전역 필터

### filter.js

```js
// 필터 관련 함수가 존재하는 파일
export function formatDate(value) {
  const date = new Date(value);
  const year = date.getFullYear();
  let month = date.getMonth() + 1;
  const day = date.getDate();
  let hours = date.getHours();
  const minutes = date.getMinutes();
  
  month = month > 9 ? month : `0${month}`;
  hours = hours > 9 ? hours : `0${hours}`;
  
  return `${year}-${month}-${day} ${hours}:${minutes}`;
}
```

### main.js

```js
import Vue from 'vue';
import App from './App.vue';
import router from '@/routes/index';
import store from '@/store/index';
import { formatDate } from '@/utils/filters';   // 여기 추가

Vue.filter('formatDate', formatDate);   // 여기 추가
Vue.config.productionTip = false;

new Vue({
  render: h => h(App),
  router,
  store,
}).$mount('#app');
```

### 사용할 .vue 파일

```js
// template 선언문에,
{{ postItem.createdAt | formatDate }}

// script 선언문에,
filters: {
  formatDate(value) {
    return new Date(value);
  },
},
```

이렇게 전역 필터를 선언해서 사용하고 싶은 곳에서 사용할 수 있다.