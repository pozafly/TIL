# toLocaleDateString

사용자의 문화권에 맞는 시간 표기법으로 객체의 시간을 리턴한다.

[여기](https://ichi.pro/ko/tolocaledatestring-eul-sayonghayeo-javascript-naljja-hyeongsig-jijeong-45462506610720)를 참고해봐도 좋을 듯.

```javascript
const today = new Date();
const dateString = today.toLocaleDateString('ko-KR', {
  year: 'numeric',
  month: 'long',
  day: 'numeric',
});

const dayName = today.toLocaleDateString('ko-KR', {
  weekday: 'long',
});

console.log(dateString); // 2021년 3월 5일
console.log(dayName);    // 금요일
```
