### Docker 환경 설정 후 PhpStorm 에서 xdebug 로 디버깅 하기



#### Docker 설정

> Nginx + php-fpm + MariaDB + Redis



아래 파일은 [PHPDocker.io](https://phpdocker.io/generator) 에서 기본적으로 생성 후 추가내용을 삽입 하였다.

````bash
.
├── docker-compose.yml
└── phpdocker
    ├── nginx
    │   └── nginx.conf
    └── php-fpm
        ├── Dockerfile
        ├── xdebug-overrides.ini
        └── php-overrides.ini
````



`docker-compose.yml`

````yaml
###############################################################################
#                          Generated on phpdocker.io                          #
###############################################################################
version: "3.1"
services:

    redis:
      image: redis:alpine
      container_name: dev-redis

    mariadb:
      image: mariadb:10.4
      container_name: dev-mariadb
      working_dir: /application
      volumes:
        - .:/application
      environment:
        - MYSQL_ROOT_PASSWORD=password
        - MYSQL_DATABASE=testdb
        - MYSQL_USER=nathan
        - MYSQL_PASSWORD=pass
      ports:
        - "3306:3306"

    webserver:
      image: nginx:alpine
      container_name: dev-nginx
      working_dir: /application
      volumes:
          - ./www/ci4:/application
          - ./phpdocker/nginx/nginx.conf:/etc/nginx/conf.d/default.conf
      ports:
       - "80:80"

    php-fpm:
      build: phpdocker/php-fpm
      container_name: dev-php-fpm
      working_dir: /application
      volumes:
        - ./www/ci4:/application
        - ./phpdocker/php-fpm/php-overrides.ini:/etc/php/7.4/fpm/conf.d/99-overrides.ini
        - ./phpdocker/php-fpm/xdebug-overrides.ini:/etc/php/7.4/fpm/conf.d/20-xdebug.ini

````

> `Working Directory`를 `application` 으로 설정하였다.
>
> 해당 디렉토리명을 변경 할 경우 아래 설정 파일들도 수정하여야 한다.
>
> `nginx.conf` : root /`application`/public;
>
> `Dockerfile` : WORKDIR "/`application`"





`nginx.conf`

````nginx
server {
    listen 80 default;
    server_name docker.dev;

    access_log /var/log/nginx/application.access.log;

    root /application/public;
    index index.php;

    if (!-e $request_filename) {
        rewrite ^.*$ /index.php last;
    }

    location ~ \.php$ {
        fastcgi_pass php-fpm:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PHP_VALUE "error_log=/var/log/nginx/application_php_errors.log";
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        # 아래 설정값은 default 60
        # xDebug 설정으로 확인을 위해 충분히 늘려놓았다.. 실서버에서는 있어서는 안될일.
        fastcgi_send_timeout 600;
        fastcgi_read_timeout 600;
        include fastcgi_params;
    }
    
}
````

> root /application/public; 부분은 작동하는 앱에 따라 다르게 설정할 필요가 있다. 
>
> ci4 의 경우 ./public/index.php 가 존재 하므로 public 설정을 하였다.
>
> 일반적인 php framework의 경우 public 디렉토리 안에 index.php 가 존재 하는것으로 알고 있다.



`php-overrides.ini`

````ini
upload_max_filesize = 100M
post_max_size = 108M
date.timezone = Asia/Seoul
short_open_tag = On
max_execution_time = 1000
max_input_time = 1000
````



`xdebug-overrides.ini` 

> `xdebug.remote_port` 는 php-fpm 이 9000 port를 사용중이므로 `9001`로 설정한다.
>
> 윈도우 운영체제의 경우 xdebug.remote_host 의 ip 또는 접근 가능한 host 명으로 변경되어야 한다.

````ini
zend_extension=xdebug.so

xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_handler=dbgp
xdebug.remote_host=docker.for.mac.localhost
xdebug.remote_port=9001
xdebug.idekey=PHPSTORM
````



`Dockerfile`

````dockerfile
FROM phpdockerio/php74-fpm:latest
WORKDIR "/application"

# Fix debconf warnings upon build
ARG DEBIAN_FRONTEND=noninteractive

# Install selected extensions and other stuff
RUN apt-get update \
    && apt-get -y --no-install-recommends install  php-memcached php7.4-mysql php7.4-pgsql php-redis php-xdebug php7.4-bcmath php7.4-gd php-igbinary php7.4-intl php7.4-odbc \
    && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*
````



- background 에서 컨테이너 시작 : `docker-compose up -d`
- foreground 에서 컨테이너 시작 : `docker-compose up`
- 컨네이너 정지 : `docker-compose stop`
- 컨테이너 로그 보기 : `docker-compose logs`

- 컨테이너 접속 커맨드 : `docker-compose exec SERVICE_NAME COMMAND`
- php-fpm 컨테이너 접속하기 : `docker-compose exec php-fpm bash`
- mariadb 접속하기 : `docker-compose exec mariadb mariadb -uroot -ppassword`



#### PhpStorm 설정 환경설정

Languages & Frameworks > PHP > Debug

Xdebug 포트 : 9001 설정

![](https://i.loli.net/2020/06/01/7w9sAbXWHTcgPQk.png)



Languages & Frameworks > PHP > Debug > DBGp Proxy

IDE Key 와 포트설정 PHPSTORM /9001

![](https://i.loli.net/2020/06/01/naQeKjHV3bwsUht.png)



Languages & Frameworks > PHP > Servers

`+` 눌러서 추가하기 FileDrectory 와 Absolute path on the server 설정

![](https://i.loli.net/2020/06/01/YpAVeK3nkhorEGX.png)



Build, Execution, Deployment > Docker 설정

`+` 눌러서 추가 하단의 Connection successful 이면 성공

![](https://i.loli.net/2020/06/01/xiNgWu7Rwnc8LZ2.png)
