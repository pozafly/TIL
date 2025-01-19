# Fork 해온 repo 잔디

https://soranhan.tistory.com/11

## 1. 일단 내 git hub에 새로운 레파지토리를 만든다

## 2. Terminal을 연다

## 3. 복사하고자 하는 repository를 bare clone한다

- 👉 복사하고자 하는 repository에 가서 저 카피 버튼을 누르면 repository 주소가 복사된다.

> $ git clone --bare https://github.com/exampleuser/old-repository.git

- 👉 터미널에 git clone --bare를 입력하고 그 뒤에 복사해온 주소를 붙여넣기한다.

## 4. 새로운 레파지토리로 Mirror-push

> $ cd old-repository.git
> $ git push --mirror https://github.com/exampleuser/new-repository.git

- 👉 cd 명령어를 이용해 새로운 레파지토리로 이동한뒤
- 👉 3번에서 한 것과 같이 새로운 레파지토리 주소를 복사한뒤, git push --mirror 뒤에 붙여넣으면 Mirror-push가 된다.

## 5. 처음에 임시로 생성했던 local repository를 삭제

> $ cd..
> $ rm -rf old-repository.git
