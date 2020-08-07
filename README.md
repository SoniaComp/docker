# Docker로 로컬 개발환경 한번에 설정

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
