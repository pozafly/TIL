# Lighthouse 지표

> [출처](https://velog.io/@iberis/%EC%9B%B9-%EC%B5%9C%EC%A0%81%ED%99%94-Lighthouse-CLI-%EC%84%B1%EB%8A%A5-%EC%A7%80%ED%91%9C), [출처2](https://nitropack.io/blog/post/lighthouse-10)

- First Contentful Paint
- Largest Contentful Paint
- Total Blocking Time
- Cumulative Layout Shift
- Speed Index

## 가중치

Time To Interactive는 lighthouse10 버전에서 제거되었다. TTI 특정 시점을 잡아내는게 네트워크 요청과 작업 단위에 너무 의존적이기 때문에 제거 되었음. 차라리 LCP(Largest Contentful Paint) 지수를 보는게 더 낫다. 또한, TTI 지표 가중치는 CLS로 이전되었음. ([출처](https://stackoverflow.com/questions/75774948/time-to-interactive-parameter-is-not-showing-anymore-in-google-pagespeed-insig))

![image](https://github.com/pozafly/TIL/assets/59427983/6c10f433-4421-43ba-914f-4b15c9cf622b)

Lighthouse 6 지표 가중치. LCP, TBT이 가장 중요해보인다.

또한, 아래는 Lighthouse 10의 가중치이다.

<img width="581" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/ee1f6e02-37a5-42ce-b402-bb0abecdcff8">

TBT가 가장 높으며, CLS가 증가했고, 동급으로 LCP 순이다.

구글이 결정을 내린 이유는 [다음과 같다.](https://developer.chrome.com/blog/lighthouse-10-0/#:~:text=TTI marks a,in the field.)

> TTI는 특정 시점을 표시하지만 비정상적 네트워크 요청과 긴 작업에 지나치게 민감하다. LCP 및 속도 지수는 일반적으로 활성 네트워크 요청 수보다 로드된 느낌의 페이지 콘텐츠에 대한 더 나은 경험적 방법이다. 한편 TBT은 긴 작업과 메인 스레드 가용성을 강력하게 처리하고, 직접적 프록시는 아니지만 현장에서 측정된 코어 웹 바이탈과 더 나은 상관관계를 보이는 경향이 있다.
>
> 즉, 사용자가 페이지에 도착 후 페이지가 완전히 상호작용 하는데 걸리는 시간을 측정하는 더 우수 항목이 TBT다.
