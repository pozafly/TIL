## API key 따로 빼기

- API key 같은 녀석(credential)들은 절대 코드에 올려두면 안된다.
- .env 파일이나 .env.product 파일을 만들어서 관리해야함.



최 상위 폴더에서 `.env` 파일을 만들자.

```
REACT_APP_YOUTUBE_API_KEY=[키를넣자]
```

키를 이렇게 선언해주고, 그리고 소스 내에서 사용할 때는

```javascript
process.env.REACT_APP_YOUTUBE_API_KEY
```

이렇게 불러 올 수 있다. vue 와 마찬가지로 역시 .env 파일은 무조건 서버 껐다 키기!!!

📌 그리고 반드시 .gitignore 추가해주기!!!!! 반드시다 반드시!!!! 제발 까먹지 말자!!!

