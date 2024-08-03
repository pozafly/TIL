# SSR 만들기

> https://github.com/pozafly/render-route

node express server를 먼저 만든다.

```js
import express from 'express';
import cors from 'cors';
import movies from './movie.json' assert { type: 'json' };
import fs from 'fs';

const app = express();
const port = 3000;

app.use(cors());
app.use(express.static('dist'));

app.get('/', (req, res) => {
  res.send('test');
});

app.get('/search', (req, res) => {
  const filteredMovies = movies.filter((movie) =>
    movie.title.toLowerCase().includes(req.query.query.toLowerCase())
  );
  res.json(filteredMovies);
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

`app.use(express.static('dist'));` 이 부분은 dist 디렉토리 내부에 있는 static 파일을 웹 서버로 서빙하겠다는 뜻이다.

vite로 프로젝트를 만들고 build를 돌린다. 그럼 dist 디렉토리에 index.html 파일이 생성될 것이다.

express server를 `nodemon` 에 연결한다. 그리고 localhost:port 로 접속하면 `app.get('/')` 이 녀석 때문에 경로가 겹치므로 index.html을 main.html로 바꿔 localhost:port/main.html 이 메인 페이지가 되도록 변경한다.

```json
{
  "scripts": {
    "build": "vite build && mv dist/index.html dist/main.html",
    ...
  }
}
```

그리고 요청하면 정상적으로 html 파일을 서빙하게 된다. 하지만, network tab에 docs를 보면 여전히 `<div id="app"><div>` 밖에 없을 것이다.

이 때 dist로 빌드된 html 파일에 

```html
<body>
  <div id="app"><!--app--></div>
</body>
```

이런 식으로 주석을 넣고, server.js에

```js
app.get('/', (req, res) => {
  fs.readFile('dist/main.html', (err, file) => {
    res.send(file.toString().replace('<!--app-->', 'hello'));
  });
});
```

이런식으로 파일을 읽어와 replace로 교체하면 

<img width="552" alt="스크린샷 2024-08-03 오후 6 44 44" src="https://github.com/user-attachments/assets/bb57659e-88f1-4dc2-aa40-41a25c102f1c">

이렇게 SSR 되어 떨어지는 것을 볼 수 있다.

이제, 함수로 이 작업을 해보자.

```js
// index.js
export function renderIndex() {
  document.querySelector('#app').innerHTML = `
    <h1>Movie Info</h1>
    <form>
      <input type="text" name="query" placeholder="Search for a movie" />
      <button type="submit">Search</button>
    </form>
  `;

  document.body.querySelector('form').addEventListener('submit', (event) => {
    event.preventDefault();
    goto(`/search?query=${event.target.query.value}`, { push: true });
  });
}
```

이 파일은 client side rendering 할 수 있는 코드다.

```js
import { goto } from '../router';

export function getInitialHTML() {
  return `
    <h1>Movie Info</h1>
    <form>
      <input type="text" name="query" placeholder="Search for a movie" />
      <button type="submit">Search</button>
    </form>
  `;
}

export function renderIndex() {
  document.querySelector('#app').innerHTML = getInitialHTML();

  document.body.querySelector('form').addEventListener('submit', (event) => {
    event.preventDefault();
    goto(`/search?query=${event.target.query.value}`, { push: true });
  });
}
```

이렇게 `getInitialHTML` 함수를 만들어 넣는다.

그러면 getInitialHTML 함수를 server.js에서 import 해서 html을 내려주는 부분에 replace 하면 되지 않을까? 하지만 불가능 하다.

왜냐하면 getInitialHTML 함수는 빌드를 해야만 main.js에서 dist 폴더로 이동할 것이고, dist로 동적으로 생성되는 파일을 server.js에서 import 할 수 없다.

그러면 `./src/pages/index.js` 와 같은 경로로 import 하기에는 함수 수가 너무 많을 것이기 때문에 어렵다. 프로매틱하지 않고 사람 손으로 계속 직접 해주어야 하기 때문이다.

또한, export const ... 한 함수는 트리 쉐이킹으로 client 쪽 빌드를 실행할 경우 코드 자체가 사라져버린다.

정리하면

- text 대체 함수는 dist폴더로 빌드됨.
- src 하위에 있는 함수는 사람 손으로 import를 각각 해주어야 함.
- 트리 쉐이킹으로 server.js에서 import 하는 코드는 빌드시 사라짐.

이 때문에 client 쪽 함수는 server.js에서 import 가 어렵다는 결론.

---

이 상황에서 할 수 있는 것은, vite의 library mode를 사용하는 것이다.

```js
// vite.config.js
import { resolve } from 'path';

