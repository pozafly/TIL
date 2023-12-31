# provider pattern

> [출처](https://velog.io/@kdeun1/Vue-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EC%97%90%EC%84%9C%EB%8A%94-Provider-Pattern%EC%9D%84-%EC%96%B4%EB%96%BB%EA%B2%8C-%ED%99%9C%EC%9A%A9%ED%95%A0%EA%B9%8C)

## 컴포넌트의 역할

react는 컴포넌트 함수를 실행하며 JSX를 평가, 화면 렌더링한다. Vue는 양방향 바인딩을 통해 템플릿 데이터를 렌더링한다.

방식은 다르지만 중점적인 것은 렌더링 된 데이터를 부모 컴포넌트로부터 자식 컴포넌트에 어떻게 전달할지가 가장 중요하다. 일반적 컴포넌트가 props를 통해 데이터를 아래로 전달. React 경우 부모 상태 setter를 자식에 전달해 상호작용하고, Vue의 경우 Proxy로 구현된 반응형 패턴으로 인해 양방향 바인딩이 지원되어 emit을 통해 데이터를 올리게 된다.

<br/>

## Props Driliing

간단 컴포넌트 구조 데이터 전달은 props, emit을 통해 전달이 비교적 쉬움. 하지만 중첩 구조가 많아질 수록 어려움.

<br/>

## Provider Pattern 방식

불필요하게 중간 컴포넌트를 거치지 않고 데이터를 전달해주는 방식이 존재함. 즉, react, vue 모두 기본적으로 Provider Pattern 방식이 있다.

|                        | 상태 제공자        | 상태 소비자        |
| ---------------------- | ------------------ | ------------------ |
| Vue2 (Option API)      | provide            | inject             |
| Vue3 (Composition API) | provide()          | inject()           |
| React (JSX)            | `<Ctx.Provider />` | `<Ctx.Consumer />` |
| React (Fn)             | createContext()    | useContext()       |

Vue 2.2.0 이후로 provide, inject가 추가 되었고, Vue 3의 `provide()`, `inject()` 는 React의 `createContext()`, `useContext()` 와 사용법이 비슷하다.

Provider Pattern을 사용하면 장점

- 데이터를 거쳐가는 중간 컴포넌트 불필요한 데이터 패싱 로직이 없어도 된다. 즉, 트리 구조에서 모든 계층 건너뛸 수 있음.
- 프로젝트를 거시적으로 볼 때 하나의 컨텍스트 안에서 `provider`, `injector(consumer)` 는 순수 함수처럼 인풀과 아웃풋은 동일함.
- provider는 injector(consumer)가 어디에 있는지 신경쓰지 않아도 된다.
  - 각자 역할은 데이터를 공금하거나 주입(소비) 하는 역할에만 집중한다. 또한 injector가 멀티로 존재할 수 있다.

<br/>

## 예시

테마 변경 및 다국어 변경과 같이 간단 동작이지만, 플젝 전역에 영향을 미치는 부분에 활용하면 좋다. 하지만 뻔한 부분 외 실제 프로젝트에 적용한 예시는 다음과 같음.

- 체크박스, 체크박스 그룹
- 라디오, 라디오 그룹
- 차트, 차트 그룹

```vue
// CheckboxGroup.vue
<template>
	<div
  	class="checkbox-group"
    role="group"
  >
   	<slot />
  </div>
</template>

<script lang="ts" setup>
const { computed, provide } from 'vue';

type CheckboxValue = 'string' | 'Number' | 'Boolean' | 'Symbol';
interface Props {
	modelValue: CheckboxValue[];
}
interface Emit {
	(e: 'update:modelValue', value: CheckboxValue): void;
}

const props = withDefault(defineProps<Props>(), {
	modelValue: () => [],
});
const emit = defineEmits<Emit>()

const mv = computed({
	get: () => props.modelValue,
  set: val => emit('update:modelValue', val);
});
provide('CheckboxGroupProvider', mv);
</script>
```

```vue
// Checkbox.vue
<template>
  <label class="checkbox">
    <input
      v-model="mv"
      type="checkbox"
      ...
    />
      ...
  </label>
</template>

<script setup lang="ts">
const { computed, inject } from 'vue';

type CheckboxValue = 'string' | 'Number' | 'Boolean' | 'Symbol';
interface Props {
	modelValue: CheckboxValue;
}
interface Emit {
	(e: 'update:modelValue', value: CheckboxValue): void;
}

defineProps<Props>();
const emit = defineEmits<Emit>()

const mv = inject(
  'CheckboxGroupProvider',
  computed({
    get: () => props.modelValue,
    set: val => emit('update:modelValue', val)
  });
);
</script>
```

핵심은 아래와 같음

- provider()에 반응형 변수 `mv` 를 넣는다. 그 뒤 inject() 함수에서 사용된다. 이렇게 구성하면 `<CheckboxGroup />` 과 `<Checkbox />` 컴포넌트 사이에 어떤 컴포넌트가 존재하더라도 **부모-자식 컨텍스트 사이에서는 데이터를 넘길 수 있다**.
- 단순 컴포넌트(atom, molecules 컴포넌트) 사이에서 Provider Pattern을 사용하였으며, **별도의 상태 관리 라이브러리를 사용하지 않는다**.
- 그룹 내 여러 개의 `<Checkbox />` 컴포넌트가 있더라도 상호작용하여 반응할 수 있다. 이 구조에서 All Check나 Indeterminate 기능 구현할 수 있음.
- `<CheckboxGroup />` 컴포넌트와 `<Checkbox />` 컴포넌트가 Provider, Injector 로서 같이 동작할 수 있지만, Vue의 Inject() 함수 두 번째 파라미터(default 값)를 활용해 독립적으로 `<Checkbox />` 컴포넌트만을 사용할 수 있다. 만약 inject() 함수에 반환된 값은 Provider key 가 없다면 두 번째 파라미터 default 코드로서 역할을 하는데, 이 위치에 반응형 변수를 넣어 injector(consumer) 역할을 하는 컴포넌트를 독립적 컴포넌트로 사용할 수 있다.
- 공통 바인딩 값을 공유해 반복적 바인딩 템플릿 코드를 줄일 수 있다. Provider Component 코드 바인딩 값을 inject() 에 내려준다면 Injector Component 템플릿 코드에서 바인딩 할 필요가 없다. 즉, Injector Component 에서 반복적으로 사용되는 바인딩 코드를 Provider Component 코드에서만 사용해 코드 량을 줄일 수 있다.
- 그 밖에 차트에서 이 패턴을 사용한다고 가정하자. 차트 그룹 안 차트 줌, 툴팁 위치, 선택된 라벨 위치, 여러 차트 컴포넌트 등을 하위에 넣고 서로 간 상호작용한 결과 값들을 공유할 때 유용할 것이다.

<br/>

Provider  패턴을 안좋은 시선으로 바라보는 것도 존재하고 실제로 공식 문서에서도 provide를 사용할 때 주의하라는 내용도 존재함.

중간 컴포넌트를 거치지 않는 부분에서는 장점으로 바라볼 수 있으나, 상태 관리 라이브러리를 사용하지 않는다는 부분 때문에 데이터 관리적 측면에서는 불편하다는 단점도 존재한다.

개인적으로 **격리된 컨텍스트가 명확하다면** 비즈니스 로직이 없는 컴포넌트에서는 블랙박스처럼 사용하고 활용하는데 있어 매력적인 패턴이라 생각.

