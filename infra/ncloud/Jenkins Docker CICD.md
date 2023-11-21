## Jenkins Docker CICD

Jenkins와 Git 연결부터 시작하겠습니다.
Jenkins에서 Git에 접근하기 위해 token을 발급 받습니다.
- repo
- admin:repo_hook
만 선택합니다.

![screencapture-github-settings-tokens-new-2023-11-21-11_32_56](https://github.com/zzangoobrother/study-organization/assets/42162127/c2858cce-17e0-4905-8e08-53c70b84a472)



토큰이 생성되면 잃어버리지 않도록 다른 곳에 저장합니다.
한번 보여진 토큰은 해당 화면을 벗어나면 다시는 보여주지 않습니다.

![스크린샷 2023-11-21 오전 11 36 50](https://github.com/zzangoobrother/study-organization/assets/42162127/5835b2ab-abd5-4b21-9c99-35dd0f784fc0)


#### github webhook 설정
배포하고자 하는 레포지토리에서 `Settings` 탭을 선택합니다.
그리고 `Webhooks` 탭을 선택합니다.

![스크린샷 2023-11-21 오전 11 40 08](https://github.com/zzangoobrother/study-organization/assets/42162127/a98ca3c2-c9be-4e62-b5f1-76d88d5a75a2)


`Add webhook` 버튼을 클릭하여 새로운 webhook 를 만듭니다.

Payload URL에는 jenkins 서버 주소와 뒤에 /github-webhook/ 경로를 추가하여 입력합니다.
Content type은 application/json으로 선택합니다.
Add webhook 버튼을 클릭하여 생성합니다.

![스크린샷 2023-11-21 오전 11 41 49](https://github.com/zzangoobrother/study-organization/assets/42162127/2184ee90-6fe4-4904-98c4-569780c28e91)


#### Credentials 만들기
jenkins 대시보드 화면에서 `Jenkins 관리` 화면으로 갑니다.
그리고 `Credentials` 버튼을 클릭합니다.

![스크린샷 2023-11-21 오전 11 52 00](https://github.com/zzangoobrother/study-organization/assets/42162127/0cc624ff-356e-4751-8a0c-cca247643aaf)

`(global)` 버튼을 클릭하고 `Add Credentials` 버튼을 클릭합니다.

![스크린샷 2023-11-21 오전 11 56 58](https://github.com/zzangoobrother/study-organization/assets/42162127/3716502b-0852-4d72-9e5f-c00d914623b9)

![스크린샷 2023-11-21 오전 11 58 23](https://github.com/zzangoobrother/study-organization/assets/42162127/12e598d0-8c01-4bde-9041-26529b6fcba8)

2021년 8월 13일부터 비밀번호를 사용한 인증은 불가능하므로 엑세스 토큰 인증을 사용합니다.

![스크린샷 2023-11-21 오전 11 59 03](https://github.com/zzangoobrother/study-organization/assets/42162127/6f5c0118-c915-45b2-9ebe-1fb0476726e9)

username : Github 사용자 이름
password : Github token
ID : 젠킨스 Credentials ID (Credentials 이름이라 생각하면 된다)

#### 프로젝트 생성
`새로운 Item` 버튼을 클릭한다.

![스크린샷 2023-11-21 오후 1 05 42](https://github.com/zzangoobrother/study-organization/assets/42162127/b1b5fcd9-6ac4-4dfe-9253-72c44c7653bd)

![스크린샷 2023-11-21 오후 1 06 52](https://github.com/zzangoobrother/study-organization/assets/42162127/ef1615e6-9a98-4b4d-bd5f-8570d1be5172)


`Pipeline`으로 생성합니다.
Freestyle project로 만들 수 있지만 Pipeline 으로 만드는 이유는 <a href='https://l-sanghyeup.medium.com/jenkins%EC%97%90%EC%84%9C-ci-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EB%A5%BC-%EC%A0%95%EC%9D%98%ED%95%98%EB%8A%94-2%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-22bf0fb1e608' target='_blank' >Jenkins에서 CI프로세스를 정의하는 2가지 방법</a> 을 참고해주세요.

체크박스를 선택합니다.
- GitHub project
- GitHub hook trigger for GITScm polling

![스크린샷 2023-11-21 오후 1 20 33](https://github.com/zzangoobrother/study-organization/assets/42162127/ea618752-7c5a-42d6-9c2b-9aa153198115)

![스크린샷 2023-11-21 오후 1 20 44](https://github.com/zzangoobrother/study-organization/assets/42162127/79ba6306-56cb-47e5-832e-ec8940881d76)

Pipeline 스크립트 영역에 스크립트를 작성합니다.

![스크린샷 2023-11-21 오후 1 23 34](https://github.com/zzangoobrother/study-organization/assets/42162127/841159d8-5f2c-4175-a32a-324adeb3097c)

```bash
pipeline {
    agent any
    stages {
        stage('source clone') {
            steps {
                git branch: 'master',    // 배포 하고자 하는 브런치를 적습니다.
                credentialsId: 'jenkins-token',  // 위에서 등록한 git token에서 credentials Id를 적습니다.
                url: 'https://github.com/choiskk/demo-cicd.git'  // 배포 하는 레포지토리의 url 를 적습니다.
            }
        }
        
        stage('build') {
            steps {  // 프로젝트를 build 합니다.
                sh "chmod +x gradlew"
                sh "./gradlew clean bootJar"
            }
        }
        
        stage('docker login') {
            steps {  // nCloud에서 docker를 저장하는 Container Registry 저장 공간입니다. Container Registry 에 로그인 합니다.
                sh "docker login -u O5YmX6iPIUKtZw86GF4J -p UEYTEJy0Kp6y132IT1BasaRGdeN9lo8cQIlzYtQL huuim-registry.kr.ncr.ntruss.com"
            }
        }
        
        stage('docker build') {
            steps { docker 를 build 합니다.
                sh "docker build -t testt:v_${env.BUILD_NUMBER} --pull=true /var/lib/jenkins/workspace/test-ci"
            }
        }
        
        stage('docker tag') {
            steps { 
                sh "docker tag testt:v_${env.BUILD_NUMBER} huuim-registry.kr.ncr.ntruss.com/sample/testt:v_${env.BUILD_NUMBER}"
            }
        }
        
        stage('docker push') {
            steps { build 된 docker를 Container Registry 에 업로드 합니다.
                sh "docker push huuim-registry.kr.ncr.ntruss.com/sample/testt:v_${env.BUILD_NUMBER}"
            }
        }
        
        stage('deploy') {
            steps { Container Registry 에 업로드한 docker 를 운영서버로 다운로드 후 실행을 합니다.
                sshagent (credentials: ['prod-server']) {
                    sh '''#!/bin/bash
                        // 8080 포트 확인 후 8080 포트 실행 중이면 8081, 8080 포트가 아니면 8080 포트로 실행합니다.
                        if curl -s "http://223.130.146.36:8080/health" > /dev/null
                        then
                            IDLE_PORT=8080
                            USED_PORT=8081
                        else
                            IDLE_PORT=8081
                            USED_PORT=8080
                        fi

                        // Container Registry 에서 docker 를 다운로드 받습니다.
                        echo "도커 pull"
                        ssh root@223.130.146.36 "docker pull huuim-registry.kr.ncr.ntruss.com/sample/testt:v_${BUILD_NUMBER}"

                        // docker를 실행합니다.
                        echo "도커 이미지 run"
                        ssh root@223.130.146.36 "docker run -d --name prod_back_${USED_PORT} -p ${USED_PORT}:8080 huuim-registry.kr.ncr.ntruss.com/sample/testt:v_${BUILD_NUMBER}"

                        // docker 실행이 완료될 때까지 기다린 후 정상 실행 되었는지 확인 합니다.
                        sleep 30
                        echo "컨테이너 실행 확인"
                        if ! [ curl -o "http://223.130.146.36:${USED_PORT}/health" ];
                        then
                            ssh root@223.130.146.36 "docker stop prod_back_${USED_PORT}"
                            ssh root@223.130.146.36 "docker rm prod_back_${USED_PORT}"
                            echo "TERMINATED"
                            exit 1
                        fi

                        // 기존 실행된 docker를 멈춥니다.
                        echo "컨테이너 stop"
                        ssh root@223.130.146.36 "docker stop prod_back_${IDLE_PORT}"

                        // 기존 docker 컨테이너 삭제 후 이미지도 삭제합니다.
                        echo "컨테이너/이미지 rm"
                        ssh root@223.130.146.36 "docker rm prod_back_${IDLE_PORT}"
                        
                        ssh root@223.130.146.36 "docker rmi huuim-registry.kr.ncr.ntruss.com/sample/testt:v_$(expr ${BUILD_NUMBER} - 1)"
                    '''
                }
            }
        }
    }
}
````

스크립트 설명은 주석으로 대신합다.

![스크린샷 2023-11-21 오후 2 15 43](https://github.com/zzangoobrother/study-organization/assets/42162127/45693617-8e02-497a-9971-705e88a39904)

모두 정상적으로 동작합니다.

