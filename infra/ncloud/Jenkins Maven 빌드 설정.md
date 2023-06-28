## Jenkins Maven 빌드 설정
Jenkins plugins 화면에서 새로운 plugin을 다운 받습니다. Maven 이라고 검색하시면 나옵니다.

![스크린샷 2023-06-28 오후 5 37 02](https://github.com/zzangoobrother/study-organization/assets/42162127/129799fb-2aa5-4f92-bdf6-48f416ed2283)

Jenkins 관리 → Tools 화면에서 Maven 탭에서 Maven 설정을 합니다.

![스크린샷 2023-06-28 오후 5 37 54](https://github.com/zzangoobrother/study-organization/assets/42162127/696288b6-a3a3-406b-9e7b-4aa9a2fcee47)

Pipeline script에서 Script 작성을 합니다.

````shell
pipeline {
    agent any
    
    tools {
        maven "maven 3.9.1"
    }

    stages {
        stage('github clone') {
            steps {
                git branch: 'main', credentialsId: 'repo-and-hook-access-token', url: 'git url'
            }
        }
        
        stage('build'){
            steps{
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        
        stage('Deploy') {
            steps {
                sshagent(credentials: ['ncloud_key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no root@10.41.4.78 uptime
                        scp /var/jenkins_home/workspace/prod-checku-back/target/jar 파일명.jar root@10.41.4.78:/root
                        ssh -t root@10.41.4.78 sh deploy.sh
                    '''
                }
            }
        }
    }
}
````


