.

Data_Engineering_TIL(20220102)

[학습자료]

GCP 공식문서를 공부하고 정리한 내용입니다.

** URL : https://cloud.google.com/bigquery/docs/time-travel

[학습내용]

- 빅쿼리는 time-travel이라는 기능을 사용하여 과거의 데이터를 액세스 할 수 있음


- 도큐먼트 설명 내용

"BigQuery에서는 시간 이동을 사용하여 변경되었거나 삭제된 BigQuery에 저장된 데이터에 액세스할 수 있습니다. 이전 7일 이내의 어느 시점에서도 데이터에 액세스할 수 있습니다. 시간 이동을 사용하여 업데이트 또는 삭제된 데이터를 쿼리하거나, 삭제된 테이블을 복원하거나, 만료된 테이블을 복원할 수 있습니다."


- time-travel 기능의 제한사항

"시간 이동은 이전 7일 동안의 테이블 데이터에 대한 액세스만 제공합니다. 테이블 데이터를 7일 이상 보존하려면 테이블 스냅샷을 사용합니다."

** 빅쿼리 테이블 스냅샷 기능 : https://cloud.google.com/bigquery/docs/table-snapshots-intro


--> 다시말해서 7일 이전의 데이터에 대해서는 time-travel 기능을 이용해서 복원이 가능하다는 얘기이다.

- time-travel 기능 예시 : 1시간 전의 이전 테이블 버전을 반환하는 쿼리

```SQL
SELECT * FROM `mydataset.mytable`
FOR SYSTEM_TIME AS OF TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR);
```

"`FOR SYSTEM_TIME AS OF` 절을 사용하여 이전 7일 이내의 어느 시점에서든 테이블의 이전 데이터를 쿼리할 수 있습니다. 이 절은 상수 타임스탬프 표현식을 사용해서 해당 타임스탬프에서 현재였던 테이블 버전을 참조합니다. 테이블은 BigQuery에 저장되어야 하며, 외부 테이블일 수 없습니다. `SYSTEM_TIME AS OF`를 사용할 때는 테이블 크기에 제한이 없습니다."