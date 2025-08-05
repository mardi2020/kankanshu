# NESTED LOOPS(NL) JOIN
``` sql
-- 개념적 동작
FOR each row in Table A (Driving Table)  
  FOR each row in Table B (Driven Table)
    IF A.key = B.key THEN
      결과에 포함
```
### 특징
- 순차적 처리: 외부 테이블의 각 행에 대해 내부 테이블을 스캔
- 즉시 결과 반환: 첫 번째 매치를 찾으면 바로 결과 반환
- 메모리 사용량 적음: 추가 메모리 공간 불필요

### 최적 사용 조건
- 소량의 데이터 조인 시
- 인덱스가 잘 구성된 경우
- 첫 번째 결과를 빠르게 원할 때
- OLTP 환경에서 소수 행 처리

# HASH JOIN
``` sql
-- 1단계: Hash Table 생성
BUILD HASH TABLE from smaller table (Build Input)
-- 2단계: 조인 수행  
FOR each row in larger table (Probe Input)
  Hash key로 매치되는 행 검색
```

### 특징
- 전체 스캔 후 처리: 모든 데이터를 읽어 해시 테이블 생성
- 메모리 집약적: 해시 테이블을 메모리에 구축
- 대량 데이터에 효율적: 전체 처리량 관점에서 우수

### 최적 사용 조건
- 대량의 데이터 조인 시
- 인덱스가 없거나 비효율적인 경우
- 전체 결과셋이 필요할 때
- OLAP/DW 환경에서 대용량 처리
