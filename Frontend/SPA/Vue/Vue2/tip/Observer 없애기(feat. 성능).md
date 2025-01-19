# Observer 없애기(feat. 성능)

> 출처: [Vue 성능 관련 포스팅](https://kdydesign.github.io/2019/04/10/vuejs-performance/)

Vue.js를 사용한 프로젝트에서 성능 관련 이슈에 대해 설명하고 있다. 대용량 데이터를 다뤄야 할 때 성능을 위한 최적화가 필요하다는 것.

요점은, 객체가 Vue의 `data` 에 바인딩 될 때, `Object.freeze()` 메서드를 사용해 오브젝트 자체를 readonly로 바꿔서 Vue.js가 감지할 필요가 없다는 것을 알려주면 대용량 객체를 다룰 때 성능이 좋아진다는 것이다.

<br/>

이전에 여러 포스팅을 보던 중 아래와 같은 문법을 만난 적이 있다.

```js
JSON.parse(JSON.stringify(this.items));
```

도대체 왜 vue 안에서 data에 있는 `items` 라는 객체를 문자열로 바꾸고, 다시 객체화 하는지 모르겠더라.

<br/>

Vue에서 관리하는 객체는 Observer라는 객체 감지 기능이 함께 따라붙는다. 이는 Vue의 반응형과 연관이 있다. Vue의 `data` 에 매핑된 객체는 data가 변할 때 버추어 돔을 사용해 화면을 다시 그려주기 때문에 반드시 필요한 녀석이라고 볼 수 있는데

![반응성 주기](https://vuejs.org/images/data.png)

위의 사진과 같이 `data` 에 `getter`, `setter` 가 붙어있는걸 볼 수 있다. [공식 페이지](https://vuejs.org/v2/guide/reactivity.html#For-Objects)에 따르면

> 모든 구성 요소 인스턴스에는 해당 **watcher** 인스턴스가 있으며, 이 인스턴스는 컴포넌트가 종속적으로 렌더링되는 동안 '수정' 된 모든 속성을 기록한다. 나중에 종속적인 setter가 트리거 되면 watcher에 알리고 컴포넌트가 다시 렌더링 된다.

라고 적혀 있다. 따라서 data를 한번 선언한 수 다시 data에 새로운 객체를 추가하려고 할 때 오류가 나게 되는데, 이는 `Vue.set()` 메서드를 통해 추가할 수 있고 추가된 객체가 반응형 객체(Observer 가 붙은)가 되는 방식을 사용하고 있는 것이다.

<br/>

그럴 때, 굳이 객체에 붙어있는 Observer를 제거할 필요가 있느냐? 하면 사실을 잘 모르겠다. 대용량 데이터 처리할 때 Object.freeze()를 사용하면 확실히 성능의 개선 여지가 보인다.

아래는 data를 3만건 넣고 돌린 결과이다.

## Object.freeze() 사용하지 않음 - 1288ms

<img width="1330" alt="nofreeze2" src="https://user-images.githubusercontent.com/59427983/132081449-76af9dad-1044-4bbf-b0fb-e214b548967e.png">

## Object.freeze() 사용 - 1186ms

<img width="1374" alt="freeze2" src="https://user-images.githubusercontent.com/59427983/132081465-03e66c8d-6f90-46b5-9c1c-9bdc067608e6.png">

어쨌든 이번 실험에서는 성능이 약간 좋아졌다는 것을 알 수 있다. console을 찍어 봤을 때도 freeze를 사용한 객체는 Observer가 붙어있지 않다.

## Object.freeze() 사용하지 않음 - 1288ms

<img width="446" alt="스크린샷 2021-09-04 오후 1 02 55" src="https://user-images.githubusercontent.com/59427983/132081781-31576446-dc3e-46b9-aced-81338a7808b7.png">

## Object.freeze() 사용 - 1186ms

<img width="140" alt="스크린샷 2021-09-04 오후 1 02 47" src="https://user-images.githubusercontent.com/59427983/132081789-c32c8fa6-474a-4835-85f2-6ff51cdb6f65.png">

<br/>

[Vue의 객체를 일반 객체로 변환하는 방법 - Vue.js 이슈](https://github.com/vuejs/Discussion/issues/292)에 따르면, 창시자인 에반유가 답하기를

```js
JSON.parse(JSON.stringify(obj));
```

문법을 사용하면 Observer를 객체에서 제거할 수 있다고 한다. 다른 답변으로는

```js
[...this.items]
{ ...this.items }
Object.assign([], this.items)
```

이런 녀석들이 있을 수 있는데, 껍데기만 Observer가 없을 수 있지만 객체 내부에는 Observer가 붙어 있다.

위 이슈에서는 Observer가 붙지 않은 일반 객체를 만드는 이유가 `console.log()` 를 찍어볼 때, 직접 클릭해서 까보고 싶지 않을 때 사용했던 것 같다.

어쨌든.. 쓸데없는 삽질. 중요한 것은 대용량 데이터를 다룰 때, 성능을 조금이라도 높게 가져가고 싶다면, data에 저장할 때 혹은 state에 저장하기 전에 `Object.freeze()` 를 사용해 가공하지 못하게 만들어둔 채 뿌려주기만 하면 될 것 같다. 반응형이라는 큰 장점은 버리고 오로지 성능 업을 위한 트릭이다.
