# isLoading, isFetching, isRefetching

> [출처](https://medium.com/@kurucaner/isloading-vs-isfetching-1617133c0e3c)

- isLoading: 초기 데이터 가져오기 중에만 true로 설정된다.
- isFetching: 수동으로 데이터를 가져왔는지 여부에 관계 없이 API 호출이 수행될 때마다 true.
- isRefetching : `refetch()` 함수를 사용해 의도적으로 다시 가져올 때만 true.

```jsx
import {
  useQuery,
  QueryClient,
  QueryClientProvider,
} from "@tanstack/react-query";

const queryClient = new QueryClient();

export default function Home() {
  return (
    <QueryClientProvider client={queryClient}>
      <RandomName />
    </QueryClientProvider>
  );
}

function RandomName() {
  const { data, isLoading, isFetching, isRefetching, refetch } = useQuery({
    queryKey: ["randomName"],
    queryFn: async () => {
      const response = await fetch("https://randomuser.me/api/");
      const data = await response.json();
      return data;
    },
    retry: 1,
  });
  console.log("isLoading", isLoading);
  console.log("isFetching", isFetching);
  console.log("isRefetching", isRefetching);

  return (
    <div>
      <button
        onClick={() => {
          refetch();
        }}
      >
        Fetch Again!
      </button>
    </div>
  );
}
```

- retry: 쿼리 실패 시 재시도 횟수를 나타냄.
- refetch: 쿼리 다시 불러오기를 수동으로 트리거하기 위해 호출 가능.

기본적으로 처음으로 데이터를 가져올 때는 `isLoading`이 반환된다. 하지만, 처음이 아니라도 언제든 isFetching이 발생함. refetch 할 때만 isRefetching이 발생함.