# Oracle DB의 인덱스
### 인덱스
인덱스: 테이블의 특정 컬럼에 대한 검색 속도를 빠르게 하기 위해 사용하는 객체
  - 테이블의 데이터를 직접 저장하지 않고, 선택한 컬럼값과 해당 데이터의 **ROWID**(데이터가 실제 저장된 위치)를 함께 저장
  - SQL에서 **WHERE, JOIN, ORDER BY, GROUP BY** 등에 사용되는 컬럼에 인덱스를 생성하면 조회 쿼리 성능을 높일 수 있음
  - ⚠️ 하지만, **INSERT, UPDATE, DELETE** 시에는 인덱스도 함께 갱신해야 하므로 오버헤드 발생 -> 모든 컬럼에 인덱스를 생성하는 것은 비효율적
  - ⚠️ **인덱스 사용 주의사항**
    - 인덱스가 효율적일 때
      - 검색되는 데이터의 비율이 적은 경우
      - 위에 말한것처럼 조건절에 사용되는 컬럼
    - 인덱스가 비효율적일 때
      - 테이블의 대부분의 데이터를 읽어와야할 때 -> FULL SCAN으로
      - DML(데이터 수정 및 삭제)시 인덱스 갱신 -> 성능 저하
  - 인덱스 종류
    - B-tree 인덱스 (기본 인덱스)
      - 균형 이진 트리(Balance Tree) 형태로 데이터 저장
      - 범위 검색, equal(=) 검색, 정렬된 결과가 빠름
      ```sql
      CREATE INDEX idx_emp_name
      ON employees(name);
      ```
    - Bitmap 인덱스
      - 값의 종류가 적은 컬럼(선택도가 낮은)에 적합
      - 각 값을 bit map으로 표현하여 공간 절약과 빠른 검색 가능 -> 대용량 데이터 분석(OLAP)에 자주 사용됨
      ```sql
      CREATE BITMAP INDEX idx_emp_gender
      ON employees(gender);
      ```
    - Hash 인덱스
      - 해시 함수를 사용해 키 값을 bucket에 매핑
      - 특정 값에 대한 검색은 해시 함수 -> 버킷 -> 데이터 방식으로 **O(1)**에 가까운 속도로 빠르게 검색
      - **정확히 일치하는 검색**에 매우 빠름 -> 범위 검색(>, <, BETWEEN)은 비효율적 -> 정렬되어 있지 않기 때문
      - Oracle DB에서는 기본적으로 해시 인덱스를 제공하지 않고 Hash Cluster로 비슷한 기능을 구현한다고 함
      ```sql
      CREATE CLUSTER emp_cluster (emp_id NUMBER(5))
      HASHKEYS 1000; -- 해시 버킷의 개수
      ```
    - 유니크 인덱스
      - 인덱스 컬럼 값이 중복될 수 없음을 보장함 -> 즉, PRIMARY KEY, UNIQUE 제약조건 사용시 자동으로 생성되는 인덱스
      ```sql
      CREATE UNIQUE INDEX idx_emp_email
      ON employees(email);
      ```
    - 함수 기반 인덱스 (Function-based Index)
      - 컬럼 값에 함수나 표현식을 적용한 결과를 인덱스로 생성
      ```sql
      CREATE INDEX idx_emp_upper_name
      ON employees(UPPER(name));
      ```
    - 복합 인덱스
      - 여러 컬럼을 결합하여 인덱스 생성, 순서가 중요함
        - 순서가 중요한 이유: 인덱스가 왼쪽부터 정렬된 상태로 구성되기 때문
      ```sql
      CREATE INDEX idx_emp_dept_job
      ON employees(dept_id, job_id);
      ```

### 복합인덱스에서 순서가 중요한 이유
> `B-tree 인덱스 구조`, `선두 컬럼(Leading Column) 원칙`

