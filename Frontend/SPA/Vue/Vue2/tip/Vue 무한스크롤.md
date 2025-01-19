# Vue 무한 스크롤

먼저 백엔드 부터 쿼리문을 바꾸어 줬다.

https://namjug-kim.github.io/2019/02/15/mysql-paging-qeury.html

https://wbluke.tistory.com/18

https://velog.io/@minsangk/커서-기반-페이지네이션-Cursor-based-Pagination-구현하기

https://jojoldu.tistory.com/528

이렇게 커서 기반 페이지네이션이 있다. 일반적으로, 오프셋 기반 페이지네이션을 사용하면,

MySQL에서 limit와 offset 두개를 사용해서, 쿼리를 조회해온다. 만약 데이터가 100000개가 넘는다고 가정했을 때, limit 와 offset을 사용하게 되면 모든 데이터를 조회한 후, 위치에 맞는 녀석을 데리고 오기 때문에 limit와 offset을 사용하게 되면 서브쿼리나 조인문으로 해결하는 방법을 사용하지 않으면 서버의 성능상 문제가 생길 가능성이 높아진다.

커서기반 페이지네이션은, 클라이언트가 가져간 마지막 row의 순서상 다음 row들을 n개 요청/응답하게 구현하는 것을 말한다.

Tripllo 같은 경우, 현재 createdAt을 desc 순으로 정렬하고 있는데, 마지막에 도착한 data의 createdAt 전 녀석들을 가져오는 방법을 사요하는 것이다. 따라서 Personal Board 를 조회하는 기준(커서)은, 마지막 DOM의 createdAt 다.

쿼리문부터 보자.

기존의 쿼리문

```jsx
<select id="readPersonalBoardList" parameterType="String" resultType="com.pozafly.tripllo.board.model.Board">
    select
        a.id,
        a.title,
        a.bg_color,
        a.public_yn,
        a.hashtag,
        a.like_count,
        a.created_at,
        a.created_by,
        EXISTS
        (
            select *
            from board_has_like
            where board_id = a.id and user_id = #{userId}
        ) as own_like
    from board a
    where a.created_by = #{userId}
    order by created_at desc
</select>
```

모든 데이터를 조회해온다.

이제 수정한 쿼리문을 보자.

```jsx
<select id="readPersonalBoardList" parameterType="Map" resultType="com.pozafly.tripllo.board.model.Board">
    select
        a.id,
        a.title,
        a.bg_color,
        a.public_yn,
        a.hashtag,
        a.like_count,
        a.created_at,
        a.created_by,
        EXISTS
        (
            select *
            from board_has_like
            where board_id = a.id and user_id = #{userId}
        ) as own_like
    from board a
    where a.created_by = #{userId}
    <choose>
        <when test='"firstCall".equals(lastCreatedAt)'>
            order by created_at desc
            limit 14
        </when>
        <otherwise>
            and created_at <![CDATA[ < ]]> #{lastCreatedAt}
            order by created_at desc
            limit 6
        </otherwise>
    </choose>
</select>
```

MyBatis의 choose when otherwise를 사용하여, 맨 처음 콜 했을 때는 firstCall 이라는 걸 매개변수로 받아서, 14개의 데이터만 가지고 가게 만들었으며, 그 후에 lastCreatedAt 변수에 마지막 DOM의 createdAt를 담아서 던지게 되면 otherwise에 걸려 and 조건문을 타게 되는 구조이다. limit에 따라 가지고 오는 데이터 양이 바뀐다.

이제 Vue를 보자. 먼저 npm으로 vue 자체에서 제공하는 무한로딩 라이브러리를 가지고 왔다.

https://peachscript.github.io/vue-infinite-loading/guide/start-with-hn.html

```bash
$ npm install vue-infinite-loading -S
```

후에, main.js에 플러그인 형식으로 등록해줬다.

```jsx
import InfiniteLoading from 'vue-infinite-loading';

Vue.use(InfiniteLoading);
```

http://yoonbumtae.com/?p=2823

그리고 사용방법은

```html
<div class="list-wrap" ref="boardItem">  <!-- 여기 ref 등록해주어야 자식의 마지막 DOM을 가져올 수 있다. -->
  ...
  <div
    class="board-list"
    v-for="board in personalBoardList"
    :key="board.id"
    :data-last-created-at="board.createdAt"  <!-- dataset을 지정해두었음. -->
  >
    <BoardItem :board="board" />
  </div>
</div>

...

<infinite-loading @infinite="infiniteHandler" spinner="waveDots">
  <div
    slot="no-more"
    style="color: rgb(102, 102, 102); font-size: 14px; padding: 10px 0px;"
  >
    목록의 끝입니다 :)
  </div>
</infinite-loading>
```

이 태그를 페이지의 밑 부분에 주자. 페이지의 스크롤이 땅에 닿이면 이게 작동을 한다. @infinite를 보면 옆에 메서드를 등록시킬 수 있는데,

```jsx
data() {
	return {
		...
		lastCreatedAt: 'firstCall',
	}
}

...

async infiniteHandler($state) {
  this.READ_PERSONAL_BOARD_LIST({   // 퍼스널 보드를 조회하는 action함수
    lastCreatedAt: this.lastCreatedAt,
  });
  await setTimeout(() => {
		// isInfinity는 state에 올라가 있다. 초기 값은 Y
    if (this.isInfinity === 'Y') {
			// 마지막 DOM의 dataset에서 createdAt을 가져와, data에 등록된 lastCreateAt에 집어넣는다.
      this.lastCreatedAt = this.$refs.boardItem.lastChild.dataset.lastCreatedAt;
      $state.loaded();  // 계속 데이터가 남아있다는 것을 infinity에게 알려준다.
    } else {
      $state.complete();  // 데이터는 모두 소진되고 다시 가져올 필요가 없다는 것을 알려준다.
    }
  }, 1000);
},
```

