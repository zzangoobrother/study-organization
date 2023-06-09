## NCloud Git Action CICD 설정

![스크린샷 2023-06-28 오후 1 32 32](https://github.com/zzangoobrother/study-organization/assets/42162127/3965720b-ac02-4706-a9e5-9f4b963ebb39)

Container Registry는 NCloud에서 제공하는 dockerhub 라고 생각하면 됩니다.  
github action을 통해 프로젝트를 빌드 후 docker image를 Container Registry에 배포합니다. 그리고 server에서 image를 pull 받아 실행 합니다.  
이런 흐름을 이해 하고 설정을 시작하겠습니다.

Container Registry : 도커 이미지를 간편하게 저장, 관리할 수 있는 서비스  
Object Storage : 모든 종류의 데이터를 저장할 수 있는 서비스

Container Registry를 사용하기 위해서는 Object Storage를 생성해야 합니다. Container Registry는 단순 도커 이미지를 관리만 할 뿐 실제 도커 이미지 저장은 Object Storage에 합니다.

Object Storage에서 버킷을 생성후 Container Registry를 생성합니다.  
레지스트리 생성 버튼을 누르시고 레지스트리 이름과 방금 만든 버킷을 선택하시면 생성이 됩니다.

해당 Container Repository에 접근하기 위해서는 로그인이 필요합니다.
````shell
docker login <CONTAINER_REGISTRY_URL>
````
Container Registry 화면에서 레지스토리 클릭하여 접근 명령어를 그대로 복사해서 사용하면 됩니다.  

NCloud container registry에 로그인하기 위해서는 마이페이지에서 인증키 생성이 필요합니다.
- 마이페이지 - 계정관리 - 인증키관리 - 신규 API 인증키 생성 - Access Key ID, Secret Key가 각각 ID, PW로 로그인시 사용됩니다.
````shell
docker build -t <CONTAINER_REGISTRY_URL>/<TARGET_IMAGE[:TAG]>
docker push <CONTAINER_REGISTRY_URL>/<TARGET_IMAGE[:TAG]>
````
CONTAINER_REGISTRY_URL → Public Endpoint   
TARGET_IMAGE → 태그이기 때문에 설정하고자 하는 이름을 넣으면 됩니다.  

운영될 서버에 docker를 설치하게 습니다.  
apt package index 최신으로 업데이트 및 apt https를 통해 저장소를 사용할 수 있도록 패키지를 설치 합니다.
````shell
$ sudo apt-get update
$ sudo apt-get install \
   ca-certificates \
   curl \
   gnupg \
   lsb-release
````

docker의 GPG키 추가
````shell
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
````

저장소 설정
````shell
$ echo \
 "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
 $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
````

docker engine 설치
````shell
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
````

docker가 잘 설치되었는지 Hello-world 이미지를 통해 확인
````shell
$ sudo docker run hello-world
````

운영서버에 docker 설치가 완료 되었다면 github repository에 github action 설정을 하겠습니다.

![스크린샷 2023-06-28 오후 2 39 11](https://github.com/zzangoobrother/study-organization/assets/42162127/66182a8d-cc11-4c59-9006-8bc275e37017)

github 메뉴바에서 Actions 탭을 클릭합니다.  
현재 프로젝트에 맞는 빌드를 선택하시면 됩니다. Java with Gradls 또는 Java with Maven 을 선택하시면 됩니다.

![스크린샷 2023-06-28 오후 2 40 43](https://github.com/zzangoobrother/study-organization/assets/42162127/26c3425e-d8df-4893-9191-bbb80a17f9c3)

Configure를 클릭하시면 yml 파일를 생성할 수 있게 나옵니다.
````shell
name: Java CI with Gradle

on:
  push:
    paths:
      - '멀티모듈이면 멀티모듈명'
    branches: [ "master" ] // 브랜치명
  pull_request:
    branches: [ "master" ] // 브랜치명 풀리퀘스트일 경우만

permissions:
  contents: read

jobs:
  push_to_registry:
    name: Push to ncp container registry // 무엇을 하는지 알기 위해 작성
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
    - name: Build with Gradle
      run: ./gradlew clean :멀티모듈명:buildNeeded -Dspring.profiles.active=prod --stacktrace --info --refresh-dependencies

    - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      - name: Login to NCP Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ secrets.NCP_CONTAINER_REGISTRY }}
          username: ${{ secrets.NCP_ACCESS_KEY }}
          password: ${{ secrets.NCP_SECRET_KEY }}

    - name: Admin build and push
        uses: docker/build-push-action@v3
        with:
          context: .
          file: ./module-admin/Dockerfile
          push: true
          tags: ${{ secrets.NCP_CONTAINER_REGISTRY }}/admin:latest
          cache-from: type=registry,ref=${{ secrets.NCP_CONTAINER_REGISTRY }}/admin:latest
          cache-to: type=inline

   pull_from_registry:
    name: Connect server ssh and pull from container registry
    needs: push_to_registry
    runs-on: ubuntu-latest
    steps:
      - name: connect ssh
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PROD_SERVER_HOST }}
          username: ${{ secrets.PROD_SERVER_USERNAME }}
          password: ${{ secrets.PROD_SERVER_PASSWORD }}
          port: ${{ secrets.PROD_SERVER_PORT_ADMIN }}
          script: |
            ./deploy.sh // 실행 스크립트 작성
