## Docker 란?
- 도커(Docker)란 컨테이너 기술을 사용하여 애플리케이션에 필요한 환경을 신속하게 구축하고 테스트 및 배포할 수 있게 해주는 플랫폼이다.
- 컨테이너 기반 오픈소스 가상화 플랫폼이라고 정의할 수 있다.

### 도커 이미지와 컨테이너
#### 이미지
- 서비스 운영에 필요한 서버 프로그램, 소스 코드 및 라이브러리, 컴파일된 실행 파일을 묶는 형태
- 즉, 특정 프로세스를 실행하기 위한 모든 파일과 설정값을 지닌 파일들의 묶음
- 도커 이미지들을 github와 유사한 서비스인 DockerHub를 통해 버전관리 및 배포가 가능하다.
#### 컨테이너
- 이미지를 실행한 상태로, 응용프로그램의 종속성과 함께 응용 프로그램 자체를 패키징 or 캡슐화하여 격리된 공간에서 프로세스를 동작시키는 기술

### 도커의 구성요소(순서)
Building Containers - Dockerfile, Image, Container  
1. Dockerfile
컨테이너를 어떻게 만들어야 하는지에 대한 설명서
- 애플리케이션 구동에 필요한 파일이 무엇인지
- 외부 dependencies
- 필요한 환경변수
- 어떻게 구동해야하는지에 대한 스크립트
2. Image
작성한 Dockerfile을 이용해 생성
- 애플리케이션을 실행하는데 필요한 코드, 런타임 환경, 시스템 툴, 라이브러리 등이 포함
- 실행되고 있는 애플리케이션의 상태를 스냅샷해서 이미지로 생성
- 만들어진 이미지는 불변의 상태
3. Container
애플리케이션의 Image를 개별적인 환경에서 실행

### 도커의 사용 이유
- VM을 사용하지 않고 도커 엔진을 이용하여 동작하기 때문에 성능 개선과 동시에 메모리 용량을 적게 요구한다.
- 컨테이너를 실행하기 위한 모든 정보를 갖고 있기 때문에, 새로운 환경에서 이것 저것 설치할 필요 없이 서버에 이미지만 다운 받아서 컨테이너를 생성할 수 있다.
- 개발환경 설정할 때 초기 세팅이 빠르고 실행환경을 강제화할 수 있다.
- 도커는 개발자가 원하는 환경 세팅을 모듈식 유닛을 조합함으로써 만들 수 있게 해준다. 이는 개발 주기, 기능 배포, 버그 수정의 속도를 높여준다.

### Virtual Machine vs Container
#### Virtual Machine

![스크린샷 2023-06-29 오후 1 03 08](https://github.com/zzangoobrother/study-organization/assets/42162127/1aabe320-cdf5-4823-8ca5-31c220e500aa)

- 무거운 운영체제를 포함함

#### Container

![스크린샷 2023-06-29 오후 1 03 52](https://github.com/zzangoobrother/study-organization/assets/42162127/2d8cec51-7846-4b74-b590-4526b5f33dee)

- container engine이라는 software설치로 개별적인 container를 만들어서 각각의 애플리케이션을 고립된 환경에서 구동할 수 있게 해준다.

## Docker compose란?
- 멀티 컨테이너 도커 애플리케이션을 정의하고 실행하는 도구이다.
- yml파일을 사용하여 애플리케이션의 서비스를 구성하며 하나의 명령을 가지고 모든 컨테이너의 생성 및 시작 프로세스를 수행한다.

## Docker 명령어
- docker 명령어(docker ps, start, stop, remove etc..)
- 도커의 명령어는 크게 이미지 관련, 컨테이너 관련, 축약 명령어, 운용 관련 명령어 등으로 나누어 진다.

### 도커 도움말
````shell
docker help
docker search --help # 도커 명령어 별 상세 사용 방법 
````

### 이미지 관련 명령어
````shell
# dockerhub 레지스트리에서 도커 이미지 검색
docker search [이미지이름] 

# 이미지 다운로드, 태그명을 생략하면 기본적으로 latest가 적용 
docker image pull [옵션] 레파지토리명[:태그명]

# 이미지 목록 보기
docker image ls
````

### 컨테이너 관련 명령어
````shell
# 컨테이너 실행
docker container run [옵션] 이미지명[:태그명]

# 컨테이너 옵션
-d # 백그라운드로 실행한다.
-p # 외부포트:컨테이너포트 / 포트를 지정하지 않은 경우 임의의 포트가 자동으로 할당된다.
-t # 유닉스 터미널 연결 활성화를 시킨다. -i옵션과 같이 많이 사용된다.
-i # 컨테이너 쪽 표준 입력과 연결을 유지한다. 컨테이너 쪽 셸에 들어가려면 이 옵션을 추가해야한다.
-rm # 컨테이너 종료 시 컨테이너를 파기한다. 
--name # 컨테이너에 원하는 이름을 붙일 수 잇다. 생성된 컨테이너는 이름으로 조회를 하거나 삭제할 수 있고
			 # name을 부여하지 않고 컨테이너를 생성하면 랜덤한 이름이 생성된다. 

# 실행중인 컨테이너 목록 조회
docker container ls

# 컨테이너 옵션
-a # 모든 컨테이너 목록 조회(종료된 컨테이너 포함)
-f # 컨테이너 목록 필터링 조회(ex. docker container ls -f "name=ubuntu")
-n # 모든 컨테이너 목록들 중 n번째 까지의 최근 목록 조회(ex. docker container ls -n 3)
-l # 모든 컨테이너 목록 중 가장 최근 컨테이너 조회
-q # 실행 중인 컨테이너 아이디 조회
-s # 실행 중인 컨테이너의 파일 사이즈 조회 

# 실행 중인 컨테이너로 들어가기
docker container attach [컨테이너명]

# 실행 중인 컨테이너 정지하기
docker container stop [컨테이너아이디|컨테이너명]

# 정지된 컨테이너 시작하기
docker container start [컨테이너아이디|컨테이너명]

# 컨테이너 재시작하기 
docker container restart [컨테이너아이디|컨테이너명]

# 컨테이너 삭제하기(기본 옵션으로는 실행 중인 컨테이너를 종료하고 삭제)
docker container rm [컨테이너아이디|컨테이너명]

# 실행 중인 컨테이너 강제 종료 후 삭제
docker container rm -f [컨테이너아이디|컨테이너명]
````

### 축약 명령어
````shell
docker image pull         docker pull
docker image ls           docker images
docker container run      docker run
docker container start    docker start
docker container ls       docker ps
docker container stop     docker stop
````

### 운용 관련 명령어
````shell
# 컨테이너 및 이미지 파기하기
# 현재 실행 중이지 않은 모든 컨테이너를 삭제할 때 사용
docker container prune

# 태그가 붙지 않은 모든 이미지를 삭제할 때
docker image prune 

# 이미지, 컨테이너 등 모든 리소스를 일괄 삭제할 때
docker system prune

# 컨테이너 시스템 리소스 사용 현황 확인하기
docker container stats
````
