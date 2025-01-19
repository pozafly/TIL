# Mutate, mutateAsync

> [출처](https://itchallenger.tistory.com/587)

mutate는 아무것도 반환하지 않는 반면, mutateAsync는 변형 결과가 포함된 Promise를 반환한다. 따라서, 변형 응답에 접근해야 할 때 `mutateAsync`를 사용하고 싶을 수 있지만, mutate를 사용해야 한다고 한다.

콜백을 통해 데이터 또는 오류에 계속 액세스 할 수 있으며 오류 처리에 대해 걱정할 필요가 없다. mutateAsync를 사용하면 Promise를 제어할 수 있으므로 수동으로 오류를 잡아야 한다.

그렇지 않으면 unhandled promise rejection을 만나게 된다.

```js
const onSubmit = () => {
  // ✅ accessing the response via onSuccess
  myMutation.mutate(someData, {
    onSuccess: (data) => history.push(data.url),
  })
}

const onSubmit = async () => {
  // 🚨 works, but is missing error handling
  const data = await myMutation.mutateAsync(someData)
  history.push(data.url)
}

const onSubmit = async () => {
  // 😕 this is okay, but look at the verbosity
  try {
    const data = await myMutation.mutateAsync(someData)
    history.push(data.url)
  } catch (error) {
    // do nothing
  }
}
```

**React Query는 내부적으로 오류를 포착(및 폐기)하기 때문에 Mutate에서는 오류를 처리할 필요가 없다.**

mutateAsync를 통해 오류처리하려면 다음과 같이 사용할 수 있다. `mutateAsync().catch(noop)`

하지만, mutateAsync가 필요할 때도 있는데, Promise가 반드시 필요한 경우에만 사용해야 한다.
