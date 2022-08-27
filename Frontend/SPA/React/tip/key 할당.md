# key 할당시 map 사용

<br/>

key를 할당하는데 있어 유용하게 사용되는 것이 Array의 `map` 이다.

```jsx
// app.jsx
fetch(
  `https://www.googleapis.com/youtube/v3/search?part=snippet&maxResults=25&q=${query}&type=video&key=AIzaSyAt8c2PYwx485f9FMJmgxfrHRIOA_IOTB4`,
  requestOptions
)
.then((response) => response.json())
.then((result) => setVideos(result.items))   // state 업뎃
...
<VideoList videos={videos} />

// video_list.jsx
<ul className={styles.videos}>
  {props.videos.map((video) => (
    <VideoItem key={video.id} video={video} />
  ))}
</ul>
```

이런식으로 `setVideos` 로 state를 업뎃 해주고, video_list에서 props로 video의 id를 key로 설정하고 있었다.

이 때, 여기서 key가 Object로 표현되고 있었는데 items 안에는 id가 Object 형태였다. 바꿔줘야 할 것은 items 안에 있는 id..

```jsx
.then((result) =>
  result.items.map((item) => ({ ...item, id: item.id.videoId }))  // 여기서 한번 가공
)
.then((items) => setVideos(items))   // map은 return 하므로 .then으로 받아서 setState.
```

여기서 보면 map으로 새 배열을 만들면서, id(Object)를 새로 id.videoId를 할당하여 새롭게 만든 items를 state(videos)에 할당해서 내려주고 있는 것을 볼 수 있다.

즉, map을 이용해서 id만 바꿔서 내려주는 형태로 작성 한 것이다.

