# Vue Object를 observer가 없는 Object로 만드는 방법

1. `JSON.parse(JSON.stringif($data));`
2. `Object.assign({}, $data);`
3. `{ …$data }`

[https://github.com/vuejs/Discussion/issues/292](https://github.com/vuejs/Discussion/issues/292)
