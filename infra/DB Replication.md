## Replication

![mysql-master-slave-hierarchy](https://github.com/zzangoobrother/study-organization/assets/42162127/c798acd9-61ec-44c8-8d55-6036d50a8773)

운영 중인 애플리케이션에서 DB 속도 저하가 발생했을 때 이를 해결하기 위해 쿼리 튜닝, 인덱스 적용, 데이터 경량화, 서버 업그레이드,
DB 이중화 등 여러가지 적용해 볼 수 있다. 그 중 DB Replication이 적용되어 있는 구조라면 애플리케이션 계층에서 특정한 기준에 따라
트래픽을 분산시켜 속도 향상을 볼 수 있다.
위 그림과 같이 Replication 처리되어 동기화되고 있는 Master, Slave EB가 각각 존재한다면
애플리케이션에서 쓰기 작업은 Master DB로 읽기 작업은 Slave DB로 분기시킵니다.

서버에서 docker를 이용해 구성하기

````bash
$ docker pull mysql

// docker 이미지가 잘 다운되었는지 확인
$ docker images
````
이미지를 master, slave를 docker로 실행

````bash
$ docker run -p 3306 --name mysql-master -e MYSQL_ROOT_PASSWORD=1234 -d docker.io/mysql
````
mysql-master라는 이름으로 3306포트에 mysql 실행 완료
docker의 exec 명령어로 mysql 내부 접속
````bash
$ docker exec -it mysql-master /bin/bash
````
mysql 내부에 vim이 없으므로 설치 합니다.
````bash
$ apt-get update
$ apt-get install -y vim
````

/etc/mysql/my.cnf 파일을 열고, 다음 2줄을 추가합니다.
````bash
log-bin=mysql-bin  
server-id=1
````
그리고 docker를 재시작합니다.
````bash
$ docker restart mysql-master
````
설정이 제대로 되었는지 확인합니다.
````bash
$ docker exec -it mysql-master /bin/bash
$ mysql -u root -p 
mysql> SHOW MASTER STATUS\G
````
## Master DB 계정 생성
Master 계정은 Slave DB로 복제를 합니다. 모든 ip에 대해 권한을 열어줍니다.
````bash
$ CREATE USER 'bookroot'@'%' IDENTIFIED BY '1234';
$ ALTER USER 'bookroot'@'%' IDENTIFIED WITH mysql_native_password BY '1234';
$ GRANT REPLICATION SLAVE ON *.* TO 'bookroot'@'%';
$ FLUSH PRIVILEGES;
````
mysql의 USER 테이블을 확인하여 생성한 계정이 있는지 확인 합니다.

DB에서 book 사용자를 생성합니다.
````bash
$ CREATE DATABSE book;
````
book에서 testtable 테이블을 만들고 test row 라는 데이터를 넣습니다.
````bash
$ use book;
$ create table testtable { text varchar(20) };
$ insert into testtable values ('test row');
$ select * from testtable;
````

Master DB에서 dump를 합니다.
````bash
$ mysqldump -u root -p bookclub > dump.sql
````

mysql 접속을 종료하고, 로컬 환경에서 dump.sql를 가져옵니다.
````bash
$ docker cp mysql-master:dump.sql .
$ cat dump.sql
````

#### Slave DB 계정 생성하기
Slave DB 생성을 위해 docker를 이용 새로운 mysql를 생성합니다.
````bash
$ docker run -p 3306 --name mysql-slave -e MYSQL_ROOT_PASSWORD=1234 --link mysql-master -d docker.io/mysql
````
생성이 완료 되었다면 확인을 합니다.
````bash
$ docker ps
````
Master, Slave DB 2개가 실행되어 있을 겁니다.

Slave DB mysql에 접속해 vim을 설치합니다.
````bash
$ docker exec -it mysql-slave /bin/bash
$ apt-get update; apt-get install vim -y
````
/etc/mysql/my.cnf에 접속해 log-bin과 server-id를 추가합니다.
````bash
log-bin=mysql-bin  
server-id=2
````
docker를 재시작합니다.
````bash
docker restart mysql-slave
````
로컬 환경에 있는 dump 파일을 Slave DB에 복사합니다.
Slave DB에 접속해 dump 파일을 적용합니다.
````bash
$ docker cp dump.sql mysql-slave:.
$ docker exec -it mysql-slave /bin/bash

$ mysql -u root -p
mysql> CREATE DATABASE bookclub;

mysql> exit

$ mysql -u root -p bookclub < dump.sql
````
Master DB에서 생성한 testtable과 데이터가 복제되어 있습니다.

Master DB mysql에 접속해서 SHOW MASTER STATUS\G를 확인합니다.
처음에 비해 DB 테이블과 쿼리들이 추가됐으므로 이전보다 Position이 증가했습니다.

#### Slave DB와 Master DB 연결
Slave DB mysql에 접속해, Master를 mysql-master로 변경하고 Slave DB를 시작합니다.
````bash
$ docker exec -it mysql-slave /bin/bash

$ mysql -u root -p 

mysql> CHANGE MASTER TO MASTER_HOST='mysql-master', MASTER_USER='bookroot', MASTER_PASSWORD='1234', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=1721;

mysql> START SLAVE;
````
명령어 설명
MASTER_HOST : master 서버의 호스트명
MASTER_USER : master 서버의 mysql에서 REPLICATION SLAVE 권한을 가진 User 계정의 이름
MASTER_PASSWORD : master 서버의 mysql에서 REPLICATION SLAVE 권한을 가진 User 계정의 비밀번호
MASTER_LOG_FILE : master 서버의 바이너리 로그 파일명
MASTER_LOG_POS : master 서버의 현재 로그의 위치

Slave DB에서 연결정보를 조회해보면 mysql-master와 연결된 정보가 나옵니다.
````bash
mysql> SHOW SLAVE STATUS\G
````

mysql-master에서 데이터를 insert하고 mysql-slave에서 조회하면 insert한 데이터가 나옵니다.

### Spring Master/Slave 설정
````java
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

import static org.springframework.transaction.support.TransactionSynchronizationManager.isCurrentTransactionReadOnly;

public class DynamicRoutingDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return isCurrentTransactionReadOnly() ? "slave" : "master";
    }

}

