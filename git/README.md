## 주요 Git Command

> **Git 이란?**
>
> Git란 소스코드를 효과적으로 관리하기 위해 개발된 분산형 버전 관리 시스템이다. SVN 같은 형상관리 시스템이다. 그렇다면 SVN 이란? 
>
> 요즘은 Git 이 대세..



![Git Add Example](https://www.jrebel.com/wp-content/uploads/2016/02/GitHub-cheat-sheet-graphic-v1.jpg)





### 기본사용

---

Git 프로젝트를 초기화 `init`  / 로컬 저장소에서 시작

```bash
$ echo "# Hello World" > README.md
$ git init
```



`add` : 새로운 파일이 생기거나 변경되었을 때 인덱스(`stage` 스테이징 이라고 표현도 함)에 반영

```bash
$ git add <변경파일> # 특정 파일에 대한 변경 내역을 알린다.
$ git add . # 모든 변경 내역을 알린다
```

`commit` : 인덱스에 반영된 내역을 로컬 저장소 HEAD에 적용 (인덱스에 존재 하는것만 커밋이 가능하다.)

```bash
$ git commit -m "커밋 메시지"
$ git commit -am "커밋 메시지"
```



`push` :  리모트(원격) 저장소에 내역 반영하기

```bash
$ git push origin master
$ git push -u origin master
$ git push # -u 옵션을 이용하면 다음 push때 이전 히스토리를 기억하고 반영한다.
```



`pull` : 리모트(원격) 저장소에서 로컬 저장소에 가져와 `merge`

```bash
$ git pull
$ git pull <remote> <branch>
```



강제로 `pull` 하기 

```bash
$ git reset --hard HEAD 
$ git pull
```



`clone` : 리모트(원격) 저장소를 복제해오기

````bash
$ git clone <repository url>
````



`fetch` : 리모트(원격) 저장소의 데이터를 로컬에 가져만 오기





### 브랜치

---

> 브랜치란?
>
> 메인 (master) 브랜치에서 자신의 전용 작업 브랜치로 만들어 각자 작업을 진행한 후 메인 브렌치로 변경 사항을 적용 함으로써 다른 사람의 작업에 영향을 받지 않고 독립적으로 수행할 수 있도록 해주는 기능이다.

처음 우리가 생성하는것은 `master branch` 이다.

새로운 branch 만들기

`master branch` 에서 `develop` 라는 새로은 `branch` 만들고 체크아웃 하기

````bash
$ git branch develop # develop 브랜치 만들기
$ git checkout develop # develop 브랜치로 체크아웃 하기
# 한방에 브랜치 만들고 체크아웃 하기
$ git checkout -b develop
````



브랜치 삭제

````bash
$ git brach -d develop
````



특정 브랜치로 부터 새로운 브랜치 만들기

````bash
$ git checkout -b develop origin/qa
````



`master branch` 로 돌아오기 (로컬) : 브랜치 간 이동 `git checkout <branch name>`

````bash
$ git checkout master
$ git branch
* master
  qa
  develop
````



작업한 `branch`로 `push`하기

```bash
$ git push -u origin develop
```



리모트(원격) 저장소에 `branch` 조회

```bash
$ git branch -r
  origin/develop
  origin/master
  origin/qa
```



리모트(원격) 저장소의 `develop` 이라는 `branch`를 로컬 저장소로 가져오기

````bash
$ git checkout -t origin/develop
Branch develop set up to track remote branch develop from origin.
Switched to a new branch 'develop'
$ git branch
* develop
  master
  qa
````



`fatal: Cannot update paths and switch to branch 'develop' at the same time.` 이라는 에러 발생시

리모트(원격) 저장소 갱신하기

```bash
$ git remote update
```



`merge` :  `develop` 을 `master`로 병합

````bash
$ git checkout master
$ git merge develop
````

가끔 병합을 할 때 충돌(`conflict`) 이 나는 경우가 있다. 당황하지 않고... 아래 참조





### Merge 와 Rebase, 그리고 cherry-pick

----

`merge` : 병합하기

리모트(원격) 저장소에서 내가 `pull` 한 이후 다른 작업자가 `push` 한 경우 내가 작업한 내역을 리모트(원격) 저장소에 `push` 할 경우 요청이 거부 된다.

이경우 `merge` 없이 이력을 덮어 쓰면 다른작업자의 `push` 내역이 안드로메다로 날아가 버린다.. =_=;;;

위와 같은 경우가 발생 할 경우 `pull` 한 후 충돌 부분이 있다면 충돌부분을 제거 하고 다시 `commit` > `push` 하면 대부분 손쉽게 해결이 된다.



#### branch merge

`branch` 에서의 `master` 로 `merge` 하기

`issue1` 이라는 `branch` 가 있고 나는 `master` 에 적용해야 하는 상황이다. 그럴경우 `master로` 이동해서 해당 `issue1 branch`를 `merge` 하면된다.

쉽게 말해서 최신소스가 있는 곳(`issue1 branch`)에서 없는 곳으로 병합 하려면 없는곳(`master`)에 가서 최신 소스를 병합(`merge`)하는것이다.

````bash
$ git checkout master
$ git merge issue1
````

하지만 충돌이 안난다고는 안했음.. 간혹 충돌이 발생할 경우가 있다.. 당황하지 않고 해결하자..

충돌 부분은

 `<<<<<<<< HEAD` 이런 화살표로 시작해서 

차이점을 나타내는 `=========` 를 만나게 되고..

`>>>>>>>> issue1`  으로 끝난다..

충돌 부분을 확인 후 수정이나 변경 한 후 위 표시된 부분을 모두 제거 후 다시 진행하면 된다. 변경사항이 있기 때문에 `add` > `commit` > `push` 한다.

````bash
$ git add sample.php
$ git commit -m "issue1 브랜치 통합"
$ git push
````



#### branch rebase

`issue1 branch` 를 `merge` 할 때 `rebase`를 먼저 실행 한 후 `merge` 한다면 그 이력을 하나의 줄기로 만들 수 있다.. 

````bash
$ git checkout issue1
$ git rebase master
````

충돌이 없다면 다행이지만 대부분 충돌을 예상하고 있는게 속편하다. 충돌 부분을 처리 한 후 진행을 계속 한다.

````bash
$ git add sample.php
$ git rebase --continue
````

>  혹 rebase 자체를 취소 하고 싶은 경우 `git rebase --abort` 

이제 `master` 로 가서 `merge` 하고 `rebase` 를 끝낸다.

````bash
$ git checkout master
$ git merge issue1
````





### 태그

---

> 태그란 커밋을 참조하기 쉽게 알기쉬운 이름을 붙이는것이다. 브랜치와는 다르게 위치가 이동하지 않고 고정된다.
>
> 태그는 일반적으로 이름만 갖는 태그와 상세정보를 갖는 주석태그로 나뉜다.

`tag` : 태그 추가하기

````bash
$ git tag <tagname>
$ git tag v.0.1

# 태그 목록 보기
$ git tag
# 태그 목록에 주석 내용 같이 보기
$ git tag -n

# 태그 정보를 포함한 이력 확인하기
$ git log --decorate

# 주석 달린 태그 달기 : 주석을 추가해서 저장해야 한다.
$ git tag -a <tagname>
# 주석을 추가해서 태그 만들기
$ git tag -am <tagname>
$ git tag -am "이것은 v.0.1 태그의 주석이요" v.0.1

# 태그 삭제
$ git tag -d <tagname>
````





### 되돌리기

----

`reset` 파일 변경 사항을 유지하면서 인덱스(`stage`)에서 내리기

````bash
$ git reset <file>
````



`reset` 마지막 커밋으로 되돌리기

````bash
$ git reset --hard
$ git reset --hard HEAD

# 리모트(원격)에 올리지 않은 마지막 commit 최소하기
$ git reset HEAD~
````

`HEAD` 뒤에 물결 표시 `~` 표시로 이전커밋을 나타낸다.

두단계 이전 커밋으로 되돌리고 싶을 경우

````bash
$ git reset --hard HEAD~~
````

실수로 `reset` 한 경우 되돌리기 `ORIG_HEAD`

````bash
$ git reset --hard ORIG_HEAD
````





### 그외

----

`config` : 계정 설정하기

````bash
$ git config -l # 계정정보 확인
# local 설정
$ git config --local user.name "jinyu"
$ git config --local user.email "jinyu.littlefox@gmail.com"
# global 설정
$ git config --global user.name "jinyu"
$ git config --global user.email "jinyu.littlefox@gmail.com"
````



`status` : 상태확인

````bash
$ git status
````



리모트(원격) URL 추가 / 설정

````bash
$ git remote add origin <repository url>
$ git remote set-url origin <repository url>
````



`diff` : 이전 커밋과 비교하기

````bash
$ git diff
````



`log` : 커밋 로그 보기

````bash
$ git log
$ git log --graph
````



이전 커밋에 내용 추가하기 `commit --amend`

````bash
# 파일을 수정하거나 내용 추가 후 이전 커밋에 추가하기
$ git add sample.php
$ git commit --amend

####### 편집 화면
이것 저것 넣기
이전 커밋에 내용 수정해서 통합하기
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Sat May 30 14:48:53 2020 +0900
#
# On branch master
# Changes to be committed:
#       modified:   sample.php


####### :wq 저장하고 나오기

[master ee090cd] 이것 저것 넣기 이전 커밋에 내용 수정해서 통합하기
 Date: Sat May 30 14:48:53 2020 +0900
 1 file changed, 5 insertions(+), 1 deletion(-)
````



끄읕..

참고자료

- [Git 입문하기](https://backlog.com/git-tutorial/kr/intro/intro1_1.html)
- [Git Cheat Sheet](https://www.jrebel.com/system/files/git-cheat-sheet.pdf)


