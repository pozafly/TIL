# 더미데이터 libs

## shortid

[shortid](https://www.npmjs.com/package/shortid) key를 할당해줄 때, 순서가 바뀌거나 렌덤한 id로 할당을 해줘야 할 때 유용한 라이브러리다.

```sh
$ npm i shortid
```

```js
import shortid from 'shortid';
 
console.log(shortid.generate());
// PPBqWA9
```

이런식으로 겹치기 정말 힘든 id를 만들어준다.

<br/>

## faker

닉네임, 댓글 컨텐츠 등, 각종 더미데이터를 알아서 뿌려준다. [faker](https://www.npmjs.com/package/faker) 이곳의 API Methods 부분에 보면 사용할 수 있는 더미데이터들을 보고 그대로 사용하면 됨.

```sh
$ npm i faker
```

```js
import faker from 'faker';

Array(20).fill().map((v, i) => ({
  id: shortid.generate(),
  User: {
    id: shortid.generate(),
    nickname: faker.name.findName(),  // 아무 이름
  },
  content: faker.lorem.paragraph(),  // 아무 글자나 넣어줌
  Image: [{
    src: faker.image.image(),  // 이미지 url 만들어줌.
  }],
}));
```



