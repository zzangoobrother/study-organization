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
