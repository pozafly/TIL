# Code-splitting

> [출처](https://ui.toast.com/weekly-pick/ko_20200128)

Vue 컴포넌트를 **경로 별**로 코드 스플리팅 사용. 페이지에 필요한 컴포넌트만 로드되고, 원한다면 다른 컴포넌트도 함께 로드할 수 있다. 많은 컴포넌트를 작성하고 여러 경로를 설정(라우팅) 해주어야 하는 대형 프로젝트에서 코드 스플리팅을 사용하면 로드 시간을 단축해 사이트의 성능을 향상시킬 것이다.

<br/>

## 설정

Vue 라우터를 사용한다.

```vue
<template>
	<div class="about">
    <h1>This is an about page</h1>
    <router-link to="/">Home Page</router-link>
  </div>
</template>
```

우선 바벨의 동적 가져오기 플러그인(`@babel/plugin-syntax-dynamic-import`)을 설치해야 함.

```shell
$ npm i -D @babel/plugin-syntax-dynamic-import
```

이것은 웹펙이 동적 import를 이해하고 이에 맞게 번들링 하는데, 서버 측에서 바벨이 import 구문을 해석하고 트랜스파일하기 위해서이다.

> `syntax-dynamic-import`는 바빌론(*babylon*)에서 구문 사용만 가능하게 해준다. 즉, 바빌론이 구문 오류 없이 코드를 구문 분석할 수 있다는 말이다. 여전히 이해할 수 없는 `import(…)`코드가 남아있으며, 트랜스 파일 해야 한다. `dynamic-import-node`가 `import(…)`를 이해할 수 있는 `require`로 트랜스파일한다. 클라이언트 측에서, 웹팩이 구문을 이해하고 번들링 하므로 트랜스파일 단계가 필요 없다. **Satyajit Sahoo**

```json
// .babelrc.json
{
  "plugin": ["@babel/plugin-syntax-dynamic-import"]
}
```

<br/>

## 구현

```vue
<script>
  import Vue from 'vue';
  import Router from 'vue-router';
  import Home from './views/Home.vue';
  import About from './views/About.vue';
  
  Vue.use(Router);
  export default new Router({
    routes: [
      {
        path: '/',
        name: 'home',
        components: Home,
      },
      {
        path: '/about',
        name: 'about',
        components: About,
      },
    ]
  });
</script>
```

코드 스플리팅 없이 초기 로드 시 About 컴포넌트는 아직 필요하지 않더라도 Home 컴포넌트와 함께 번들 파일로 제공된다. 초기 로드가 필요하지 않은 컴포넌트는 번들 파일에 포함되지 않도록 수정해보자. Promise 방식으로 가져오는 방법이 있다. 프로미스를 지원하지 않는 구형 브라우저를 위해 폴리필을 포함하는 것도 잊으면 안된다.

```vue
<script>
  import Vue from 'vue';
  import Router from 'vue-router';
  const Home = () => import('./views/Home.vue');
  const About = () => import('./views/About.vue');
  
  Vue.use(Router);
  export default new Router({
    routes: [
      {
        path: '/',
        name: 'home',
        components: Home,
      },
      ...
    ]
  });
</script>
```

import 구문을 반환하는 함수를 작성해주면 끝이다. 이것이 동적 import 구문이다. 웹팩은 import 구문을 만날 때마다 프로미스에 대한 응답으로 코드 스플리팅이라고 불리는 청크(Chunk)를 생성함. 리로드 후 네트워크 탭을 확인해보면 컴포넌트가 모두 로드 되지 않음. 경로를 바꿔 하나씩 방문하면 컴포넌트가 결과 창에 차례로 추가되는 것을 확인할 수 있음.

이제 네트워크 탭에 표시된 파일 이름이 숫자로 표시되었던 것을 주석을 통해 이름을 붙여보자.

```vue
<script>
  (...)
  const Home = () => import(/* webpackChunkName: "Home" */'./views/Home.vue');
  const About = () => import(/* webpackChunkName: "About" */'./views/About.vue');
  (...)
</script>
```

웹팩은 주석에 있는 문자 그대로 청크 이름으로 해석한다. 숫자는 문자열로 바뀌었다.
