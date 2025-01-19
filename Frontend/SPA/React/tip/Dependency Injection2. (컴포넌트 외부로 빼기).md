# 컴포넌트 외부로 빼기(Dependency Injection)

특정 컴포넌트가 여러 개의 컴포넌트에서 사용되고 있다면, 컴포넌트 자체를 외부로 빼서 사용하는게 효과적일 때가 있다.

예를들어, imageLoad를 사용할 때, 이미지를 업로드 하는 기능을 외부 js인, `image_upload.js` 파일로 빼서 만들었다면, 이 기능을 쓰기 위해서 app.jsx에서부터 객체화 시켜 DI하면 이어지는 컴포넌트에 props로 전달해야하는 불편함이 생긴다.

그리고 props로 전달된 image_upload.js 객체는 이것을 사용하는 `<ImageUpload />` 라는 컴포넌트에서 사용되는데, 이 컴포넌트도 마찬가지로 여러번 생성되어 props로 전달 되어야 할 것이다.

따라서 `<ImageUpload />` 라는 컴포넌트를 외부에서 1번 생성하여 props로 내려주는 형태로 작성하는 것이 좋다. 해당 컴포넌트에 새로운 service가 생겼을 때 바로 그곳에서만 고치면 되기 때문에 유지 보수 측면에서도 좋음.

## 1. Service js 파일 생성

```javascript
// image_uploader.js
class ImageUploader {
  async upload(file) {
    return 'file';
  }
}
export default ImageUploader;
```

이렇게 임의의 upload() 함수를 만들어두자.

```jsx
// index.js
const imageUploader = new ImageUploader();   // image_upload.js 객체 생성
const FileInput = (props) => (   // 컴포넌트 객체 생성(컴포넌트 객체이므로 대문자 사용)
  // 1. {...props} 라는 props을 복사해서 내려줌.
  // 2. 그리고 image_upload.js 생성된 객체를 내려줌
  <ImageFileInput {...props} imageUploader={imageUploader} />
);

ReactDOM.render(
  <React.StrictMode>
    <App FileInput={FileInput} />   // 여기 app으로 내려줌.
  </React.StrictMode>,
  document.getElementById('root')
);
```

`{…props}` 을 같이 내려주는 이유는, `<ImageFileInput />`이라는 컴포넌트에서 기능을 완성해서 내려주면,(`{…props}` 없이 컴포넌트 자체를 내려주면) `<ImageFileInput />` 라는 컴포넌트에 onClick 이라던지 다른 props을 전달하고 싶을 때 불가능 하게 됨. 즉, 컴포넌트를 사용하는 사람이 더 이상 컴포넌트에 기능을 추가할 수 없게 되는 말임. 확장성 down.

📌 하지만 여기서 궁금증이 있을 수 있는데 어짜피 여기서 생성해서 가장 말단까지 내려주면 다른 곳에서 props으로 전달하는거랑 뭐가 달라?

- 만약 const FileInput 이 다른 서비스나 다른 것은 DI 받는다면 다른 곳에서 받는 것 보다 `<App FileInput={FileInput} />` 이곳에서 확장해서 사용하는것이 좋기 때문.

<br/>

## 2. 컴포넌트를 내려줌

```jsx
function App({ FileInput, authService }) {
  return (
    ...
    <Maker FileInput={FileInput} authService={authService} />
    ...
  );
}
```

이런식으로 계속 이어지는 부분에 컴포넌트 자체를 props로 내려주자. 보통 컴포넌트 props의 첫글짜는 **대문자**로 작성하며, props 의 가장 앞에 둔다.

<br/>

이렇게 사용했을 때의 장점

- 쓸데없이 많은 서비스를 전달하지 않아도 된다.
- 내려준 컴포넌트에서 더 많은 서비스가 필요하다면 해당 컴포넌트에서만 수정하면 된다. props로 서비스를 또 내려주지 않고.

예를 들면,

- 만약 FileInput 컴포넌트를 만들기 위해서 youtubeService, mediaService, imageFetcher 라는 세가지의 오브젝트를 전달해야 한다면,
- `<FileInput youtubeService={youtubeService} mediaService={mediaService} imageFetcher={imageFetcher}>` 이렇게 FileInput을 만들기 위해서는 최상위 부모로 부터 저 세가지의 서비스들을 모두 props으로 전달해야 한다.
- 하지만 최상위 컴포넌트 밖에서 먼저 FileInput 컴포넌트를 만들어서 주입해주게 되면 FileInput에만 필요한 모든 서비스들을 일일이 props으로 전달해 주지 않아도 된다.
