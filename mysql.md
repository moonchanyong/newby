# mysql
### db공부
## keyword 중심


### innoDB
```
이노DB(InnoDB)는 MySQL을 위한 데이터베이스 엔진이며, MySQL AB가 배포하는 모든 바이너리에 내장되어 있다. MySQL과 사용할 수있는 다른 데이터베이스 엔진에 대한 개선 사항으로 PostgreSQL을 닮은 ACID 호환 트랜잭션에 대응하고 있는 것이 있다. 또한 외래 키(FK)도 지원하고 있다. (이것을 선언적 참조 무결성이라 한다.)

innoDB와 대조적으로 MyISAM은 트랜잭션 지원이없다.(참조 무결성 제약 조건과 높은 동시성을 보장하기위해 MySQL5.5부터 innoDB엔진으로 전환했다.)
```

### 실행계획
* 쿼리를 최적으로 실행하기 위해 각  테이블의 데이터가 어떤 분포로 저장 돼 있는지 통계 정보를 참조하여 기본 데이터를 비교해 최적의 실행 계획을 수립하는 작업이 필요하다. (옵티마이저가 이 기능을 담당한다.)

* 실행계획을 이해해야 불합리한 방법을 찾아내고 더욱 최적한 방법으로 실행계획을 수립한다.

#### 쿼리 실행 절차(1, 2는 MySQL엔진에서 처리하고 3은 MySQL엔진과 스토리지 엔진이 동시에 참여한다.)

1. 사용자로부터 요청된 SQL문장을 잘게 쪼개서 MySQL 서버가 이해할 수 있는 수준으로 분리한다.
    * parse tree 생성

2. SQL의 파싱 정보(파스 트리)를 확인하면서 어떤 테이블부터 읽고 어떤 인덱스를 이용해 테이블을 읽을지 선택한다. (SQL서버의 옵티마이저에서 처리)
    * 불필요한 조건의 제거 및 복잡한 연산의 단순화
    * 여러 테이블의 조인이 있는 경우 어떤 순서로 테이블을 읽을지 결정
    * 각 테이블에 사용된 조건과 인덱스 통계 정보를 이용해 사용할 인덱스 결정
    * 가져온 레코드들을 임시 테이블에 넣고 다시 한번 가공해야 하는지 결정
    * 쿼리 실행계획 생성

3. 두 번째 단계에서 결정된 테이블의 읽기 순서나 선택된 인덱스를 이용해 스토리지 엔진으로부터 데이터를 가져온다.

#### 옵티마이저의 종류

##### 비용 기반 최적화 (Cost-based optimizer, CBO)
##### 규칙 기반 최적화 (Rule-based optimizer, RBO)