복합 인덱스의 구조와 순서
```sql
CREATE INDEX idx_emp_dept_job
ON employees(dept_id, job_id);
```
- 이 인덱스는 dept_id ->  job_id 순서로 정렬됨

예시 데이터
```
(dept_id=10, job_id=CLERK) -> ROWID
(dept_id=10, job_id=MANAGER) -> ROWID
(dept_id=20, job_id=CLERK) -> ROWID
```
- dept_id가 먼저 정렬되고, 같은 dept_id 내에서 job_id가 정렬됨
- 그래서 WHERE 조건에서 dept_id가 빠지면 job_id 단독 검색에 이 인덱스를 활용하기 어려움

선두 컬럼 원칙(Leading Column Rule)
- 인덱스 사용 조건: 복합 인덱스가 생성된 순서에 맞춰 WHERE 조건에 포함되어야 인덱스를 효과적으로 사용함

(dept_id, job_id)의 복합 인덱스가 있다면, 
```sql
-- 1) 인덱스 사용 (dept_id 조건이 선두 컬럼임)
SELECT * FROM employees WHERE dept_id = 10;

-- 2) 인덱스 사용 (dept_id + job_id 조합)
SELECT * FROM employees WHERE dept_id = 10 AND job_id = 'CLERK';

-- 3) 인덱스 사용 불가 (job_id 단독 검색)
SELECT * FROM employees WHERE job_id = 'CLERK';
```
- job_id 만으로는 B-tree의 첫번째 레벨에서 어떤 dept_id 블록부터 탐색해야할지 알 수 없음

복합 인덱스 순서를 결정하는 법
- 선택도가 높은 컬럼을 먼저 배치함
  - Ex) dept_id보다 employee_id가 훨씬 다양한 값을 가진다면 (employee_id, dept_id) 순서가 효율적
  - **쿼리 패턴을 분석하여 WHERE 절에 가장 자주 등장하는 컬럼을 앞에 배치**
 
### IN 조건과 인덱스 사용
IN 조건은 사실상 **OR 조건을 여러번 실행하는 것과 동일**

예를 들어 복합 인덱스 (dept_id, job_id)가 있다면,
```sql
SELECT *
FROM employees
WHERE dept_id IN (10, 20) AND job_id = 'CLERK';
```
위 쿼리의 실행계획을 보면
```
(dept_id=10 AND job_id='CLERK')
OR
(dept_id=20 AND job_id='CLERK')
```
- 선두 컬럼에 IN이 있어도 결국엔 인덱스를 활용하여 조회함
- 하지만, 선두 컬럼이 없고 후행 컬럼인 job_id만 IN 조건이 붙으면 인덱스 사용 불가능

### LIKE 조건과 인덱스 사용
LIKE 조건은 **패턴의 시작 위치**가 중요함

앞 부분이 고정된 경우(인덱스 사용 가능)
```sql
SELECT *
FROM employees
WHERE name LIKE 'A%';
```
- 'A'로 시작하는 데이터를 찾으므로 B-tree 인덱스를 이용해 빠르게 조회 가능
- 'A~~~' 범위 검색이 가능하므로 인덱스 효율이 높음

앞에 와일드카드가 있는 경우(인덱스 사용 불가능)
```sql
SELECT *
FROM employees
WHERE name LIKE '%SON';
```
- '%SON' 앞이 고정되어 있지 않아 FULL SCAN 유발
- 이럴 때에는 함수 기반 인덱스나 리버스 인덱스를 사용하면 됨

실행 계획 확인해보기
```sql
EXPLAIN PLAN FOR
SELECT *
FROM employees WHERE dept_id = 10 AND job_id IN ('CLERK', 'MANAGER');

SELECT *
FROM TABLE(DBMS_XPLAN.DISPLAY);
```
- 실행계획에서 INDEX RANGE SCAN, INDEX SKIP SCAN, FULL TABLE SCAN 등을 확인할 수 있음

