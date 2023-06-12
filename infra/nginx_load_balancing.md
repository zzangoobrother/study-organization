예제는 ubuntu 18.04 로 실습합니다.

우선 nginx를 설치하기 위해 ubuntu는 설정으 해야 합니다. centOS는 따로 설정이 필요 없습니다.
````bash
$ apt install curl gnupg2 ca-certificates lsb-release
````

nginc 저장소 설정
````bash
$ echo "deb http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
````

공개키 추가
````bash
$ curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
````

Verify that you now have the proper key
````bash
$ sudo apt-key fingerprint ABF5BD827BD9BF62
````

NGINX 설치
````bash
$ sudo apt update
$ sudo apt install nginx
````

nginx 실행 및 확인
````bash
$ sudo service nginx status  // 상태확인
$ sudo service nginx start   // 시작
$ sudo service nginx restart // 재시작
````

설치가 완료되면 nginx config 파일 설정을 해야 합니다.
우선 로드밸런싱을 하기 위해 설정을 하겠습니다.
````bash
$ cd /etc/nginx
````

위 경로로 들어 갑니다. nginx.conf 파일을 수정합니다.

<img width="595" alt="11" src="https://github.com/zzangoobrother/study-organization/assets/42162127/9082eee5-fda5-4178-bb46-a381c0b56cbb">

````bash
upstream 이름 아무거나 {
	server 운영서버 private ip:8080 weight=100 max_fails=3 fail_timeout=3s;
	server 운영서버 private ip:8080 weight=100 max_fails=3 fail_timeout=3s;
}
````

로드밸런싱 하기 위해 추가된 부분입니다.
운영서버 대수 만큼 추가하면 됩니다.
다음 수정입니다.
````bash
$ cd /etc/nginx/conf.d
````

위 경로로 들어 갑니다. default.conf 파일을 수정합니다.
````bash
$ vi default.conf
````

<img width="588" alt="22" src="https://github.com/zzangoobrother/study-organization/assets/42162127/683a98e3-a2c9-45f5-822f-47e8fc90ac6d">

위에서 설정된 내용을 proxy에 적용합니다.
````bash
proxy_pass http://이름 아무거나;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection 'upgrade';
proxy_set_header Host $host;
proxy_cache_bypass $http_upgrade;
````

nginx를 재시작하고 테스트 해봅니다.