if 안에 state인, isInfinity 를 검사하는 부분이 있다. 초기 값은 Y이며, 데이터를 들고올 때 마지막 데이터라 아무 데이터가 없다면 N으로 쳐진다. 이 때 N으로 쳐진다면 $state.complete() 함수로 infinity에게 데이터를 가져올 필요가 없고, 목록의 끝임을 클라이언트에게 알려주게 된다. setTimeout을 준 이유는 너무 빠르게 내리면 무한 스크롤이 걸리지 않은 것 같은 효과를 내기 때문에 1초 정도 텀을 줌.

isInfinity를 N으로 치는 로직을 보자.

```jsx
READ_PERSONAL_BOARD_LIST({ commit }, { lastCreatedAt }) {
  return boardApi
    .readPersonalBoardList({ lastCreatedAt })
    .then(({ data }) => {
      if (data.data === null) {
        commit('setIsInfinity', 'N');
        return;
      }
      commit('setPersonalBoardList', data.data);
    });
},
```

이렇게 되어 있는데, $state.loaded()와 $state.complete() 의 역할을 잘 몰라서, 목록이 끝났을 때도 fetch하는 액션함수가 계속 실행되어 오류를 내기도하고 아예 액션함수가 동작하지 않기도 했다. 원래는 Vuex로 되어있지 않고 로컬 data에서 list에 밀어넣는 구조가 되어있었다. 하지만, Vuex를 사용하면서 action 단에서 어떻게 하면 모든 데이터가 소진되었다는걸 component 단으로 알려줄 수 있을까 고민하다가 state 하나를 만들어서, 그것으로 판별하게 하면 되겠다고 생각했다.

마지막으로 action에서 commit 하는 Mutation 함수를 보자.

```jsx
pushPersonalBoardList(state, data) {
  state.personalBoardList = state.personalBoardList.concat(data);
},
resetPersonalBoardList(state) {
  state.personalBoardList = [];
	state.isInfinity = 'Y';
},
```

state에 저장하는 presonalBoardList에 concat하는 형식으로 이루어지고 있다. 그리고 reset 뮤테이션을 넣어준 이유는, 다른 ID로 로그인 했을 경우 다시 데이터를 불러와서 concat 하는 구조이므로, 이전 ID의 Board 데이터가 그대로 남아있기 때문에 vue의 beforeDestroy를 사용하여 페이지를 벗어날 때마다 reset 되도록 해주었다. 또한 isInfinity 값도 Y로 쳐줘야하는 이유는 이전 사용자가 데이터 전부를 불러오고 나서 state 값이 'N' 이 되었을 때, 다음 사용자 또한 N이므로 불러오지 못하는 이슈가 생기므로. 따라서 concat 해주는 녀석들은 반드시 이런 reset 관련 메서드가 필요할 것이다.

기존에 boardList를 조회하는데 recentBoard(최근 본 보드), invitedBoard(초대된 보드) 목록을 한 Http Method에서 받아서 처리하는 형식으로 하고 있었다.

```java
@GetMapping("{userId}/{recentLists}/{invitedLists}")
public ResponseEntity<Message> readBoardList(
  @PathVariable String userId,
  @PathVariable String recentLists,
  @PathVariable String invitedLists
) {
  List<String> recentList = null;
  if(!"null".equals(recentLists)) {
      String[] el = recentLists.split(",");
      recentList = new ArrayList<>(Arrays.asList(el));
  }
  List<String> invitedList = null;
  if(!"null".equals(invitedLists)) {
      String[] el = invitedLists.split(",");
      invitedList = new ArrayList<>(Arrays.asList(el));
  }

  return boardService.readBoardList(userId, recentList, invitedList);
}
```

@PathVariable이 무려 3개다;; 그래서 언젠가 이 구조를 따로 떼어서 작업 해야겠다고 생각했는데 이번 기회에 따로 떼어 사용하게 되었다. 지향하는 서비스는 비슷하지만, 가져오는 데이터 종류가 다르므로 컨트롤러 2개를 새로 추가하고 기능을 떼어내서 분리시켰음.

<br/>

## 좋아요 클릭 시 반영되지 않는 문제

이런 구조로 바꾸게 되면서 문제점이 생겼는데, 좋아요를 클릭 시 데이터베이스에는 등록 되지만 화면에 바로 반영되지 않는 문제점이 생기게 되었다. 이유는 action 함수인, `READ_PERSONAL_BOARD_LIST` 이녀석. 좋아요 버튼을 클릭 한 뒤, 이 녀석을 action에서 이어서 dispatch 하게 되는데 이 때 들어갈 매개변수의 종류가 바뀌게 되었기 때문이다.

여기서 고민점이 생긴다. `READ_PERSONAL_BOARD_LIST` 는 PathVariable로 lastCreatedAt 를 받는데 무한 스크롤 이후에 가져오는 녀석들이 현재 state에 존재하는 list에 concat 되기 때문인데.. 그렇다면 api를 새로 다시 만들어주어야겠다고 생각했다.
