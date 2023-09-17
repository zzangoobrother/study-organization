## AWS ECS, ECR, Fargate를 이용한 gitAction 서버리스 CICD 배포

### Amazon ECS Fargate
Amazon Elastic Container Service(ECS)는 AWS에서 제공하는 컨테이너 오케스트레이션 서비스 이다.
ECS Fargate는 ECS의 실행 모드 중 하나로, 서버리스를 지원하는 컨테이너 오케스트레이션 서비스 이다.

Fargate를 사용하면 컨테이너를 보다 쉽고 간편하게 실행할 수 있으며, 인프라 관리를 최소화할 수 있다.
컨테이너 운영을 더욱 쉽고 빠르게 시작할 수 있도록 도와주며, 서버리스 환경에서도 컨테이너를 실행할 수 있는 유연성을 제공한다.

### ECR 생성

<img width="1454" alt="image" src="https://github.com/zzangoobrother/study-organization/assets/42162127/97ae86f8-b2f9-4eed-b5d2-dc085ddfed25">

<img width="818" alt="image" src="https://github.com/zzangoobrother/study-organization/assets/42162127/2db39643-75ab-4449-a43e-123773e26a84">

리포지토리 이름을 작성합니다.
나머지는 건들지 않고 생성을 합니다.

### ECS 클러스터 생성
ECS에서 클러스터를 생성합니다.
<img width="1417" alt="image" src="https://github.com/zzangoobrother/study-organization/assets/42162127/ca51fe33-d314-4b2a-b71b-c5e895cc8244">

<img width="825" alt="image" src="https://github.com/zzangoobrother/study-organization/assets/42162127/6414a90e-382a-4ebb-94a9-ab157cb1c975">

클러스터 이름을 작성합니다.
인프라에서 AWS Fargate(서버리스)를 선택합니다.

<img width="1422" alt="image" src="https://github.com/zzangoobrother/study-organization/assets/42162127/aed2c95a-459e-42d2-886c-ec0bfcb038f0">

### ECS 테스크 정의
<img width="1434" alt="image" src="https://github.com/zzangoobrother/study-organization/assets/42162127/f021dede-7826-4904-b598-f288e55fc679">

<img width="1126" alt="image" src="https://github.com/zzangoobrother/study-organization/assets/42162127/27e83ed6-7a58-40b0-81c4-c2d15db7c5ea">

테스크 정의 이름을 입력한다.

<img width="1094" alt="image" src="https://github.com/zzangoobrother/study-organization/assets/42162127/59c19f58-6b2d-42db-a8f7-17d46b4effd3">

AWS Fargate를 체크하고, CPU와 메모리를 선택하는데 메모리를 2GB로 하겠다.
테스크 역할도 선택한다.

<img width="1095" alt="image" src="https://github.com/zzangoobrother/study-organization/assets/42162127/4c76413d-c24d-4548-915d-b876eed26a31">

컨테이너의 이름과 위에서 생성한 ECR의 URI:Image Tag 를 입력한다.
컨테이너 포트는 8080으로 포트 이름을 입력한다.

<img width="1410" alt="image" src="https://github.com/zzangoobrother/study-organization/assets/42162127/75edca79-4fc4-4ea1-8e31-30f024461f2d">

![image](https://github.com/zzangoobrother/study-organization/assets/42162127/98a50ff7-fc2a-4b01-ae48-21b6139bec81)


생성한 컨테이너 정의에서 json 파일을 다운받아 프로젝트 루트 경로에 넣어준다.
ECR 도커 이미지 배포 후 해당 json 파일로 ECS 클러스터 내에서 Task를 실행하게 된다.


