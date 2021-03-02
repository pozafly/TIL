# SDK

<img width="681" alt="스크린샷 2021-03-02 오후 8 34 34" src="https://user-images.githubusercontent.com/59427983/109642768-bd873e00-7b96-11eb-8705-948c41b5d417.png">

파이어 베이스의 warning 이다. 해석하면 firebase앱을 production에 배포하면 필요한 것만 import 해서 사용하는 것이 추천되어진다. 즉, 쓰지도 않는데 전부다 import 했다는 뜻. 그럼 firebase.js 를 한번 보자.

```javascript
// firebase.js
import firebase from 'firebase';

const firebaseConfig = {
  apiKey: process.env.REACT_APP_FIREBASE_API_KEY,
  authDomain: process.env.REACT_APP_FIREBASE_AUTH_DOMAIN,
  databaseURL: process.env.REACT_APP_FIREBASE_DB_URL,
  projectId: process.env.REACT_APP_FIREBASE_PROJECT_ID,
};
// Initialize Firebase
const firebaseApp = firebase.initializeApp(firebaseConfig);
export default firebaseApp;
```

`import firebase from 'firebase';` 이 import 구문에서 보면 벌써, 모든 것을 다 들고 왔다. 지금 하고 있는 플젝에서는 인증(auth), 데이터베이스(database) 만 사용하므로, firebase를 쓸 수 있는 'firebase/app'과 나머지만 임포트 하면 된다.

```javascript
import firebase from 'firebase/app';
import 'firebase/auth';
import 'firebase/database';
...
export const firebaseAuth = firebaseApp.auth();
export const firebaseDatabase = firebaseApp.database();
export const googleProvider = new firebase.auth.GoogleAuthProvider();
export const githubProvider = new firebase.auth.GithubAuthProvider();
```

이렇게.

또 이걸 쓰고 있는 파일들을 보자.

```javascript
// auth_service.js
import firebase from 'firebase';
import firebaseApp from './firebase';

이걸 

import { firebaseAuth, githubProvider, googleProvider } from './firebase';
```

이렇게 바꿔서 쓰면 됨.

```javascript
// card_repository.js
import firebaseApp from './firebase';

이걸 

import { firebaseDatabase } from './firebase';
```

이렇게 바꿔서 필요한 것만 import 해서 쓰면 된다.