export default {
  build: {
    lib: {
      entry: resolve(__dirname, 'main.js'),
      name: 'index',
      fileName: 'index',
    },
  },
};
```

이렇게 vite.config.js에 lib를 넣어주자. 빌드 실패나면

package.json에 mv로 index.html을 main.html로 변경했던 것을 지워주자. lib 모드를 사용하니, treeshaking이 되지 않는 것을 볼 수 있다.

```js
// routes.js
import { renderIndex, getInitialHTML as getInitialHTMLForIndex } from './pages/index';
import { renderSearch, getInitialHTML as getInitialHTMLForSearch } from './pages/search';

export const routes = {
  '/': renderIndex,
  '/search': renderSearch,
};

export const getInitialHTML = {
  '/': getInitialHTMLForIndex,
  '/search': getInitialHTMLForSearch,
};
```

이제, 이렇게 만들어줄 수 있다. 이 때 search 같은 경우는 

```js
export async function renderSearch({ searchParams }) {
  document.querySelector('#app').innerHTML = `
    <h1>Search Results</h1>
    <p>Searching for: ${searchParams.query}...</p>
  `;
  ...
}
```

이런 형태였지만, 

```js
export const getInitialHTML = () => {
  return `
    <h1>Search Results</h1>
    <p>Searching for: ...</p>
  `;
};
```

이런 형태로 `searchParams.query` 부분을 없애고 넣었다. getInitialHTML을 export 했으니 build의 진입점인 main.js에서 export 하자.

```js
// main.js
import { start } from './src/router';
import { routes, getInitialHTML } from './src/routes';

export { getInitialHTML };

start({ routes });
```

빌드 결과를 보면

<img width="285" alt="image" src="https://github.com/user-attachments/assets/9ca83b8b-650c-471b-848d-dab7b54d0a97">

위와 같음. dist/index.js에서 잘 export 되었다.

```js
// server.js
import { getInitialHTML } from './dist/index.js';

app.get('/', (req, res) => {
  fs.readFile('index.html', (err, file) => {
    res.send(file.toString().replace('<!--app-->', getInitialHTML['/']));
  });
});
```

<img width="533" alt="image" src="https://github.com/user-attachments/assets/7602e6bf-c403-4184-b471-d148755a24e8">

자 이렇게 첫 렌더링 시 HTML이 잘 입혀서 들어온 것을 볼 수 있다.

<br/>

## data fetch for SSR

```js
app.get('/api/search', (req, res) => {
  const filteredMovies = movies.filter((movie) =>
    movie.title.toLowerCase().includes(req.query.query.toLowerCase())
  );
  res.json(filteredMovies);
});

app.get('/search', (req, res) => {
  fs.readFile('index.html', (err, file) => {
    res.send(file.toString().replace('<!--app-->', getInitialHTML['/search']));
  });
});
```

기존 /search를 /api/search 로 바꾸어주고, getInitialHTML을 넣어주면 

```html
<body>
  <div id="app">
    <h1>Search Results</h1>
    <p>Searching for: ...</p>
  </div>
	<script type="module" src="/index.js"></script>
</body>
```

이렇게 SSR 잘 되는 것을 볼 수 있다. 하지만, fetching 된 데이터는 브라우저에서 페칭된 것이다. 서버에서 이걸 불러서 사용할 수는 없을까?

```js
const getFiltedMovies = (query) => {
  return movies.filter((movie) =>
    movie.title.toLowerCase().includes(query.toLowerCase())
  );
};

app.get('/api/search', (req, res) => {
  res.json(getFiltedMovies(req.query.query));
});

app.get('/search', (req, res) => {
  const filteredMovies = getFiltedMovies(req.query.query);
  fs.readFile('index.html', (err, file) => {
    res.send(
      file.toString().replace(
        '<!--app-->',
        getInitialHTML['/search']({
          movies: filteredMovies,
        })
      )
    );
  });
});
```

`getFiltedMovies()` 함수를 만들어 페이지와 api 모두에 적용했다. 그리고, getInitialHTML 함수에 이를 넣어주었다. 즉, 페이지 요청이 있을 때, fetch 하여 내용을 담아 getIntialHTML로 넘겼다.

그럼 search.js에는

```js
// search.js
export const getInitialHTML = ({ movies } = {}) => {
  if (movies) {
    return `
      <h1>Search Results</h1>
      <ul>
        ${movies.map((movie) => `<li>${movie.title}</li>`).join('')}
      </ul>
    `;
  } else {
    return `
      <h1>Search Results</h1>
      <p>Searching for: ...</p>
    `;
  }
};
```

이와 같이 template literal로 html을 만들고, 다시 server.js로 넘어와 html을 생성 후 뿌리게 되는 것이다.

<img width="561" alt="image" src="https://github.com/user-attachments/assets/8a4105e2-a3da-4fec-9974-d6b8d40bbc5e">

하지만, 여전히 브라우저에서는 fetch가 일어나고 있다.

<br/>

## Hydration

```js
export const getInitialHTML = ({ movies } = {}) => {
  if (movies) {
    return `
      <h1>Search Results</h1>
      ${movies
        .map(
          (movie) => `
            <div class="movie">
              <p>${movie.title}</p>
              <button type="button">click</button>
            </div>
          `
        )
        .join('')}
    `;
  } else {
    return `
      <h1>Search Results</h1>
      <p>Searching for: ...</p>
    `;
  }
};

