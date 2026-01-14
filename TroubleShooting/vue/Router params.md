# Router params

라우터로 params를 받고 싶었다. 먼저, 이전 화면에서 넘어올 경우 변수로 받을 수 있는 것은 두종류.

1. query
2. params

query는 url이 찍히기 때문에 개인 정보 노출 때문에 params를 사용하고 싶었다. 처음 코드는,
```js
this.$router.push({ name: 'auth', params: { id } });
```
이렇게 해주었다.

![[assets/images/20f04775a3df76b6c34ca2a8a623447a_MD5.png]]

`[vue-router] Route with name 'auth' dose not exist` 라고 뜨는 것이다. 하지만, routes/index.js에는
```js
path: '/auth',
redirect: '/auth/login',
component: () => import('@/views/AuthPage.vue'),
  children: [
    {
      path: 'login',
      component: () => import('@/components/auth/LoginForm.vue'),
    },
    (...)
```
이렇게 `path` 가 존재했다. 중요한 것은, params를 넘길 때는 `this.$router.push({ path: 'auth', params: { id } });` 이 부분에서 path를 넘기는 것이 아니라, 반드시 `name` 을 넘겨야 한다. 즉,
```js
this.$router.push({ name: 'auth', params: { id } });
```
이렇게 name 값을 넘겨주어야 함. 그리고, routes/index.js에서는 name 속성을 적어주어야 제대로 path를 찾아간다.
```js
path: '/auth',
redirect: '/auth/login',
component: () => import('@/views/AuthPage.vue'),
  children: [
    {
      path: 'login',
      name: 'login',  // 🌈
      component: () => import('@/components/auth/LoginForm.vue'),
    },
    (...)
```
- 🌈: 이 부분이 필요한 것이다. 그러면 제대로 params이 도착한다.
```js
mounted() {
  if (this.$route.params) {
    this.userData.id = this.$route.params.id;
  }
},
```
이런 식으로 해줄 수 있겠다.
