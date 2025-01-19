# staleTime, cacheTime 차이

> [출처](https://yrnana.dev/post/2021-04-10-react-query-staletime-cachetime/), https://ttaerrim.tistory.com/53

## React Query의 라이프 사이클

- A 쿼리 인스턴스가 mount 됨
- 네트워크에서 데이터 fetch 하고 A라는 query key로 캐싱함
- 이 데이터는 `fresh` 상태에서 `staleTime` (기본값 0) 이후 `stale` 상태로 변경됨
- A 쿼리 인스턴스가 unmount 됨
- 캐시는 `cacheTime` (기본값 5분) 만큼 유지되다가 가비지 콜렉터로 수집됨
- 만약 `cacheTime` 이 지나기 전 A 쿼리 인스턴스가 새롭게 mount되면, fetch가 실행되고 `fresh` 한 값을 가져오는 동안 캐시 데이터를 보여줌

<br />

## staleTime

데이터가 신선한 상태로 남아있는 시간을 말함. 리액트 쿼리에서의 기본 값은 `0`. 데이터를 페치 해오자마자 신선하지 않다고 판단하는 것임. 즉, 쿼리 한 데이터는 **fresh**할 수도 있고, **stale**할 수도 있다.

- fresh한 데이터라면, API 호출 없이 캐싱된 데이터가 다시 사용된다.
- stale한 데이터라면, 윈도 포커스 및 컴포넌트 다시 마운트, 네트워크가 다시 연결 되면 신선한 데이터를 다시 페치해올 것이다.

---

- 데이터가 `fresh` -> `stale` 상태로 변경되는데 걸리는 시간
- `fresh` 상태일 때는 쿼리 인스턴스가 새롭게 mount 되어도 네트워크 fetch가 일어나지 않는다.
- 데이터가 한번 fetch 되고 나서 `staleTime` 이 지나지 않았다면 unmount 후 mount 되어도 fetch가 일어나지 않는다.

<br />

## cacheTime

메모리에 저장된 캐싱 데이터가 유효한 시간을 말한다. 기본 값은 `300000`으로 5분. 쿼리를 사용하는 모든 컴포넌트가 언마운트 되었을 때, 쿼리는 비활성화(inactive) 된다. 비활성화된 데이터는 캐시에서 삭제된다. 데이터가 **stale**한 상태라면 데이터를 다시 페치해올 텐데, API 요청을 통해 신선한 데이터를 완전히 가져오기 전까지 캐싱된 데이터를 사용한다.

※ 데이터가 `inactive` 상태일 때 `캐싱 된 상태로` 남아있는 시간

- 데이터가 `inactive` 상태일 때 캐싱된 상태로 남아있는 시간
- 쿼리 인스턴스가 unmount 되면 데이터는 `inactive` 상태로 변경되며, 캐시는 `cacheTime` 만큼 유지된다.
- `cacheTime` 이 지나면 가비지 콜렉터로 수집된다.
- `cacheTime` 이 지나기 전에 쿼리 인스턴스가 다시 mount 되면, 데이터를 fetch 하는 동안 캐시 데이터를 보여준다.
- `cacheTime`은 `staleTime`과 관계없이, 무조건 inactive 된 시점을 기준으로 캐시 데이터 삭제를 결정한다.

<br/>

## 그 외

- `isLoading`: 캐싱된 데이터가 없을 때 fetch 중에 `true`
- `isFetching`: 데이터가 fetch될 때 `true`, 캐싱 데이터가 있어서 백그라운드에서 fetch 되더라도 `true`

---

<br />

### 기본 설정일 경우(staleTime = 0, cacheTime = 5m)

![image](https://github.com/pozafly/TIL/assets/59427983/8234c529-fa58-492b-b2b9-6b5b0315416a)

처음으로 쿼리가 호출되었을 때, API 요청을 하고 데이터를 캐싱한다. 지정된 staleTime에 따라 해당 데이터는 페치와 동시에 stale 상태로 변경된다.

![image](https://github.com/pozafly/TIL/assets/59427983/7bac9aa9-49d3-4e8f-b2bc-2a52918a9b45)

1분 뒤 같은 쿼리를 호출한다. 우선 캐시가 유효한지 확인한다. 캐시는 5분동안 유효하기 때문에 사용할 수 있다. 하지만 쿼리의 staleTime이 0이기 때문에 데이터는 신선하지 않고, 데이터 페치를 요청한다. 반환된 데이터가 캐시된 데이터와 다른 경우 캐시를 업데이트하고, 해당 데이터를 사용한다.

**기본 설정일 경우**, 데이터의 상태는 반환되지마자 stale로 판단되기 때문에 늘 stale한 데이터라고 할 수 있다. 따라서 쿼리를 **요청할 때마다 API 요청**을 하게 된다.

### staleTime이 5분, cacheTime이 5분일 때

![image](https://github.com/pozafly/TIL/assets/59427983/72c8a92e-c444-49f2-a65e-0a555fedd9ae)

처음 쿼리가 호출되면 위와 동일함. 1분 뒤 같은 쿼리를 호출한다. 캐시는 5분 동안 유효하기 때문에 사용 가능. staleTime도 동일하게 5분이므로, 아직 신선 데이터를 가지고 있다. 따라서 캐시된 데이터를 사용하고, API 요청은 하지 않는다. 30분 뒤 같은 쿼리를 호출한다. 5분이 지났기 때문에 캐시도 유효하지 않고 데이터도 stale 상태. API 요청을 통해 다시 데이터를 요청해 캐싱하고 전달 후 사용.

### staleTime 5분, cacheTime 0분일 때

![image](https://github.com/pozafly/TIL/assets/59427983/dc98d1c0-a2f2-470a-a04c-44c9117dde59)

처음은 동일. 1분뒤 같은 쿼리를 호출한다. 데이터는 신선한 상태. API 요청을 하지 않고 1에서 반환된 데이터를 그대로 사용함. 30분 뒤 같은 쿼리를 호출하면, 데이터는 stale 상태로 변질되었기 때문에 새로운 데이터를 페칭함.

### staleTime 언제 조정하면 좋을까?

`staleTime`이 기본 값인 0인 이산, 데이터는 절대 신선하지 않다. 만약 사용하는 데이터가 자주 변경되는 데이터라면, 기본 설정을 바꾸지 말자. 하지만 데이터의 변경이 자주 일어나지 않는다면, `staleTime`을 조정해 사용하는게 더 이득일 수 있다.

예를 들면 변경할 수 있는 사용자가 로그인 한 유저 단 한 명일 경우(개인 투두 리스트를 관리하는 경우, 프로픨 수정 등)에는 `staleTime`을 `Infinity`로 설정하고, POST / UPDATE / DELETE 이벤트가 발생했을 경우에만 쿼리 무효화를 통해 새로운 데이터를 갱신해 불필요한 HTTP Request 요청을 줄이고 효과적으로 서버 데이터를 관리할 수 있다.