export async function renderSearch({ searchParams }) {
  document.querySelector('#app').innerHTML = `
    <h1>Search Results</h1>
    <p>Searching for: ${searchParams.query}...</p>
  `;

  const response = await fetch(
    `${import.meta.env.DEV ? 'http://localhost:3000' : ''}/api/search?query=${
      searchParams.query
    }`
  );
  const movies = await response.json();

  document.querySelector('#app').innerHTML = getInitialHTML({ movies });
  Array.from(document.querySelectorAll('.movie button')).forEach((button) =>
    button.addEventListener('click', (event) => {
      console.log('click');
    })
  );
}
```

이렇게 버튼을 넣고, 각 button에 addEventListener를 달아주었다.

하지만, 명심해야할 것은 브라우저는 모든 데이터가 입혀진 채 HTML을 받았지만, 다시 fetch 하고 있다.

server.js에 initial data를 script 태그로 넣어줘보자.

```js
app.get('/search', (req, res) => {
  const filteredMovies = getFiltedMovies(req.query.query);

  fs.readFile('index.html', (err, file) => {
    res.send(
      file.toString().replace(
        '<!--app-->',
        `<script>
          window.__INITIAL_DATA__ = ${JSON.stringify({
            movies: filteredMovies,
          })}
        </script>` +
          getInitialHTML['/search']({
            movies: filteredMovies,
          })
      )
    );
  });
});
```

<img width="600" alt="image" src="https://github.com/user-attachments/assets/568dd4ce-4027-49fd-949b-ede728a307c4">

이 작업은 브라우저에서 뭔가를 사용할 수 있게끔 뭔가늘 넣어준 것이다.

main.js에서 이걸찍어보면

```js
if (typeof window !== 'undefined') {
  console.log('initial data', window.__INITIAL_DATA__);
  start({ routes });
}
```

정상적으로 들어오는 것을 확인할 수 있음. 전역적으로 사용할 수 있으니,

```js
goto(location.pathname + location.search, {
  initialData: window.__initial_DATA__,
});
```

goto 함수에서 받아, routes로 넘겨주고, search.js로 넘겨주면, 

```js
export async function renderSearch({ searchParams, initialData }) {
  if (!initialData) {
    document.querySelector('#app').innerHTML = `
      <h1>Search Results</h1>
      <p>Searching for: ${searchParams.query}...</p>
    `;

    const response = await fetch(
      `${import.meta.env.DEV ? 'http://localhost:3000' : ''}/api/search?query=${
        searchParams.query
      }`
    );
    const movies = await response.json();

    document.querySelector('#app').innerHTML = getInitialHTML({ movies });
  }

  Array.from(document.querySelectorAll('.movie button')).forEach((button) =>
    button.addEventListener('click', (event) => {
      console.log('click');
    })
  );
}
```

initialData가 없을 경우는 fetch 해서 rendering 하도록 하고, `addEventListener` 는 어떤 경우라도 실행되도록 하는 것이다.

<img width="182" alt="image" src="https://github.com/user-attachments/assets/21ce19b5-86f8-4cca-8d95-61100707f2df">

이제 network로 initialData가 존재하니 다시 fetch 하지 않는다. 단지 index.js 파일을 불러오고, `addEventListener` 가 실행되면서 Hydration이 되는 것이다.

---

여기까지가 Hydration 과정이다.

우리는 innerHTML을 건너 뛰었지만, react에서는 건너뛰지 않는다. React는 가상 DOM을 이용하기 때문에 렌더링을 하는데, 바뀐게 없으면 깜빡 거리지 않고 화면을 painting 하지 않기 때문이다. Next.js 같은 경우는 만약 서버에서 그린 HTML과 클라이언트에서 그린 HTML에 차이가 있으면 missmatch error를 나타내주는 기능도 포함되어 있다.

