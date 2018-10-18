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


##### 규칙 기반 최적화 (Rule-based optimizer, RBO, 망한 기술)
* 기본저긍로 대상 테이블의 레코드 건수나 선택도 등을 고려하지않음
* 옵티마이저에 내장된 우선순위에 따라 실행계획을 수립
* 같은 쿼리에 따라 같은 실행방법을 만든다.

##### 비용 기반 최적화 (Cost-based optimizer, CBO)
* 여러가지 방법중, 각 단위 작업의 비용 정보와 대상 테이블의 예측 된 통계정볼르 이용해 각 실행 계획별 비용 산출
* 각 계획 중 최소비용이 소요되는 방식을 택함

###### 통계정보 
* MySQL은 이 통계정보가 다양하지 않다.(동적으로 순간순간 자동으로 변경) <-> 오라클은 정적이고 수집에 많은 시간 소요(백업하는 경우도 있음)
* 레코드 건 수, 인덱스의 유니크한 값의 개수 정도
* 통계정보가 부정확한 경우, ANALYZE로 강제적으로 통계 정보를 갱신해야 할 때도 있다.

#### 실행계획 분석
```
    EXPLAIN 
    /* 쿼리문 */
```
* select_type
    * SELECT 쿼리가 어떤 타입의 쿼리인지 표시 된다.
    * SIMPLE: UNION 이나 서브 쿼리를 사용하지 않는 단순한 SELECT 쿼리인 경우
        * 아무리 복잡한 쿼리라도 SIMPE인 단위 쿼리는 반드시 하나 존재한다.
        * 일반적으로 제일 바깥 SELECT쿼리의 select_type이 SIMPLE로 표시된다.
    * PRIMARY: UNION이나 서브쿼리가 포함된 SELECT 쿼리의 실행계획에서 제일 바깥쪽에 있는 단위쿼리
        * 하나만 존재하며, 제일 바깥쪽이 PRIMARY로 포함된다.
    * UNION: UNION으로 결합하는 단위 SELECT 쿼리 가운데 첫 번째를 제외한 두 번째 이후 단위 SELECT 쿼리의 select_type
        * UNION의 첫 번째 단위 SELECT는 select_type이 UNION이 아니라 UNION 쿼리로 결합된 전체 집합의 select_type이 표시된다.
        * UNION의 첫 번째 타입은 UNION결과를 대표하는 select_type으로 설정됨
    * DEPENDENT UNION: UNION이나 UNIONALL로 결합된 단위 쿼리가 외부의 영향을 받는 것
        * 서브쿼리가 사용된 경우에는 외부 쿼리보다 서브쿼리가 먼저 실행되는 것이 일반적이며 
        빠르게 처리되지만, DEPENDENT 키워드를 포함하는 서브쿼리는 외부 쿼리에 의존적이며 절대 외부쿼리보다 먼저 실행될 수 없다. 그래서 DEPENDENT 키워드가 포함된 서버쿼리는 비효율 적인 경우가 많다.
    ``` sql
    -- 예시 쿼리
    EXPLAIN 
    SELECT
    e.first_name,
    (   SELECT CONCAT('Salary change count : ', COUNT(*)) AS message
        FROM salaries s WHERE s.emp_no=e.emp_no -- 단위쿼리가 외부의 값을 참조해서 처리한다. e.emp_no
        UNION 
        SELECT CONCAT('Department change count : ', COUNT(*)) AS message
        FROM dept_emp de WHERE de.emp_no=e.emp_no
        UNION
    ) AS message
    FROM employees e
    WHERE e.emp_no=10001
    ```