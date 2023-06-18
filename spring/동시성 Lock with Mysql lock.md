## 동시성 해결 - Mysql lock

일정 수량의 쿠폰 선착순 구현 중 많은 고객들이 동시에 접근을 하면 원래 쿠폰 재고보다 많은 쿠폰이 발행되는 현상이 발생된다.
이를 해결하기 위해 하나의 고객이 쿠폰 발급 후 다른 고객이 접근하여 쿠폰을 발급해야 이 문제를 해결할 수 있을 것이다.
여러가지 방법이 있겠지만 지금은 DB를 이용한 lock를 사용 하겠다.

1. 테이블의 row에 접근시 lock를 걸고 다른 lock이 걸려 있지 않은 경우 수정이 가능하다.
2. 수정할 때 먼저 수정했다고 명시하고, 다른 사람이 동일한 조건으로 값을 수정할 수 없게 합니다.

#### 비관적 락 (pessimistic lock)
비과적 락은 트랜잭션이 시작될 때 shared lock 또는 exclusive lock을 걸고 시작하는 방법입니다.
즉 shared lock을 걸면 write를 하기위해 exclusive lock을 얻어야하는데 shared lock이 다른 트랜잭션에 의해서
걸려 있으면 해당 lock을 얻지 못해 업데이트를 할 수 없습니다.

![20230619-1](https://github.com/zzangoobrother/study-organization/assets/42162127/a365781c-a438-4845-8fb0-88ddee696539)

1. A 에서 table의 id 2를 읽음 (name = karol)
2. B 에서 table의 id 2를 읽음
3. B 에서 table의 id 2의 name을 karol2로 변경 요청, 하지만 A 에서 이미 shared lock를 잡고 있기 때문에 blocking
4. A 에서 트랜잭션 해제 (commit)
5. blocking 되어있던 B의 update 요청이 정상 처리

Transaction을 이용하여 충돌을 예방하는 것이 바로 비관적 락 입니다.

#### 낙관적 락 (optimistic lock)
낙관적 락은 내가 먼저 이 값을 수정했다고 명시하여 다른 사람이 동일한 조건으로 수정할 수 없게 합니다.
이 특징은 db에서 제공해주는 특징이 아닌 application level에서 잡아주는 lock 입니다.

![20230619-2](https://github.com/zzangoobrother/study-organization/assets/42162127/423b2ccb-751d-4b43-8df3-17c541b772a2)

1. A가 table의 id 2를 읽음 (name = karol, version = 1)
2. B가 id 2를 읽음
3. B가 id 2인 row 를 갱신 (name = karol2, version = 2) -> 성공
4. A가 id 2인 row 를 갱신 (name = karol1, version = 2) -> 실패, id 2는 이미 version = 2로 업데이트 되었기에 해당 row를 갱신하지 못함

같은 row에 대해 각기 다른 2개의 수정 요청이 있지만 1개가 업데이트 됨에 따라 version이 변경되어 뒤의 수정 요청은 반영되지 않습니다.
version 뿐만 아니라 hash code 또는 timestamp를 이용하기도 합니다.

#### 롤백
비관적 락과 난관적 락이 어떻게 롤백하는지 알아보도록 합니다.

1. 비관적 락
````sql
SELECT id, `name`
       FROM theTable
       WHERE id = 2;
{새로운 값으로 연산하는 코드}
BEGIN TRANSACTION;
UPDATE anotherTable
   SET col1 = @newCol1,
      col2 = @newCol2
WHERE id = 2;
UPDATE theTable
   SET `name` = 'Karol2',
WHERE id = 2;
{if AffectedRows == 1 }
  COMMIT TRANSACTION;
  {정상 처리}
{else}
  ROLLBACK TRANSACTION;
  {DB 롤백 이후 처리}
{endif}
````
비관적 락은 트랜잭션으로 묶여있기 때문에 수정에 실패하면 db 단에서 전체 롤백 합니다.

2. 낙관적 락
낙관적 락은 트랜잭션을 db에서 잡지 않습니다.
만약 충돌이 발생하면 application 단에서 롤백을 수동으로 합니다.
````sql
SELECT id, `name`, `version`
  FROM theTable
WHERE iD = 2;
{새로운 값으로 연산하는 코드}
UPDATE theTable
  SET val1 = @newVal1,
    version` = `version` + 1
WHERE iD = 2
 AND version = @oldversion;
{if AffectedRows == 1 }
  {정상 처리}
{else}
  {롤백 처리}
{endif}
````

#### 언제, 어디서 사용을 할까?
낙관적 락은 트랜잭션을 필요로 하지 않기 때문에 성능적으로 비관적 락보다 좋습니다.
트랜잭션을 필요로 하지 않기 때문에 로직의 흐름을 가지고 충돌을 감지합니다.
따라서 충돌을 감지하고 해결하려면 개발자가 수동으로 롤백처리를 합니다.
따라서 비용이 많이 듭니다.
때문에 충돌이 많이 일어나지 않을 것이라 보여지는 곳에서 사용하면 좋습니다.

비관적 락은 낙관적 락 반대의 경우에 사용하면 됩니다.
