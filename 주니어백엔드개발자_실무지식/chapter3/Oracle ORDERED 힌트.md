# ORDERED hint
- ORDERED hint: 조인 순서를 개발자가 강제로 지정하여 optimizer의 실행 계획을 바꾸는 튜닝 기법

**조인 순서 변경을 통해 불필요한 데이터 접근을 크게 줄인 사례 소개**
```sql
SELECT /*+ ORDERED */
        ...
FROM small_table s, mid_table m, big_table b
WHERE s.key = m.key AND m.key = b.key;
```
- small_table -> mid_table -> big_table 순으로 조인됨

ORDERED 힌트의 역할
- 옵티마이저는 일반적으로 통계 정보를 기반으로 조인 순서를 자동으로 결정함
- 하지만 통계정보가 부정확하거나 데이터 분포가 특정 조건에서 예상과 달라질 때, 비효율적인 조인 순서를 선택할 수 있음
- `/*+ ORDERED */` 힌트를 사용하면 **FROM 절에 작성된 테이블 순서 그대로 조인을 수행**하도록 강제함

개선 전
- 옵티마이저가 대용량 테이블부터 스캔함
- 이후 작은 테이블과 조인하면서 필요없는 레코드까지 먼저 읽게되는 상황이 발생함

개선 후
- 작은 테이블 부터 조인 -> 작은 테이블에서 먼저 필터링하여 big_table 접근 범위를 크게 줄임
  - 대용량 테이블에 접근하는 시점에 이미 후보 데이터가 최소화되어 I/O가 줄어들게 됨
- 읽어야 할 블록 수와 실행 시간이 감소함

### 주의사항
- 테이블 순서를 FROM절에 잘 설계하자
  - 작은 테이블이나 필터링이 잘 되는 테이블을 앞에 둬야 효과적
- 옵티마이저의 통계 정보가 정확하면 굳이 할 필요가 없음
  - ORDERED 힌트는 고정된 조인 순서를 강제함, 데이터 분포가 변하면 성능 저하가 발생할 가능성

ORDERED 힌트와 비슷한 **LEADING 힌트**
- 특정 테이블을 조인 순서의 선두로 명시할 수 있음
```sql
SELECT /*+ LEADING(s m b) */
      ...
FROM small_table s, mid_table m, big_table b
WHERE ...
```
