# JENKINS

##### Jenkins Tutorial
[젠킨스시작하기](https://riptutorial.com/ko/jenkins)

##### Jenkins & GitLab
[우왕해봐야함](https://gwonsungjun.github.io/articles/2019-04/jenkins_tutorial_2)


### Docker 를 활용해서 Jenkins & GitLab 설치

내컴에 깔았더니 너무 느려서 -_-;;; AWS 에 인스턴스 생성해서 다시 해봐야겠다.

##### Jenkins 설치를 위한 도커파일 생성

````bash
$ vi Dockerfile-jenkins 

# 1. Jenkins Long Term Support(LTS) 이미지 생성
FROM jenkins/jenkins:lts
    
# 2. 명령을 실행할 사용자 설정
USER root
    
# 3. Jenkins build 시 필요한 zip command install
RUN apt-get update
RUN apt-get install -y zip
    
# 4.Jenkins build 시 필요한 awscli command install
RUN apt-get install -y python-pip
RUN pip install awscli
RUN pip install --upgrade pip

````



##### docker-compose.yml 파일 생성

( yaml 파일은 문법상 들여쓰기를 tab 사용하면 안된다.)

````yaml
version: "3.7"
services:

  jenkins:
    build:
      context: .
      dockerfile: Dockerfile-jenkins
    container_name: jenkins
    restart: always
    user: root
    ports:
      - "8080:8080"
    volumes:
      - "./jenkins/jenkins_home:/var/jenkins_home"


  gitlab:
    image: "gitlab/gitlab-ce:latest"
    container_name: gitlab
    restart: always
    hostname: "gitlab.nathan.com"
    environment:
      GITLAB_OMNIBUS_CONFIG:
        external_url = "gitlab.nathan.com"
    ports:
      - "80:80"
      - "443:443"
      - "22:22"
    volumes:
      - "./gitlab/config:/etc/gitlab"
      - "./gitlab/logs:/var/log/gitlab"
      - "./gitlab/data:/var/opt/gitlab"
````


