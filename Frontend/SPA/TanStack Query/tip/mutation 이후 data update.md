# mutation 이후 data update

mutation은 query와 다르게 조회(get) 말고, post, delete, update 와 같은 데이터를 변경하는 작업을 할 수 있도록 하는 함수다.

```ts
export const useDeleteDoc = () => {
  
  return useMutation<{ message: string } | string, AxiosError, string>({
    mutationKey: [DIARY_KEY],
    mutationFn: (fileId: string) => deleteDoc(fileId),
  });
};
```

이와 같이 사용할 수 있다.

mutationKey를 통해 mutation을 식별할 수 있고, 후에 `useIsMutating` 훅을 통해 mutation 중인 작업을 loading spinner로 표시해줄 수있다.

하지만, 위 작업은 단지 mutation 할 뿐, 변경된 데이터를 다시 화면에 표시하려면, 어떤 작업이 필요하다.

1. refetch를 통해 **서버에서** 다시 데이터 조회해오기
2. mutation이 완료되면, **클라이언트**에서 기존 데이터 기반으로 데이터를 가공해 데이터를 다시 넣어주기

두가지 방법이 존재한다. 두 작업 모두 알아보자.

## 서버에서 재조회

서버에서 재조회 하는 방법은, `queryClient.invalidateQueries({ queryKey: [...] });` 를 통해 할 수 있다.

```ts
export const useDeleteDoc = () => {
  const queryClient = useQueryClient(); // queryClient를 가져와야 함
  
  return useMutation<{ message: string } | string, AxiosError, string>({
    mutationKey: [DIARY_KEY],
    mutationFn: (fileId: string) => deleteDoc(fileId),
    onSuccess() { // here
      queryClient.invalidateQueries({ queryKey: [DIARY_KEY] });
    }
  });
};
```

`onSuccess` 메서드를 생성하고, 불러온 queryClient를 통해 `invlidateQueries` 메서드를 쿼리키와 함께 호출하면, 기존에 useQuery로 조회해온 데이터가 다시 조회가 되면서 기존 데이터가 변경된다.

하지만, 이 작업을 할 경우 mutation 시 동작했던, loading spinner가 2번 뜰 수 있다. 따라서 따로 처리를 해주어야 하는 번거로움이 생긴다.

또한, 만약에 서버의 데이터를 변경 후 즉시 조회하게 되면, 서버에 여러 시스템이 연동되어 있을 경우, 변경 내용이 반영되지 않은 채 변경 전 데이터가 조회될 가능성이 있다. 서버가 비동기로 동작하거나 적용 되는데 시간이 걸리는 경우이다. 그러면 동시에 딜레이를 주어 재조회 해줄 수 있다.

```ts
onSuccess() {
  setTimeout(() => {
    queryClient.invalidateQueries({ queryKey: [DIARY_KEY] });
  }, 800);
}
```

하지만, 이런 경우도 좋지 않은 로직이다. 이유는, 800ms 이 정확한 수치가 아니기 때문이다. 서버의 상태에 따라서 정확한 데이터를 다시 조회할 수도 있지만, 서버에 과부하가 걸려 있는 상태라면, 다시 변경되지 않은 데이터가 조회될 확률이 높기 때문이다.

## 요청에 따른 클라이언트 데이터 변경

그렇다면, 요청이 마무리 되었다면, 클라이언트 데이터를 변경후 리렌더링해주어 사용자가 즉시 변경된 데이터를 볼 수 있도록 해줄 수 있다.

1. getQueryData & setQueryData
2. setQueryData & Updater

### getQueryData & setQueryData

react-query의 `getQueryData`는 조회한 데이터 캐시의 데이터를 가져오는 작업이다.

```ts
onSuccess(_, variables) { // variables는 mutation 함수가 받는 변수다.
  const query = queryClient.getQueryData<DriveFile>([DIARY_KEY, 'docList']);
  queryClient.setQueryData<DriveFile>([DIARY_KEY, 'docList'], {
    ...query,
    files: query?.files.filter(file => file.id !== variables) as File[],
  });
}
```

1. `getQueryData`를 통해 데이터를 캐시에서 조회한다.
2. 캐시 데이터를 가공한다.
3. 가공한 데이터를 불변성을 지켜서 `setQueryData` 메서드의 인자로 넣어준다.

📌 주의점은 반드시 불변성을 지켜서 넣어주는 것이 좋다. 불변성을 지켜주지 않으면 react의 setState와 같이 리렌더링이 일어나지 않기 때문이다.

### setQueryData & Updater

getQueryData를 사용하지 않고 setQueryData만 사용하는 방법이다. 이 방법은 캐시에서 다시 조회하는 로직이 없어도 useMutation 자체에서 제공하는 방법이다.

```ts
onSuccess(_, variables) {
  queryClient.setQueryData<DriveFile>([DIARY_KEY, 'docList'],
    (oldData): DriveFile => ({
      ...oldData,
      files: oldData?.files.filter(file => file.id !== variables) as File[],
    })
  );
},
```

`setQueryData`의 두번째 인자로 함수를 넣어줄 수 있는데, 이는 업데이터 함수이다. 업데이터 함수는 react의 setState 함수와 사용법이 동일하다. 인자로 기존 쿼리한 데이터를 가지고 오며, 쿼리한 데이터를 기반으로 가공한 값을 불변성을 지켜 반환하면, 캐시된 데이터가 업데이트 되고 화면이 리렌더링 된다.

첫번째 방법처럼, 조회 한 후 업데이트 하기보다는 업데이터 함수를 사용하는 방법이 좀 더 간편하다고 느껴진다.