### 성능 최적화하기
- LIKE '%value%' 처럼 앞에 %가 붙으면 인덱스를 사용할 수 없음
  - **CONTAINS() (Oracle Text Index)나 INSTR() + 함수 기반 인덱스** 고려
    ```sql
    CREATE INDEX idx_emp_name_text
    ON employees(name) INDEXTYPE IS CTXSYS.CONTEXT;
  
    SELECT * FROM employees
    WHERE CONTAINS(name, 'SMITH') > 0;
    ```
    - CONTAINS(): 부분 검색, 단어 검색, 패턴 매칭에 매우 빠름
      - 'SMITH'가 포함된 이름을 찾기 위해 FULL SCAN을 하지 않고 텍스트 인덱스를 사용
      - CLOB, VARCHAR2 등 긴 텍스트 컬럼 검색에 적합함
  - INSTR(): 특정 문자열이 어디 위치에 있는지 알려줌, 기본적으로 인덱스를 타지 않음
    ```sql
    SELECT * FROM employees
    WHERE INSTR(name, 'SMITH') > 0;
    ```
    - name 컬럼안에 'SMITH'라는 문자열이 포함되어 있으면 해당 row를 반환
    - INSTR()은 인덱스를 사용하지 않으나 함수기반 인덱스를 만들면 인덱스를 사용할 수 있음
    - 하지만, 위 쿼리처럼 'SMITH'는 동적인 값이 아니고 고정된 값이기 때문에 **동적 검색어를 사용할 때는 매번 새로운 인덱스를 만들 수 없기 때문에 실용성이 떨어짐**
      - 그래서 실무에서는 CONTAINS()를 더 활용함
- IN 조건이 많으면(IN (1, 2, 3, ..., 1000))
  - IN 대신 JOIN이나 EXISTS로 리팩토링하여 사용

### 인덱스 힌트
- 인덱스 힌트의 필요성
  - DB optimizer가 통계정보에 따라 FULL SCAN을 선택할 수 있는데, 특정 상황에서는 인덱스가 더 효율적인 경우가 있어 힌트로 개입
  - Ex) 테이블의 통계정보가 오래되어 카디널리티 예측이 부정확할 때
- ⚠️주의사항
  - 힌트 남발 금지
  - 힌트는 쿼리의 실행 계획을 고정하므로 데이터 분포가 변경되면 성능 저하 발생
```sql
SELECT /*+ INDEX(table_name index_name) */
      컬럼들
FROM 테이블명 table_name
WHERE 조건;
```
- INDEX(..): 지정한 테이블과 인덱스를 강제로 사용
- table_name: 테이블의 별칭(alias) 또는 테이블 명
  - 별칭을 사용할 경우, 힌트에도 동일한 별칭 사용
- index_name: 사용할 인덱스의 이름

1. 특정 인덱스 강제 사용
  ```sql
  SELECT /*+ INDEX(e idx_emp_name) */
        *
  FROM employees e
  WHERE name = 'SMITH';
  ```
2. 인덱스 풀스캔(INDEX_FFS)
   ```sql
  SELECT /*+ INDEX_FFS(e idx_emp_name) */
        name
  FROM employees e;
  ```
  - 테이블의 접근을 최소화하고 인덱스에서 바로 데이터를 반환할 수 있는 경우에 사용
3. 인덱스 스킵 스캔 (INDEX_SS)
  ```sql
  SELECT /*+ INDEX_SS(e idx_emp_dept_job) */
        *
  FROM employees e
  WHERE job_id = 'CLERK';
  ```
  - 원래는 (dept_id, job_id) 인덱스에서 dept_id 조건이 없으면 인덱스를 못쓰지만, 스킵 스캔으로 인덱스를 타게할 수 있음
    - 효율은 보장되지 않음

이외에도 인덱스 힌트가 여러개 있음(INDEX_ASC, INDEX_DESC, INDEX_COMBINE)
