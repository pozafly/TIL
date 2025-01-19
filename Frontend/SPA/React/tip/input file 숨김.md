# Input File 숨김

<img width="168" alt="스크린샷 2021-03-02 오후 3 32 30" src="https://user-images.githubusercontent.com/59427983/109608022-89972300-7b6c-11eb-84b1-142aa70a9644.png">

`<input type="file" />` 을 하게되면 사진과 같이 엄청 못생김. 따라서 button을 이용해서 이쁘게 만들고 button을 클릭하면 input file을 클릭한 것과 같은 효과를 내게 만들 것임.

```jsx
import React, { useRef } from 'react';
import styles from './image_file_input.module.css';

const ImageFileInput = ({ name }) => {
  const inputRef = useRef();
  const onButtonClick = () => {
    inputRef.current.click();   // ref를 사용해 click 효과를 낸다.
  };
  return (
    <div className={styles.container}>
      <input
        ref={inputRef}
        className={styles.input}
        type='file'
        accept='image/*'
        name='file'
      />
      <button className={styles.button} onClick={onButtonClick}>
        {name || 'No file'}
      </button>
    </div>
  );
};

export default ImageFileInput;
```

이렇게 하고 css에서는 `<input type="file" />` 이녀석을 display: none; 으로 두면 됨.
