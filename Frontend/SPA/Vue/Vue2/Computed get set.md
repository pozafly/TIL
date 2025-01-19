# Computed set & get

> 출처: https://tofusand-dev.tistory.com/44

computed 프로퍼티는 methods와 비슷하지만, 강력한 힘을 가지고 있다. **Computed는 캐싱(cache)**을 가지고 있다.

캐시(cache)는 잠시 저장해둔다는 의미를 가진 특별한 기능임. 네트워크 영역에서도 캐시는 **로컬 장소에 파일을 미리 받아놓는 것**을 의미한다. 웹 서버에서도 매번 번거롭고 느리게 로딩을 해야하는 파일들을 미리 로딩해두고, 빠른 응답을 주기도 함.

<br/>

## Computed 프로퍼티로 퍼포먼스 최적화 하기

computed는, Vue 인스턴스 function 내에서 `this` 키워드와 연결 된다. Computed는 data /methods 와 같은 scope에 존재하고, 같은 방법으로 `<template>` 안에서 접근 가능함.

computed는 기본적으로 `getter` 만 가지고 있지만, 필요하다면 `setter` 를 지정하는 것도 가능하다. 그러기 위해서는 computed를 `get` 과 `set` 두가지 속성을 가진 Object로 변경해주어야 함. `get` 은 값을 가져올 때, `set` 은 값을 변경할 때 사용함.

```js
computed: {
  fullName: {
    get: function() {...}   // get() {...}
    set: function(newValue) {...}   // set() {...}
  }
}
```

`set` 은 newValue를 받는다. 해당 매개변수는 computed 프로퍼티에 특정 값을 등록하고자 할 때 입력할 새로운 값을 포함하고 있다. 따라서 `set` 메서드 안에서는 data 프로퍼티를 간단하게 업뎃할 수 있음.
