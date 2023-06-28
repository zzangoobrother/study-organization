## Jenkins 설치
빌드할 프로젝트 java 버전에 맞게 java를 설치합니다. 지금은 java 17를 설치합니다.
```shell
$ sudo docker run -d --name jenkins -p 8080:8080 jenkins/jenkins:jdk17
````

- d : 컨테이너를 데몬으로 띄운다.(데몬 스레드에 대해 알아보시면 좋습니다.)
- name : 컨테이너 이름입니다. 개발자가 편한대로 지으시면 됩니다.
- p 8080:8080 : 컨테이너의 외부와 통신할 포트를 내부적으로 사용할 포트와 포워딩해준다.

이 말은 즉, 외부에서 8080 포트로 접속하면 내부에서 8080포트를 사용하는 프로그램과 연결한다는 뜻입니다.

NCloud에서는 ACG 에서 포트를 열어줘야 접근이 가능합니다.  
젠킨스는 모두 접속 가능하게 0.0.0.0/0으로 설정했고, 8080 포트 허용을 추가 했습니다.  
이제 젠킨스에 접속합니다. 공인 ip:8080 으로 접속합니다.

![스크린샷 2023-06-28 오후 4 51 03](https://github.com/zzangoobrother/study-organization/assets/42162127/08be69fc-3d7e-4dd3-8ec2-8bd925c05c76)

비밀번호를 입력해야 합니다. 비밀번호는 서버에서 찾습니다.  
젠킨스 docker에 접속 후 비밀번호를 확인 합니다.
````shell
$ sudo docker exec -it jenkins bash
$ cat /var/jenkins_home/secrets/initialAdminPassword
````

비밀번호를 입력하면 아래 화면이 나옵니다.  
![스크린샷 2023-06-28 오후 4 58 51](https://github.com/zzangoobrother/study-organization/assets/42162127/7c1c102c-9dfc-4255-9ef9-84776ef22c6d)

install를 클릭 합니다.  
![스크린샷 2023-06-28 오후 4 59 28](https://github.com/zzangoobrother/study-organization/assets/42162127/ffa4a6a3-dfdf-461e-a642-16be40803b9a)

![스크린샷 2023-06-28 오후 5 00 05](https://github.com/zzangoobrother/study-organization/assets/42162127/6d1e8e4a-36c5-466c-b39c-849cb8afef12)

설치가 완료되면 사용할 계정명과 암호를 입력 합니다.

![스크린샷 2023-06-28 오후 5 03 18](https://github.com/zzangoobrother/study-organization/assets/42162127/74d48d12-463c-4428-b8aa-44153796280a)

url 확인 후 finish 합니다.

![스크린샷 2023-06-28 오후 5 09 17](https://github.com/zzangoobrother/study-organization/assets/42162127/2cbbb8e2-8c60-4005-a28e-710c5156674f)

설치가 완료 되었습니다.

