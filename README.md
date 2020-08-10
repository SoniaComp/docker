# Docker로 로컬 개발환경 한번에 설정
시간이 없다면 이 문서만 봐도!!: 감사합니다ㅠㅜ[44bit프로젝트](https://www.44bits.io/ko/keyword/linux-container#%EB%8F%84%EC%BB%A4-%EC%BB%B4%ED%8F%AC%EC%A6%88docker-compose)

기본용어 및 개념: 감사합니다ㅠㅜ[욱's노트블로그](https://opennote46.tistory.com/207)

컨테이너가 VM에 비해 좋은점: 마이크로 서비스, 오케스트레이션, 빠르게 배포, 그러나 보안! 감사합니다ㅠㅜ[서강혁님의 medium](https://medium.com/@darkrasid/docker%EC%99%80-vm-d95d60e56fdd)

## 이미지와 컨테이너 사용방법(기본)

### docker-compose.yml
컨테이너 실행에 필요한 옵션을 docker-compose.yml이라는 파일에 적어둘 수 있고, 컨테이너 간 실행 순서나 의존성도 관리할 수 있습니다.

### dockerfile
이미지가 실제로는 컨테이너를 기반으로 만들어진다는 사실을 꼭 기억해야합니다. 앞에서는 이해를 위해 파일을 추가하거나 삭제하는 의미없는 작업을 했습니다만, 이번에는 좀 더 실용적인 예제로 도커 빌드의 원리에 대해서 알아보고자합니다.
이번에 만들 이미지는 깃Git이 설치된 우분투Ubuntu 이미지입니다. Dockerfile을 사용할 수도 있습니다만, 이해를 돕기 위해 앞서 배운 docker commit 명령어로 한 단계씩 이미지를 만들어보겠습니다. 먼저 우툰부 이미지에 git이 없는 것부터 확인해봅니다.

### 명령어
이미지 출력: sudo docker images

이미지 받기: sudo docker pull <이미지이름>:<태그> # sudo docker pull ubuntu:latest

run 명령으로 컨테이너 실행: sudo docekr run <옵션> <이미지이름> <실행할 파일> # sudo docker run -i -t --name hello ubuntu /bin/bash
* 호스트 OS와 완전히 격리된 공간이 생성
* bash 셸에서 빠져나오려면 exit: 우분투 이미지에서 직접 실행했기 떄문에 여기서 빠져나오면 컨테이너가 정지
* --name: 컨테이너 이름 지정

start 명령으로 실행: sudo docker start <컨테이너 이름>

restart 컨테이너 재시작(OS 재부팅): sudo docker restart <컨테이너 이름>

attach 컨테이너 접속: sudo docker attach <컨테이너 이름>
* 엔터를 한번더 입력하면 bash 셸이 표시된다.
* ctrl+D 컨테이너 정지
* ctrl+P, ctrl+Q 컨테이너에서 빠져나옴

exec (bin/bash를 실행하지 않고) 외부에서 컨테이너 안의 명령 실행하기: sudo docker exec <컨테이너 이름> <명령> <매개변수>
* sudo docker exec hello echo "Hello World"
* 이미 실행된 컨테이너 안에서 apt-get, yum 명령으로 패키지를 설치하거나, 각종 데몬을 실행할 때 활용

stop 명령으로 컨테이너 정지
* sudo docker ps # 컨테이너 목록 출력
* sudo docker stop <컨테이너 이름>

rm 컨테이너 삭제
* sudo docker rm <컨테이너 이름>

rmi 이미지 삭제: sudo docker rmi <이미지 이름>:<태그>

## Docker 이미지 생성-Dockerfile

### Bash 익히기
출력
* > : 출력 (>> 표준 출력)
* 2> 명령 실행의 표준 에러를 파일로 저장
* 2>> 표준 에러를 파일에 추가
* &> 표준 출력과 표준 에러를 모두 파일로 저장
* 1>&2 표준 출력을 표준 에러로 보냄. echo명령으로 문자열을 표준으로 출력했지만, 표준 에러로 보냈기 때문에 변수에는 문자열이 들어가지 않음
* 2>&1 표준 에러를 표준 출력으로 보냄.  

입력
* < : 입력
* | : 파이프. 명령 실행의 표준 출력을 다른 명령의 표준 입력으로 보냅니다. 즉 첫번째 명령의 출력 값을 두번째 명령에서 처리

변수
* $ : Bash의 변수. 값을 저장할 때는 $를 붙이지 않고, 변수를 가져다 쓸 때만 $를 붙입니다.
* $() : 명령 실행 결과를 변수화합니다. 명령 실행 결과를 변수에 저장하거나 다른 명령의 매개 변수로 넘겨줄 때 사용. 문자열 안의 명령 실행결과를 넣을 때 사용

명령
* `` : $()과 마찬가지로 명령 실행 결과를 변수화
* && : 한줄에서 명령을 여러개 실행. 
* ; : 한줄에서 명령 여러개 실행
* \ : 한줄로 된 명령을 여러줄로 표현할 때

문자열
* '' : ''안에 들어 있는 변수는 처리 되지 않고 변수명 그대로 사용됩니다. 또한 "와 $()도 처리되지 않고 그대로 사용됩니다.
* "" : 문자에 문자열 매개변수를 입력하거나 변수에 저장할 때 주로 사용, ''와 달리 ""안에 변수가 들어 있으면 변수의 내용으로 바뀝니다.
* " ' ' " : ""안에 ''들어갈 수 있다.
* \", \$ : ", $문자열 앞에는 \
* ${ } : 변수 문자열 치환. 문자열 안에서 변수 출력할 때 사용 ${}대신 $만사용해도 됨
* {1..10} : 연속된 숫자 {시작숫자..끝숫자}
* {문자열1, 문자열2} : {}안에 문자열을 여러개 지정하여 명령 실행횟수를 줄입니다. cp ./{hello.txt, world.txt} hello-dir/ hello-dir아래에 복사

실행문
* if : 조건문, 변수와 변수끼리 또는 문자열과 비교할 때 사용
숫자 비교: -eq(같다), -ne(같지 않다), -gt(초과), -ge(이상), -lt(미만), -le(이하)
문자열 비교: =, ==(같다), !=(같지 않다), -z(문자열이 NULL), -n(문자열이 NULL이 아닐때)
* for, while: 반복문

문자열 표준 입력
* <<<: 문자열을 명령(프로세스)의 표준 입력으로 보냅니다.
* <<EOF, EOF: 여러줄의 문자열을 명령(프로세스)의 표준 입력으로 보냅니다.

* export: 설정한 값을 환경변수로 만듭니다. export <변수>=<값>
* printf: 지정한 형식대로 값을 출력. 파이프와 연동하여 명령(프로세스)에 값을 입력하는 효과를 낼 수 있다.
* 텍스트 파일에서 문자열 바꾸기: sed -i "s/<찾을 문자열><바꿀 문자열>/g" <파일명>

### Dockerfile 문법
FROM: 어떤 이미지를 기반으로 할지 설정합니다. 기존에 만들어진 이미지를 기반으로 생성. <이미지이름>:<태그>

MAINTAINER: 메인테이너 정보

RUN: 셸스크립트 혹은 명령을 실행
* 이미지 생성중에는 사용자 입력을 받을 수 없으므로 -y dhqtus tkdyd

VOLUME: 호스트와 공유할 디렉토리 목록
* docker run 명령에서 -v 옵션으로 설정할 수 있다.

CMD: 컨테이너가 시작되었을 때, 실행할 실행 파일 또는 셸스크립트

WORKDIR: CMD에서 설정한 실행파일이 실행될 디렉터리

EXPOSE: 호스트와 연결할 포트 번호

### Dockerfile -> 이미지 생성(build) -> 이미지로 컨테이너 실행(run)
sudo docker build <옵션> <Dockerfile경로>  # sudo docker build --tag hello:0.1
* --t(--tag) 이미지 이름 설정

sudo docker run --name hello-nginx -d -p 80:80 -v /root/data:/data hello:0.1
* -d: 컨테이너를 백그라운드로 실행
* -p: 80:80 옵션으로 호스트의 80번 포트와 컨테이너의 80번 포트를 연결하고 외부에 노출합니다. 이렇게 설정한뒤 http://호스트ip:80에 접속하면 컨테이너의 80번 포트로 접속됩니다.
* -v /root/data/:data 옵션으로 /root/data 디렉터리를 컨테이너의 /data 디렉토리에 연결합니다. /root/data 디렉토리에 파일을 넣으면 컨테이너에서 해당파일을 읽을 수 있습니다.

### Dockerfile 자세히 알아보기
기본형식: <명령><매개변수>

항상 시작은 "FROM"

각 명령은 독립적으로 실행

.dockerignore
* docker file 과 같은 디렉토리에 들어있는 모든 파일을 '컨텍스트'라고 하빈다. 특히 이미지를 생성할 때 컨텍스트를 모두 Docker 데몬에 전송하므로 필요없는 파일이 포함되지 않도록 주의합니다.
* docker는 Go 언어로 작성되어 있기 때문에 파일 매칭도 Go 언어의 규칙을 따릅니다.

FROM
* Dockerfile로 이미지를 생성할 때는 항상 기존에 있는 이미지를 기반으로 생성하기 때문에 FROM은 반드시 설정해야 합니다.

MAINTAINER
* 이미지를 생성한 사람의 정보를 설정합니다. MAINTAINER <작성자 정보>

RUN 
* FROM에서 설정한 이미지 위에서 스트립트 혹은 명령을 실행합니다. RUN으로 실행한 결과가 새 이미지로 생성되고, 실행 내역은 이미지의 히스토리에 기록됩니다.
* RUN <명령>
* 이미지에 포함된 /bin/sh 실행파일을 사용하게 된다.
* RUN ["실행파일", "매개변수1", "매개변수2"] 형식
* 실행파일과 매개변수를 배열 형태로 설정
* RUN으로 실행한 결과는 캐시되며 다음 빌드 때 재사용
* 캐시된 결과를 사용하지 않으려면 --no-cache 옵션을 사용

CMD
* 컨테이너가 시작되었을 때 스크립트 혹은 명령을 실행합니다.


## 도커 머신
도커 머신은 사용자의 로컬 컴퓨터, 클라우드 서비스가 제공하는 인스턴스, 원격 서버에 도커 호스트를 구성해준다. 도커 머신을 이용하면 자동으로 도커를 설치하고 도커 호스트를 만들 수 있고, docker 클라이언트를 사용해서 설정한 서버의 도커 호스트에 도커 명령을 실행할 수 있다.
[도커 머신 설치 공식 URL](https://docs.docker.com/machine/install-machine/)
자세한 설명 블로그: [소용환의 생각저장소](https://www.sauru.so/blog/provision-docker-node-with-docker-machine/)

## Docker composer 작성
여러 컨테이너에 대한 설정을 한번에 할 수 있다.

Docker compose 문법: 출처는 [SilNex님의 블로그](https://blog.silnex.kr/dockerdocker-compose-%EC%A0%95%EB%A6%AC/)
```
version: '{버전}'
services:
  {도커 이름}:
    driver: {네트워크 이름}
    build: {dockerfile build 할 시 디렉토리}
    ports:
    - "{외부포트}:{컨테이너포트}"
    volumes:
    - {볼륨명}: {컨테이너 위치}
    - logvolume01:/var/log
    links:
    - redis #링크
  redis:
    image: redis # dockerfile에서 build 하지않고 이미지를 받아옴
    environment:
        {환경 변수명}: {컨테이너이름}:3456
        DB_PASSWORD: redis:password
volumes:
  logvolume01: {}

networks:
  {네트워크 이름}:
    # 커스텀 네트워크 사용
    driver: custom-driver-1
  {네트워크 이름}:
    # 커스텀 네트워크 옵션 사용
    driver: custom-driver-2
    driver_opts:
      foo: "1"
      bar: "2"
```

## Docker로 리버스 프록시 서버 만들기
참고 블로그:

감사합니다ㅠㅜ [jm4488님](https://jm4488.tistory.com/63)
[코마님 감사합니다ㅠㅜㅠㅜㅠㅜ](https://code-machina.github.io/2019/08/01/Docker-Express-API-Gateway-Part-1.html)

1. nginx-proxy 컨테이너 실행
```

$ docker 
$ docker pull jwilder/nginx-proxy:latest
$ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/usr/local/bin/docker-machine &&
  chmod +x /usr/local/bin/docker-machine
$ sudo docker run -i -t --name proxy-vue jwilder/nginx-proxy
```
2. Dockerfile로 Node.js 이미지 파일 빌드

3. 빌드한 이미지로 Node.js 웹서버 실행

## 개발환경 반영하게 하기
감사합니다 [44Bits](https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose#%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EC%8B%A4%ED%96%89)

--volume: 이 옵션을 사용하여 로컬 디렉토리의 특정 경로를 컨테이너 내부로 마운트할 수 있습니다.
* ex) --volume=$(pwd)/docker/data:/var/lib/postgresql/data
* 컨테이너 내부의 디렉토리(/var/lib/postgresql/data)를 로컬 컴퓨터의 디렉토리($(pwd)/docker/data)로 연결했습니다. 
* 데이터베이스가 데이터를 저장할 파일 시스템으로 로컬 컴퓨터의 $(pwd)/docker/data 디렉토리를 사용합니다.

--rm: 종료시 데이터도 깔끔하게 사라짐 

## 태초에 도커는 CMD에서 명령어를 쳤었다아아!
* 장황한 옵션
* 앱컨테이너와 데이터 베이스 컨테이너의 실행 순서

### 해결책1: 도커파일 -> docker build -> run
Docker File이란 Docker Image를 만들기 위한 설정 파일입니다. 여러가지 명령어를 토대로 Docker File을 작성하면 설정된 내용대로 Docker Image를 만들 수 있습니다.

### 해결책2: docker-compose.yml -> docker-compose up -d
docker-compose 란, 한번에 여러개의 container 을 통합 관리 할 수 있게 하는 툴입니다.

* build: db 서비스와 달리 앱서비스는 특정 이미지 대신 build 옵션을 추가합니다. context는 docker build 명령을 실행할 디렉토리 경로라고 보시면 됩니다. dockerfile 에는 '개발용' 도커 이미지를 빌드하는데 사용할 dockerfile을 지정하면 됩니다.
* environment: 앱 서비스의 환경변수로 설정

docker-compose.yml 안에 있는 서비스들은 별도로 지정하지 않으면 하나의 네트워크에 속합니다. (네트워크와 관련된 더 자세한 내용은 [Networking in Compose](https://docs.docker.com/compose/networking/)를 참고하세요.)