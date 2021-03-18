# 아이콘 서비스 gravatar

http://gravatar.com/

이곳에 가면 아래 url 을 입력하면 무료로 아바타 아이콘을 만들어준다. 아래와 같이 url 을 사용하면 됨.

```json
import md5 from 'md5';
// ...
photoURL: `http://gravatar.com/avatar/${md5(createdUser.user.email)}?d=identicon`
```

이렇게 사용하는데, 여기서 md5는 UUID와 같이 랜덤한 숫자를 생성해주는 라이브러리임. 매개 변수로 유니크한 값을 넣어줘야 한다.

```sh
$ npm install md5 --save
```

로 깔아서 사용할 수 있다. gravatar에서는 공식적으로 md5를 통해서 생성하라고 나와 있다.

