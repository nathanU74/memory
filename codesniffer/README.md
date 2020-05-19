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



###### 음... Git에 올리기전에 검사를 해서 올릴 수 있을거 같아서 찾아보았다..
