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

### IN 조건과 인덱스 사용

  - Ex) dept_id보다 employee_id가 훨씬 다양한 값을 가진다면 (employee_id, dept_id) 순서가 효율적
  - **쿼리 패턴을 분석하여 WHERE 절에 가장 자주 등장하는 컬럼을 앞에 배치**
