### CodeSniffer 사용하기

---

##### 1. 설치

````bash
# 글로벌 설치를 할 경우
composer global require "squizlabs/php_codesniffer=*"

# 해당 프로젝트에서 설치 할 경우
composer require-dev "squizlabs/php_codesniffer=*"

# composer.json 확인

{
    "require-dev": {
        "squizlabs/php_codesniffer": "3.*"
    }
}
````



##### 2. 사용법

> 검사 파일의 줄번호와 함께 발견되는 규칙위반 메시지를 출력한다.
>
> 해당 메시지에 대한 정보는 [여기](https://ujuc.github.io/2019/02/05/psr-2:_coding_style_guide/)를 참조
>
> [command line usage](https://www.drupal.org/docs/8/modules/code-review-module/php-codesniffer-command-line-usage)

````bash
# 단일 파일 검사
$ phpcs /path/to/code/myfile.php
FILE: myfile.php
--------------------------------------------------------------------------------
FOUND 24 ERRORS AND 1 WARNING AFFECTING 11 LINES
--------------------------------------------------------------------------------
  2 | ERROR   | [x] There must be one blank line after the namespace
    |         |     declaration
  3 | ERROR   | [x] Each PHP statement must be on a line by itself
  3 | ERROR   | [x] There must be one blank line after the last USE statement;
 17 | ERROR   | [x] Expected 1 newline at end of file; 0 found
--------------------------------------------------------------------------------
PHPCBF CAN FIX THE 25 MARKED SNIFF VIOLATIONS AUTOMATICALLY
--------------------------------------------------------------------------------

Time: 19ms; Memory: 3.25Mb

# 해당 디렉토리 검사
$ phpcs /path/to/code-directory

# PSR-2 코딩 표준과 비교하여 코드를 확인하려면 --standard 옵션사용
$ phpcs --standard=PSR2 /path/to/code-directory
````



##### 3. 수정하기

````bash
# 일반적인 규칙을 적용하여 수정
$ phpcbf /path/to/code/myfile.php

# PSR-2 코딩 표준으로 소스 수정을 하려면 --standard 옵션사용
$ phpcbf /path/to/code/myfile.php --standard=PSR2

PHPCBF RESULT SUMMARY
-----------------------------------------------------------------------------------------------
FILE                                                                            FIXED REMAINING
-----------------------------------------------------------------------------------------------
/Users/jinyu/git/designpattern/Behavioral/templatemethod/AbstGameConnectHelper.php     1      0
-----------------------------------------------------------------------------------------------
A TOTAL OF 1 ERROR WERE FIXED IN 1 FILE
-----------------------------------------------------------------------------------------------

Time: 65ms; Memory: 6MB
````



##### 음... Git에 올리기전에 검사를 해서 올릴 수 있을거 같아서 찾아보았다..

> 그래서 찾아봤다 hook을 이용한 pre-commit 설정으로 commit 단계에서 검사 스크립트를 작성해서 적용하면 된다.

````bash
$ git commit -m "confirm pre-commit"
Checking PHP Lint...
No syntax errors detected in /Users/jinyu/git/designpattern/Behavioral/strategy/DateComparator.php
No syntax errors detected in /Users/jinyu/git/designpattern/Behavioral/strategy/GameCharactor.php
Running Code Sniffer. Code standard PSR2.
EE 2 / 2 (100%)


FILE: ...s/jinyu/git/designpattern/Behavioral/strategy/DateComparator.php
----------------------------------------------------------------------
FOUND 1 ERROR AFFECTING 1 LINE
----------------------------------------------------------------------
 24 | ERROR | [x] Expected 1 blank line at end of file; 3 found
----------------------------------------------------------------------
PHPCBF CAN FIX THE 1 MARKED SNIFF VIOLATIONS AUTOMATICALLY
----------------------------------------------------------------------


FILE: ...rs/jinyu/git/designpattern/Behavioral/strategy/GameCharactor.php
----------------------------------------------------------------------
FOUND 1 ERROR AFFECTING 1 LINE
----------------------------------------------------------------------
 2 | ERROR | [x] There must be one blank line after the namespace
   |       |     declaration
----------------------------------------------------------------------
PHPCBF CAN FIX THE 1 MARKED SNIFF VIOLATIONS AUTOMATICALLY
----------------------------------------------------------------------

Time: 68ms; Memory: 6MB

Fix automatically (where possible)? [y/N]
y
automagically fixing files

PHPCBF RESULT SUMMARY
------------------------------------------------------------------------------------------
FILE                                                                      FIXED  REMAINING
------------------------------------------------------------------------------------------
/Users/jinyu/git/designpattern/Behavioral/strategy/DateComparator.php     1      0
/Users/jinyu/git/designpattern/Behavioral/strategy/GameCharactor.php      1      0
------------------------------------------------------------------------------------------
A TOTAL OF 2 ERRORS WERE FIXED IN 2 FILES
------------------------------------------------------------------------------------------

Time: 74ms; Memory: 6MB


re-staging updated files
[master eac99c1] confirm pre-commit

````

###### 설정을 해보자

````bash
$ cd .git/hooks
$ .git/hooks
$ ll
total 112
-rwxr-xr-x  1 jinyu  staff   478B May 15 23:34 applypatch-msg.sample
-rwxr-xr-x  1 jinyu  staff   896B May 15 23:34 commit-msg.sample
-rwxr-xr-x  1 jinyu  staff   3.2K May 15 23:34 fsmonitor-watchman.sample
-rwxr-xr-x  1 jinyu  staff   189B May 15 23:34 post-update.sample
-rwxr-xr-x  1 jinyu  staff   424B May 15 23:34 pre-applypatch.sample
-rwxr-xr-x  1 jinyu  staff   1.6K May 15 23:34 pre-commit.sample
-rwxr-xr-x  1 jinyu  staff   416B May 15 23:34 pre-merge-commit.sample
-rwxr-xr-x  1 jinyu  staff   1.3K May 15 23:34 pre-push.sample
-rwxr-xr-x  1 jinyu  staff   4.8K May 15 23:34 pre-rebase.sample
-rwxr-xr-x  1 jinyu  staff   544B May 15 23:34 pre-receive.sample
-rwxr-xr-x  1 jinyu  staff   1.5K May 15 23:34 prepare-commit-msg.sample
-rwxr-xr-x  1 jinyu  staff   3.5K May 15 23:34 update.sample

# 여기서 pre-commit.sample 을 참고하면 되지만 그냥 pre-commit 을 만들고 실행 권한을 주었다.

````



###### 아래 코드는 찾아보니 있더라.. 약간 수정으로 사용가능.. [참고](https://robjmills.co.uk/2018/01/14/automatic-psr2-coding-standard.html)

````bash
$ vi pre-commit

#!/bin/bash

PROJECT=`php -r "echo dirname(dirname(dirname(realpath('$0'))));"`
STAGED_FILES_CMD=`git diff --cached --name-only --diff-filter=ACMR HEAD | grep \\\\.php`

# Determine if a file list is passed
if [ "$#" -eq 1 ]
then
    oIFS=$IFS
    IFS='
    '
    SFILES="$1"
    IFS=${oIFS}
fi
SFILES=${SFILES:-$STAGED_FILES_CMD}

echo "Checking PHP Lint..."
for FILE in ${SFILES}
do
    php -l -d display_errors=0 ${PROJECT}/${FILE}
    if [ $? != 0 ]
    then
        echo "Fix the error before commit."
        exit 1
    fi
    FILES="${FILES} $PROJECT/$FILE"
done



# Run the code style fixes if phpcs exists and environment variable is set
if [[ -f "vendor/bin/phpcs" ]]
then
    if [ "${FILES}" != "" ]
    then
        echo "Running Code Sniffer. Code standard PSR2."
        ./vendor/bin/phpcs --standard=PSR2 --encoding=utf-8 -n -p ${FILES}
        if [ $? != 0 ]
        then
            exec < /dev/tty
            #echo "If you would like to commit without running phpcs, run your commit overriding the env var with: 'APPLY_CODE_STYLE=false git commit'"
            echo 'Fix automatically (where possible)? [y/N]'
            read proceed
            if [[ ! ("$proceed" == 'y' || "$proceed" == 'Y') ]]
            then
                exit 1
            else
              echo "automagically fixing files"
              ./vendor/bin/phpcbf ${FILES} --standard=PSR2

              echo "re-staging updated files"
              git add ${FILES}
            fi
        fi
    fi
fi
````



---

###### 일단 요기까지 끄읕....

#### 메모리 부족 현상을 부딪히게 된다면?
````bash
Fatal error: Allowed memory size....

# -d memory_limit=256 메모리 옵션을 주고 실행하자..
$ phpcs -d memory_limit=256M --standard=PSR12 --extensions=php --ignore=vendor path/to/code

````
