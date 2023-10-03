# gltf webpack load



## 문제점

Three.js 를 사용해 webpack으로 번들링 된 파일을 webpack dev server로 바로 띄워주려고 할 때,

404 찾을 수 없다는 오류가 계속해서 떴음.

gltf 파일은, `.bin` 파일과 함께 존재하는데, 바이너리 파일을 찾을 수 없다는 오류 였음.

```js
// threejs
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import * as THREE from 'three';
import earth from './assets/3d/earth/scene.gltf';  // here

const scene = new THREE.Scene();
const renderer = new THREE.WebGLRenderer({
  canvas: document.querySelector('#canvas'),
});

const camera = new THREE.PerspectiveCamera(30, 1);

const loader = new GLTFLoader();
loader.load(earth, (gltf) => {  // import 한 earth 파일
  scene.add(gltf.scene);
  renderer.render(scene, camera);
});
```

earth라는 gltf 파일을 webpack으로 load 해서 js에 적용하려고 시도 함.

따라서, webpack.config.js 파일 내부에 file-loader로 불러오기 위해 gltf, bin 을 설정해주고 읽도록 했음.

```js
module: {
  rules: [
    {
      test: /\.(gltf|bin)$/,
      loader: 'file-loader',
      options: {
        name: '[name].[ext]',
        outputPath: './src/assets',
      },
    },
  ]
}
```

문제 해결되지 않음. build 시 dist 폴더 내부에 .gltf 파일 하나만 덩그러니 빌드되는 상황.

<br/>

## 해결방법

GLTFLoader 자체에서 load 해주므로, 절대 경로를 사용해 load 해주면 된다.

```js
const loader = new GLTFLoader();
loader.load('src/assets/3d/earth/scene.gltf', (gltf) => {
  scene.add(gltf.scene);
  renderer.render(scene, camera);
});
```

그러면 GLTFLoader가 해당 경로에 있는 gltf 파일과 같은 위치에 있는 bin 파일을 읽어서 표현해준다.