# fetch API vs Axios

---

## fetch API

```javascript
class Youtube {
  constructor(key) {
    this.key = key;
    this.getRequestOptions = {
      method: 'GET',
      redirect: 'follow',
    };
  }

  async mostPopular() {
    const response = await fetch(
      `https://www.googleapis.com/youtube/v3/videos?part=snippet&chart=mostPopular&maxResults=25&key=${this.key}`,
      this.getRequestOptions
    );
    const result = await response.json();   // 일일이 json으로 변경해주었다.
    return result.items;
  }
}
```

반환된 결과 값을 일일이 json으로 변경해서 사용해주어야 함. 인터넷 익스플로러와 같은 이전 버전의 브라우저에서는 fetch API는 동작하지 않고 XMLHttpsRequest 를 사용해주어야 하는 번거로움이 있다.

<br/>

## Axios

```jsx
class Youtube {
  constructor(key) {
    this.youtube = axios.create({
      baseURL: 'https://www.googleapis.com/youtube/v3',   // 베이스 url
      params: { key },
    });
  }

  async mostPopular() {
    const response = await this.youtube.get('videos',  {   // get 메서드 호출
      params: {  // 파라미터 설정
        part: 'snippet',
        chart: 'mostPopular',
        maxResults: 25,
      },
    });
    return response.data.items;  // json으로 알아서 변경시켜 줌.
  }
```

- fetch API는 예전 브라우저(익스플로러)에서 동작하지 않지만, axios는 예전 브라우저에서도 잘 동작하게 만들어졌다. 
- 예전 브라우저에서는 XMLHttpsRequest를 사용해주고, 요즘 브라우저에서는 fetch API를 사용해준다.
- 일일이 json으로 return값을 변경해주지 않아도 알아서 해줌.