````

````shell
on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]
````
소스를 push 또는 pull request 할때 github action이 실행됩니다.

지금은 예제를 보여주기 위해 push pull request 모두 있지만 운영 안정성을 생각하면 코드 리뷰와 함께 pull request가 발생하면 실행 되게 해야 합니다. 그리고 main, stage, dev 같이 서버가 따로 설정되어 있고, 할 때는
````shell
on: pull_request
````
이렇게 하시면 pull request가 발생하면 github action이 실행 됩니다.

````shell
runs-on: ubuntu-latest
````
서버가 ubuntu에서 실행되기 때문에 설정했습니다.

````shell
- name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      - name: Login to NCP Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ secrets.NCP_CONTAINER_REGISTRY }}
          username: ${{ secrets.NCP_ACCESS_KEY }}
          password: ${{ secrets.NCP_SECRET_KEY }}
````
nCloud Container Registry에 접속합니다.

````shell
- name: Admin build and push
        uses: docker/build-push-action@v3
        with:
          context: .
          file: ./module-admin/Dockerfile
          push: true
          tags: ${{ secrets.NCP_CONTAINER_REGISTRY }}/admin:latest
          cache-from: type=registry,ref=${{ secrets.NCP_CONTAINER_REGISTRY }}/admin:latest
          cache-to: type=inline
````
Dockerfile를 실행하고 Container Registry에 push 합니다.

Dockerfile
````shell
FROM openjdk:17

COPY module-admin/build/libs/module-admin-0.0.1-SNAPSHOT.jar ./app.jar
ENV TZ=Asia/Seoul
ENTRYPOINT ["java", "-Dspring.profiles.active=${USE_PROFILE}", "-jar", "./app.jar"]
````

CI 기능은 끝났습니다. 다음은 CD 입니다.

````shell
name: Connect server ssh and pull from container registry
needs: push_to_registry
````
needs 는 push_to_registry 가 끝나고 시작됩니다.

````java
steps:
   - name: connect ssh
     uses: appleboy/ssh-action@master
     with:
      host: ${{ secrets.PROD_SERVER_HOST }}
      username: ${{ secrets.PROD_SERVER_USERNAME }}
      password: ${{ secrets.PROD_SERVER_PASSWORD }}
      port: ${{ secrets.PROD_SERVER_PORT_ADMIN }}
````
ssh를 통해 운영서버에 접속합니다.  
host : 서버 접속용 공인 IP  
username: root  
password: 서버 설정한 password  
port: 서버 접속용 공인 포트번호

Container Registry에 새로 업로드된 docker image를 pull 받고 지금 실행되고 있는 docker를 중지합니다.  
그리고 기존 docker image를 삭제 후 새로 받은 docker imgea를 실행 합니다.

secrets key
NCP_ACCESS_KEY : 마이페이지에서 받은 인증키  
NCP_SECRET_KEY : 마이페이지에서 받은 비밀키  
NCP_CONTAINER_REGISTRY : Container Registry public endpoint  
DEV_HOST : server 접속용 공인 ip  
DEV_USERNAME : root  
DEV_PASSWORD : server 설정 비밀번호  
DEV_PORT : ssh 포트  

그리고 만약 pull request를 거의 동시에 하여 동시에 배포 중이라면 선행 작업이 완료되고 후행 작업이 완료되면 괜찮지만 후행 작업이 먼저 완료하고 선행 작업이 완료된다면 이전 버전이 운영서버에 배포된 상황이다.  
이 문제 해결은 추후 다루도록 하겠다. 우선 아래 참고 블로그를 보시면 된다.

+ <a href='https://hyeon9mak.github.io/github-actions-deployment-concurrency-setting/' target='_blank' >현구막님 중복 배포 해결</a>
