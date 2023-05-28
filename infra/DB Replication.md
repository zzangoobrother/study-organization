## Replication

![mysql-master-slave-hierarchy](https://github.com/zzangoobrother/study-organization/assets/42162127/c798acd9-61ec-44c8-8d55-6036d50a8773)

운영 중인 애플리케이션에서 DB 속도 저하가 발생했을 때 이를 해결하기 위해 쿼리 튜닝, 인덱스 적용, 데이터 경량화, 서버 업그레이드,
DB 이중화 등 여러가지 적용해 볼 수 있다. 그 중 DB Replication이 적용되어 있는 구조라면 애플리케이션 계층에서 특정한 기준에 따라
트래픽을 분산시켜 속도 향상을 볼 수 있다.
위 그림과 같이 Replication 처리되어 동기화되고 있는 Master, Slave EB가 각각 존재한다면
애플리케이션에서 쓰기 작업은 Master DB로 읽기 작업은 Slave DB로 분기시킵니다.
