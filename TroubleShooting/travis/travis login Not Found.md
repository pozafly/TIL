# Travis login Not Found

vue 배포 자동화를 위해 travis 설정파일을 만들다가, vue의.env 파일을 gitignore 시켜놨기때문에 배포가 실패했다. 그러는 도중, env 파일을 암호화 하여 사용하기 위해 travis CI의 CLC(Command Line Client)를 이용하기 위해 터미널에
```bash
$ travis login --pro
```
명령어를 때리고, github의 id, pw 를 입력했더니 다음과 같은 오류문을 뱉더라.

![[assets/images/4b68335029ed4d2945a41b3a4145fe59_MD5.png]]

not found 분명 몇번이고 해도 안됨. 그러다가 여길 발견했음. [stackoverflow](https://stackoverflow.com/questions/65313155/i-have-an-github-account-and-my-user-credintials-are-true-however-i-can-not-lo). 하는 말이 travis login에서 github token을 사용하란다. [github token 생성 페이지](https://github.com/settings/tokens)에 들어가서 token을 만들어주자.

![[assets/images/3ba0331362f2b34f7b568a85bd0db9ca_MD5.png]]

notifications, repo 두개만 체크해주고 진행. 만들면 클립보드에 자동으로 token이 복사됨. 그담에 터미널 창에 `travis login --pro --github-token [토큰]` 이렇게 적어주면 로그인 성공!
