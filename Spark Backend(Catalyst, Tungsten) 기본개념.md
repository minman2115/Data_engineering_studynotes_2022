.

Data_Engineering_TIL(20220103)

[참고자료]

패스트캠퍼스 "올인원 패키지 : 실시간 빅데이터 처리를 위한 Spark & Flink Online" 강의내용

URL : https://fastcampus.co.kr/data_online_flink

[학습내용]

- Spark은 Catalyst와 Tungsten이라는 핵심적인 최적화 엔진을 사용해서 연산을 최적화한다.

```text
                        Catalyst 엔진을 이용하여 최적화            Tungsten 엔진을 이용하여 최적화
SQL or Dataframe -->       Query Plan Optimization          -->     RDD

```

- Catalyst는 하이레벨의 SQL과 DataFrame과 같은 구조가 있는 데이터를 로우레벨의 RDD로 연산할 수 있게 해주는 모듈이라고 할 수 있음

```text
하이레벨 API        MLLib       Spark Streaming
                   Spark SQL      DataFrame
                           Catalyst
로우레벨 API            Spark Core (RDD)
```

- Catalyst는 Logical Plan을 Physical Plan으로 바꿔주는 모듈임


1) Logical plan : 


(1-1) 수행해야하는 모든 transformation단계에 대한 추상화


(1-2) 데이터가 어떻게 변해야 하는지 정의하지만


(1-3) 실제 어디서 어떻게 동작 하는지는 정의하지 않음 


2) Physical Plan :


(2-1) Logical Plan이 어떻게 클러스터 위에서 실행될지 정의


(2-2) 실행 전략을 만들고 Cost Model에 따라 최적화


- Catalyst가 Logical Plan을 Physical Plan으로 바꾸는 프로세스

STEP 1) 분석: DataFrame객체의 relation을 계산, 칼럼의 타입과 이름 확인

STEP 2) Logical Plan 최적화

step 2-1) 상수로 표현된 표현식을 Compile Time에 계산 (x runtime)

step 2-2) Predicate Pushdown: join & filter --> filter & join, 필터를 먼저해서 연산량 억제

step 2-3) Projection Pruning: 연산에 필요한 칼럼만 가져오기, 풀스캔 억제

STEP 3) Physical Plan 만들기: Spark에서 실행 가능한 Plan으로 변환

STEP 4) 코드 제너레이션: 최적화된 Physical Plan을 Java Bytecode

```text

                             분석            최적화                                                      -->
SQL or Dataframe -->       Logical plan       -->     최적화된 Logical plan   -->    physical planning   -->
                                                                                                        -->

--> physical plan                                                         code generation
--> physical plan     -->   cost model    -->   optimized physical plan        -->               RDD
--> physical plan

```

- spark sql에서 plan 확인하는 방법

```python
spark.sql(query).explain(True)
```

확인가능한 plan 목록

(1) Parsed Logical Plan

(2) Analyzed Logical Plan

(3) Optimized Logical Plan

(4) Physical Plan

- Tungsten 이란

Physical Plan이 선택되고 나면 분산 환경에서 실행될 Bytecode가 만들어지게 되는데 이 프로세스를 Code Generation이라고 부른다. 이때 Tungsten 엔진이 가동되어 성능을 최적화한다.

Tungsten 엔진으로 최적화 하는 부분 : 메모리 관리 최적화, 캐시 활용 연산, 코드 생성