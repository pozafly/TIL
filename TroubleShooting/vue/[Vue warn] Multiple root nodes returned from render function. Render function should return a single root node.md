# [Vue warn]: Multiple root nodes returned from render function. Render function should return a single root node.

> 출처 : https://www.inflearn.com/questions/32168

풀이하면, 렌더링 기능에서 여러 루트 노드가 반환되었다는 뜻. 렌더링 함수는 단일 루트 노드를 반환해야 한다는 뜻이다. 즉, 코드로 보면,

```html
<template>
  <div>hi</div>
  <div>hi2</div>
</template>
```

이런식으로 컴포넌트의 형태가 잡혀있다는 말이다. 이 오류를 해결하기 위해서는

```html
<template>
	<div>
  	<div>hi</div>
  	<div>hi2</div>
  </div>
</template>
```

요렇게 가장 상위 태그를 사용해주면 위 warning이 깔끔하게 해결된다. 다만, 문제는 지금처럼 .vue 파일을 직접 쓰는게 아니라 `renderless` 컴포넌트를 만들 때 발생했다는 것이다.

<br/>

📌 renderless 컴포넌트란, [출처](https://velog.io/@jtwjs/%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4) 

`templete` 표현식이 없고 `script` 단에서 비즈니스 로직만 처리해 상위 컴포넌트로 데이터를 노출시키주는 컴포넌트.

```html
<!-- Parent.vue -->
<template>
  <div>
    <fetch-data url="https://jsonplaceholder.typicode.com/users/">
      <!-- slot-scope 안에서만 데이터 접근 가능 -->
      <template v-slot="{ response, loading }">
        <div v-if="loading">
          {{ response }}
        </div>
        <div v-else>Loading...</div>
      </template>
    </fetch-data>
  </div>
</template>

<script>
import FetchData from '@/components/renderless/FetchData';

export default {
  components: {
    'fetch-data': FetchData,
  },
};
</script>
```

```js
// Child.js
import axios from 'axios';

export default {
  props: ['url'],
  data() {
    return {
      response: null,
      loading: false,
    };
  },
  created() {
    axios
      .get(this.url)
      .then(response => {
        setTimeout(() => {
          this.response = response.data;
          this.loading = true;
        }, 500);
      })
      .catch(err => {
        console.log(err);
      });
  },
  render() {
    return this.$scopedSlots.default({
        response: this.response,
        loading: this.loading,
      }),
    );
  },
};
```

이렇게 되어있다면, 위와 같은 오류가 난다.

기존의 slot을 `<div slot-scope="{ response, loading }">` 이라고 표현했으면 나지 않았겠지만, `v-slot` 이 도입되면서 `v-slot` 이 `<template>` 태그에 걸려 있기 때문에 `render()` 에서 하나의 태그로 묶지 않고 이를 2개 노드로 표현한 것.

따라서 Child.js의 `render()` 를 아래와 같이 바꿔 주면 된다.

```js
render() {
  return this.$createElement('div', [
    this.$scopedSlots.default({
      response: this.response,
      loading: this.loading,
    }),
  ]);
}
```