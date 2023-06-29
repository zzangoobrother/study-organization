해당 배포는 gradle 버전입니다.

![스크린샷 2023-06-29 오전 11 30 24](https://github.com/zzangoobrother/study-organization/assets/42162127/e0a1b403-a79e-4d5e-a4a5-bed1917a3cc2)

젠킨스 사이드바 메뉴에서 Jenkins관리 → Systemp 화면에서 GitHub로 갑니다.

Add 버튼 클릭 github token을 등록합니다.

![스크린샷 2023-06-29 오전 11 31 18](https://github.com/zzangoobrother/study-organization/assets/42162127/c365ad87-36ea-4d2f-aeda-5b231fe96971)

![스크린샷 2023-06-29 오전 11 32 28](https://github.com/zzangoobrother/study-organization/assets/42162127/1da52285-f0f7-442a-a0a9-1727c61f45ab)

팝업 화면에서 Secret text로 변경 후 Secret 필드에 github token을 입력합니다.

github 화면에서 Settings 클릭

![스크린샷 2023-06-29 오전 11 33 32](https://github.com/zzangoobrother/study-organization/assets/42162127/59ae636e-0d5c-4305-b64f-367379e79179)

사이드바 메뉴 가장 아래에 Developer settings 클릭합니다.

그리고 Personal access tokens → Tokens → Generate new token 생성합니다.

![스크린샷 2023-06-29 오전 11 34 01](https://github.com/zzangoobrother/study-organization/assets/42162127/ac801b88-2e13-44fd-8d8c-b3f114f0630e)

![스크린샷 2023-06-29 오전 11 34 42](https://github.com/zzangoobrother/study-organization/assets/42162127/a2d7b073-a8b2-4e84-ab90-e622260f467b)

![스크린샷 2023-06-29 오전 11 35 03](https://github.com/zzangoobrother/study-organization/assets/42162127/bc3bc848-6b71-4cb5-9c94-98c6f61db98a)

token 이 생성되었다면 복사하시고 secret 필드에 붙여넣습니다.

완료 후 Credentials 에서 방금 생성한걸 선택 후 Test connection 버튼을 클릭 하시면 아래와 같은 화면이 나옵니다.

![스크린샷 2023-06-29 오전 11 35 29](https://github.com/zzangoobrother/study-organization/assets/42162127/127c1048-7d8c-44cc-a46c-97f44182bd36)

다시 Add 눌러 Username with password 를 생성합니다.

Username : github 아이디

Password : github token

id : 다른 페이지에서 보일 이름

![스크린샷 2023-06-29 오전 11 36 00](https://github.com/zzangoobrother/study-organization/assets/42162127/9e9d5d65-b2b4-426a-ac57-308de2474f66)

파이프라인을 생성합니다.

젠킨스 메인 화면에서 새로운 Item을 클릭합니다. Pipeline 선택 합니다.

![스크린샷 2023-06-29 오전 11 36 30](https://github.com/zzangoobrother/study-organization/assets/42162127/c5318b79-6b14-4f59-a4cf-5c286a2a4ee2)

Pipeline이 생성되면 새로운 화면이 나올겁니다.

맨 아래 Pipeline 에서 Script 필드 위 콤보박스에서 Hello World를 선택합니다.

![스크린샷 2023-06-29 오전 11 36 50](https://github.com/zzangoobrother/study-organization/assets/42162127/118e8673-409a-4982-a423-d3970a3db1a1)

아래 화면처럼 script가 작성됩니다. script 작성을 통해 git clone과 빌드, jar 파일 실행을 하게 됩니다. 

![스크린샷 2023-06-29 오전 11 37 17](https://github.com/zzangoobrother/study-organization/assets/42162127/2dc3ebb8-90b5-4fab-a31b-3a0e737419ec)

다음으로 Pipeline Syntax 를 클릭합니다. 새로운 탭이 나오고 Sample Step에서 git 을 선택합니다.

![스크린샷 2023-06-29 오전 11 37 37](https://github.com/zzangoobrother/study-organization/assets/42162127/825bc122-1672-46bb-ba17-ee47cb1006b5)

Repository URL에 clone 하고자 하는 URL 입력, branch 입력, Credentials에는 아까 생성한 username/password를 선택할 수 있다.

![스크린샷 2023-06-29 오전 11 38 00](https://github.com/zzangoobrother/study-organization/assets/42162127/a1409b06-9c15-496a-94b9-fbcdee3607c1)

그리고 아래 Generate Pipeline Script 버튼을 클릭하면 필드에 글이 나올것이다. 이 글을 복사합니다.

![스크린샷 2023-06-29 오전 11 38 18](https://github.com/zzangoobrother/study-organization/assets/42162127/9cd8e9b7-8d1d-43d5-b9ee-b2023a85d7e7)

복사 후 아까 Pipeline Script에 복사 후 저장합니다. 그리고 stage 를 이미지처럼 변경합니다.

![스크린샷 2023-06-29 오전 11 38 48](https://github.com/zzangoobrother/study-organization/assets/42162127/be23adbc-057b-41bd-b30c-0dea641af717)

이제 clone할 git 과 연결 되었습니다. 지금 빌드를 클릭합니다.

클릭한다면 github clone이 실행됩니다.

![스크린샷 2023-06-29 오전 11 39 05](https://github.com/zzangoobrother/study-organization/assets/42162127/4424af16-75ee-4fe4-a275-30ef95b303ff)

그리고 #2를 클릭후 Console Output 를 클릭하여 success가 출력되는지 확인합니다.

![스크린샷 2023-06-29 오전 11 40 23](https://github.com/zzangoobrother/study-organization/assets/42162127/472bdb8b-d192-4f00-bb86-bff00fe6e95c)

다시 구성에 들어가서 Pipeline script를 작성합니다.

![스크린샷 2023-06-29 오전 11 40 41](https://github.com/zzangoobrother/study-organization/assets/42162127/a73e2492-82f4-48e1-9fed-e97dc5666d34)

빌드하는 script를 작성 완료 후 저장, 지금 빌드를 클릭 합니다.

![스크린샷 2023-06-29 오전 11 41 00](https://github.com/zzangoobrother/study-organization/assets/42162127/7722c8ab-d70d-48be-ab8c-8f009960486a)

새로 build라는 컬럼이 생기고 성공했습니다. 아까 처럼 #3에 들어간 후 Console Output 창에서 success를 확인 합니다.

이제 젠킨스 서버와 실제 운영되는 서버와 연결 후 빌드된 jar 파일을 운영서버에 전달, 실행 시켜야 한다.

우선 운영서버를 새로 만듭니다. 

생성 하셨다면 dashboard → Jenkins 관리 → Manage Credentials 로 갑니다.

![스크린샷 2023-06-29 오전 11 41 19](https://github.com/zzangoobrother/study-organization/assets/42162127/3a492623-8c61-48e9-8c2a-8d1a65df7cc5)

Stores scoped to Jenkins 에서 System → Global credentials(unrestricted) 아래 화살표 → Add credentials 클릭

![스크린샷 2023-06-29 오전 11 41 35](https://github.com/zzangoobrother/study-organization/assets/42162127/7882c749-5212-43da-b886-5171b6cf94d0)

kind를 SSH Username with private key 로 변경

![스크린샷 2023-06-29 오전 11 41 53](https://github.com/zzangoobrother/study-organization/assets/42162127/7e980cd5-dcbe-4095-aca7-f22450511728)

id에는 어떤 key인지 쓰시고 Username에는 root를 적습니다.

Treat username as secret 체크, Enter directly 체크 후 key에 젠킨스 서버의 id_rsa key를 넣습니다.

그럼 젠킨스 서버에서 key를 생성 하겠습니다.

젠킨스 서버에서 아래 코드를 입력, 입력 후 나오는 것들은 엔터키를 누르면 됩니다.

```bash
$ ssh-keygen -t rsa
```

```bash
$ ls -al .ssh/
```

위 코드를 입력하면 아래 이미지 처럼 나옵니다.

![스크린샷 2023-06-29 오전 11 42 12](https://github.com/zzangoobrother/study-organization/assets/42162127/22a46291-3830-4ee4-a6bd-4d32cc1179e7)

```bash
$ cat ~/.ssh/id_rsa
```

위 코드를 입력 후 나오는 모든 글을 복사 후 붙여넣습니다.

![스크린샷 2023-06-29 오전 11 42 33](https://github.com/zzangoobrother/study-organization/assets/42162127/f0ccfe0d-1882-49e5-87cf-1ceeecd9ef06)

![스크린샷 2023-06-29 오전 11 42 49](https://github.com/zzangoobrother/study-organization/assets/42162127/53f7f33d-2bd4-49a8-aaf8-02e96b0c8529)

입력 완료 후 create를 클릭 합니다.

이제 다시 기존 작업 중인 Item → 구성에 가서 나머지 Pipeline script를 작성합니다.

```bash
stage('Deploy') {
            steps {
                sshagent(credentials: [위 이미지 ID]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no root@비공인ip uptime
                        scp /var/jenkins_home/workspace/test1/build/libs/cicdtesttt-0.0.1-SNAPSHOT.jar root@비공인ip:/root
                        ssh -t root@비공인ip sh deploy.sh
                    '''
                }
            }
        }
```

![스크린샷 2023-06-29 오전 11 43 12](https://github.com/zzangoobrother/study-organization/assets/42162127/636e6437-3fed-4952-850d-98e693fb3747)

우리가 만든 **ncloud-key 로 운영서버 접근을 확인 합니다.**

이렇게 작성 후 지금 빌드를 하시면 에러가 발생합니다. 에러 내용은 plugin 설치 입니다.

plugin 설치는 SSH Agent 입니다.

Jenkins 관리 → plugins 로 갑니다.

Available plugins 에서 SSH Agent 검색 후 설치 합니다.

SSH Agent 체크 박스 선택 후 Install without restart 클릭, 설치 완료 후 메인 화면으로 갑니다.

재시작 안 하셔도 됩니다.

설치 완료 후 다시 지금 빌드를 합니다.

실패 하실 겁니다. 아직 운영서버로 접근이 불가능합니다.

젠킨스 서버와 운영서버에서 rsa 키를 만들어 공유 할 수 있게 설정합니다.(aws는 이 과정이 없을 수 있습니다.)

운영 서버도 위와 같이 합니다.

이제 젠킨스 서버에 운영서버를 등록합니다.

젠킨스 서버에서

```bash
$ ssh-keyscan -H 운영서버 비공인ip >> ~/.ssh/known_hosts
```

입력 합니다.

젠킨스 서버에서 생성한  공개키를 운영서버에 등록합니다.

젠킨스 서버 코드

```bash
$ cat ~/.ssh/id_rsa.pub
```

![스크린샷 2023-06-29 오전 11 43 40](https://github.com/zzangoobrother/study-organization/assets/42162127/3432a4da-576c-40ce-9d82-3a044a47071b)

운영서버 코드

```bash
$ vi ~/.ssh/authorized_keys
```

![스크린샷 2023-06-29 오전 11 43 57](https://github.com/zzangoobrother/study-organization/assets/42162127/9c0bef5f-548a-4aef-b245-33148354ba81)

위 과정까지 마쳤다면 다시 빌드를 해보시면 정상이라면 운영서버와 연결 되었습니다.

다음으로 scp 명령어는 docker에서 빌드된 jar 파일을 운영서버에 보내는 것입니다.

이때 경로를 잘 파악 해서 넣으시면 문제 없이 보내집니다.

다음으로 [deploy.sh](http://deploy.sh) 작성입니다.

```bash
echo "> pid 확인"
CURRENT_PID=$(ps -ef | grep java | grep cicd | grep -v nohup | awk '{print $2}')
echo "&CURRENT_PID"

if [ -z ${CURRENT_PID}] ;then
echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않음."
else
        echo "> sudo kill -9 $CURRENT_PID"
        sudo kill -9 $CURRENT_PID
        sleep 10
fi

nohup java -jar cicdtesttt-0.0.1-SNAPSHOT.jar > /dev/null 2>&1 &
```

위 코드를 복사후 운영서버에 [deploy.sh](http://deploy.sh) 파일을 생성 해주세요.

다시 빌드를 해보시면 모든 과정이 성공할 것입니다.

이제 마지막으로 github webhook 설정입니다.

구성에 들어가서 아래 이미지처럼 체크박스 체크 후 저장합니다.

![스크린샷 2023-06-29 오전 11 44 16](https://github.com/zzangoobrother/study-organization/assets/42162127/d7e557d4-cdf1-4bf7-af5c-ca11f4063f34)

배포 하고자 하는 repository에서 Settings → Webhooks에서 생성합니다.

젠킨스 서버 주소를 입력합니다.

![스크린샷 2023-06-29 오전 11 44 33](https://github.com/zzangoobrother/study-organization/assets/42162127/e64aa689-0e1b-4ea7-b526-e2ca474cfbff)

아래 이미지처럼 URL 주소 옆에 초록 체크가 뜨면 정상입니다.

![스크린샷 2023-06-29 오전 11 44 50](https://github.com/zzangoobrother/study-organization/assets/42162127/859fc0ae-b62d-4388-87e5-cffb892b3e8e)

이제 소스를 수정 해보시고 push 해보시면 정상 배포가 될 것입니다.
