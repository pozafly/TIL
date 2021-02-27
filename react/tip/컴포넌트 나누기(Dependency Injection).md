# 컴포넌트 나누기(Dependency Injection)

디자인 패턴이 있다. mvc, mvvm, mvp 등.. 역할을 나누는 것인데, 역할을 나누어 두면 어떤 컴포넌트가 어떤 역할을 하는지 유지보수가 쉬워지고 찾아가는데도 어렵지 않게 된다.

따라서, react에서 각 component는 멍청해야 한다. 한 component안에서 서버 통신도 하고 ui도 보여주고 많은 일을 하는 똑똑한 component가 되면 나중에 역할을 찾게 되는 것이 어렵게 된다. 따라서 컴포넌트를 멍청하게 단일 역할을 하는 구조로 만드는 것이 좋다.

즉,

- 컴포넌트 안에서 네트워크 통신을 하면 좋지 않다.
- 특히 네트워크 통신하는 녀석을 분리해두지 않으면 mock 객체로 테스트 하기 어렵다.

<br/>

## 네트워크 통신 부분 모듈화

기존에 네트워크 통신 하는 녀석이 중복되기도 하므로 src에 service라는 디렉토리를 만들어 `youtube.js` 라는 순수 자바스크립트 파일로 빼보자. js는 class로 만들 것이다.

```jsx
// app.jsx
import { useEffect, useState } from 'react';
import styles from './app.module.css';
import SearchHeader from './components/search_header/search_header';
import VideoList from './components/video_list/video_list';

function App() {
  const [videos, setVideos] = useState([]);
  const search = (query) => {
    const requestOptions = {
      method: 'GET',
      redirect: 'follow',
    };

    fetch(
      `https://www.googleapis.com/youtube/v3/search?part=snippet&maxResults=25&q=${query}&type=video&key=AIzaSyAt8c2PYwx485f9FMJmgxfrHRIOA_IOTB4`,
      requestOptions
    )
      .then((response) => response.json())
      .then((result) =>
        result.items.map((item) => ({ ...item, id: item.id.videoId }))
      )
      .then((items) => setVideos(items))
      .catch((error) => console.log('error', error));
  };
  useEffect(() => {
    const requestOptions = {
      method: 'GET',
      redirect: 'follow',
    };

    fetch(
      'https://www.googleapis.com/youtube/v3/videos?part=snippet&chart=mostPopular&maxResults=25&key=AIzaSyAt8c2PYwx485f9FMJmgxfrHRIOA_IOTB4',
      requestOptions
    )
      .then((response) => response.json())
      .then((result) => setVideos(result.items))
      .catch((error) => console.log('error', error));
  }, []);
  return (
    <div className={styles.app}>
      <SearchHeader onSearch={search} />
      <VideoList videos={videos} />
    </div>
  );
}

export default App;

```

이 녀석인데 fetch가 2번 동일하게 반복되는 것을 볼 수 있다.

```javascript
// youtube.js
class Youtube {
  constructor(key) {
    this.key = key;   // 키 받아올 것임
    this.getRequestOptions = {   // 옵션을 밑에서 쓸 것임.
      method: 'GET',
      redirect: 'follow',
    };
  }

  mostPopular() {
    return fetch(
      'https://www.googleapis.com/youtube/v3/videos?part=snippet&chart=mostPopular&maxResults=25&key=AIzaSyAt8c2PYwx485f9FMJmgxfrHRIOA_IOTB4',
      this.getRequestOptions
    )
      .then((response) => response.json())
      .then((result) => result.items);
  }

  search(query) {
    return fetch(
      `https://www.googleapis.com/youtube/v3/search?part=snippet&maxResults=25&q=${query}&type=video&key=AIzaSyAt8c2PYwx485f9FMJmgxfrHRIOA_IOTB4`,
      this.getRequestOptions
    )
      .then((response) => response.json())
      .then((result) =>
        result.items.map((item) => ({ ...item, id: item.id.videoId }))
      );
  }
}

export default Youtube;
```

이렇게 작성할 수 있다. 다만 app.jsx에서 이제 `const youtube = new Youtube()` 로 객체를 생성해서 쓴다고 생각할 수 있는데, 이것은 함수형 컴포넌트가 계속 불려질 때마다 객체를 다시 생성하기 때문에 좋은 방법이 아니다.

```jsx
// app.jsx
function App({ youtube })   // DI를 위해 객체 주소를 받아올 것임.
```

이렇게 외부에서 받아오는 것으로 하고, (DI를 위해.) index.js에서 `const youtube = new Youtube(key)` 로 객체를 상위에서 만들어주고 1번만 주입해줄 것이다.

```jsx
// index.js
import Youtube from './service/youtube';

const youtube = new Youtube('aaa');  // 여기 객체 1개 생성. key는 아직 옮기지 않음
ReactDOM.render(
  <React.StrictMode>
    <App youtube={youtube} />  // props로 내려줌
  </React.StrictMode>,
  document.getElementById('root')
);
```

이렇게 되면 이제 DI 준비는 마무리 되었다. app.jsx를 약간만 손보면 됨.

```jsx
function App({ youtube }) {   // DI
  const [videos, setVideos] = useState([]);
  const search = (query) => {
    youtube
      .search(query) // 요부분
      .then((videos) => setVideos(videos));  // 비디오 업데이트
  };
  useEffect(() => {
    youtube
      .mostPopular() // 요부분
      .then((videos) => setVideos(videos));  // 비디오 업데이트
  }, []);
  return (
    <div className={styles.app}>
      <SearchHeader onSearch={search} />
      <VideoList videos={videos} />
    </div>
  );
}
```