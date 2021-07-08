# loading spinner

이미지가 업로드 되거나 다른 비동기 작업 중 기다려야 할 때, loading spinner를 추가할 수 있다.

## 1. state 만들기

```jsx
const [loading, setLoding] = useState(false);
...
const onChange = async (event) => {
  setLoding(true);   // 보여주고
  const uploaded = await 비동기 작업...
  setLoding(false);  // 비동기 작업 끝나면 없애고
}
```

## 2. UI 마크업

```jsx
{!loading && (
 <button className={styles.button} onClick={onButtonClick}>
   {name || 'No file'}
 </button>
 )}
 {loading && <div className={styles.loading}></div>}
```

로딩 중일 때는 div 태그를 보여주고, 로딩이 끝나면 다시 button을 만들어준다.

## 3. CSS 작업

```css
.loading {
  width: 1.5em;
  height: 1.5em;
  border-radius: 50%;
  border: 3px solid makerLightGrey;
  border-top: 3px solid makerPink;
  animation: spin 1s linear infinite;   // 여기서 spin은 밑에 keyframes 에 정의 됨.
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
```

이렇게 로딩 스피너를 만들어줬다.