@RequiredArgsConstructor
@Configuration
@EnableTransactionManagement
public class RoutingDataSourceConfig {

    private final Environment env;

    @ConfigurationProperties(prefix = "spring.datasource.hikari.master")
    @Bean
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @ConfigurationProperties(prefix = "spring.datasource.hikari.slave")
    @Bean
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @DependsOn({"masterDataSource", "slaveDataSource"})
    @Bean
    public DataSource routingDataSource(
            @Qualifier("masterDataSource") DataSource master,
            @Qualifier("slaveDataSource") DataSource slave) {
        DynamicRoutingDataSource routingDataSource = new DynamicRoutingDataSource();

        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put("master", master);
        dataSourceMap.put("slave", slave);

        routingDataSource.setTargetDataSources(dataSourceMap);
        routingDataSource.setDefaultTargetDataSource(master);

        return routingDataSource;
    }

    @DependsOn({"routingDataSource"})
    @Bean
    public DataSource dataSource(DataSource routingDataSource) {
        return new LazyConnectionDataSourceProxy(routingDataSource);
    }

    @Primary
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource) {

        LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
        em.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        em.setDataSource(dataSource);
        em.setPackagesToScan(new String[]{"com.flab.goodchoice"});
        em.setPersistenceUnitName("master");

        Map<String, Object> properties = new HashMap<>();
        properties.put("hibernate.physical_naming_strategy",
                SpringPhysicalNamingStrategy.class.getName());
        properties.put("hibernate.implicit_naming_strategy",
                SpringImplicitNamingStrategy.class.getName());
        properties.put("hibernate.hbm2ddl.auto", env.getProperty("spring.jpa.hibernate.ddl-auto"));
        em.setJpaPropertyMap(properties);

        return em;
    }

    @Primary
    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(entityManagerFactory);

        return transactionManager;
    }

}
````
