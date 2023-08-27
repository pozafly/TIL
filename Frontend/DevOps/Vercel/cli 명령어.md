# cli 명령어

> [출처](https://vercel.com/guides/how-can-i-use-github-actions-with-vercel)

먼저 GitHub Actions에서 소개하고 있는 vercel 배포 flow는 아래와 같음.

1. checkout
2. install vercel cli
   - `npm install --global vercel@latest`
3. Pull Vercel Environment Information
   - `vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}`
4. Deploy Project Artifacts to Vercel
   - `vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}`

가볍게 명령어를 정리하면,

1. `vercel pull`: Vercel로 배포된 프로젝트를 로컬로 가져옵니다. 프로젝트 설정 및 소스 코드를 로컬 환경으로 다운로드하여 수정할 수 있게 해줍니다.
2. `vercel build`: 로컬에서 프로젝트를 빌드합니다. 이 명령어를 사용하여 프로젝트를 배포 가능한 형식으로 미리 빌드할 수 있습니다. 이 단계에서 프로덕션용 번들이 생성됩니다.
3. `vercel deploy`: 빌드된 프로젝트를 Vercel에 배포합니다. 로컬에서 빌드한 프로젝트를 실제 서버에 업로드하여 웹 앱을 배포합니다.

<br/>

## vercel pull

환경 변수 및 프로젝트 설정을 로컬 캐시(`.vercel/.env.$target.local.`)에 저장하는데 사용한다.

<img width="258" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/a716ff43-a6bc-46f9-a826-888ca55f0272">

이렇게 `npm i -g vercel` 후, `vercel pull`을 받으면 환경 변수를 가져온다.

<img width="875" alt="스크린샷 2023-08-27 오후 6 05 36" src="https://github.com/pozafly/TIL/assets/59427983/fafe7379-c369-4661-9e76-c4d3df09df4b">

이런 명령을 사용하지 않는 경우 vercel pull을 실행할 필요가 없다. -> GitHub Actions에서는 매번 환경변수가 필요하기 때문에 가져와 주어야 한다.

버셀에서 환경 변수 또는 프로젝트 설정이 업데이트된 경우, vercel pull을 다시 사용해 로컬 환경 변수 및 프로젝트 설정 값을 `.vercel/` 에서 업데이트해야함.

<br/>

## vercel build

빌드 아티팩트는 빌드 출력 API에 따라 `.vercel/output` 디렉토리에 배치된다. 

> 여기서 `Artifacts` 라는 용어가 등장하는데, 이는 `환경 변수` + `컴파일된 코드` 를 빌드 해 탄생하는 자료들을 뜻한다고 한다. 즉, Artifacts 라는 용어는 빌드 결과물인 것이다. `output` 아래 생성되는 모든 파일들을 뜻하는 단어임.

<img width="244" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/ee94fc4f-5b12-456c-a1c8-92eb073b38d5">

생성되는 파일은, Next.js 소스 코드가 전부 빌드된 모습을 볼 수 있다. 그렇다면 `vercel build` 명령어는 내부적으로 또 어떤 빌드 명령어를 수행하나?

<img width="439" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/dd68dde9-d272-487c-b319-5e9fb9403e36">

`husky` 를 실행하는 것을 보아, git hook이 실행 되었는데, 패키지를 다운 받는다. 위 사진을 보면, package-lock.json 파일을 기준으로 패키지를 다운 받고 있는 것을 볼 수 있음.

따라서, `vercel build` 명령어에서 패키지를 다운 받고 있으므로 `npm ci` 혹은 `npm install` 명령어는 GitHub Actions에는 따로 필요가 없다.

<br/>

## vercel deploy

단순 배포 명령어임. `--prebuilt` 명령어를 붙여주고 있는데, 로컬에서 빌드된 파일을 **Vercel에 직접 업로드** 하는 옵션이다. Vercel 서버에서 다시 빌드하는 단계를 생략하는 옵션. 이로 인해 배포 시간을 단축시킬 수 있다.

기본적으로 `vercel deploy` 명령어는 vercel 서버에서 프로젝트를 빌드한 후 배포하는데 `--prebuilt` 옵션을 사용하면 로컬에서 빌드된 파일을 업로드 하기 때문에 속도를 높여주는 옵션인 것임.





















































