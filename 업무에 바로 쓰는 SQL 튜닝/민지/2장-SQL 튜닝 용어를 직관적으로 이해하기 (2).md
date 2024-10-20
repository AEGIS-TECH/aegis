## 4. 조인 연산방식 용어
### 내부 조인
교집합, INNER JOIN
: 양쪽에 모두 존재하는 데이터만 반환

### 왼쪽 외부 조인
LEFT JOIN , LEFT OUTER JOIN
먼저 작성된 왼쪽 테이블 기준으로 나중에 작성된 오른쪽 테이블과 조인
조인 조건과 일치하지 않더라도 왼쪽 테이블의 결과는 최종 결과에 포함
없는 값에 대해서는 NULL로 표시

### 오른쪽 외부 조인
RIGHT JOIN, RIGHT OUTER JOIN
왼쪽 외부 조인의 반대, 나중에 작성된 오른쪽 테이블을 기준으로 먼저 작성된 왼쪽 테이블과 조인
오른쪽 테이블의 결과가 최종 결과에 포함
없는 값에 대해서 NULL 표시

**전체 외부 조인**
왼쪽 외부 조인 + 오른쪽 외부 조인의 형태, MySQL과 MariaDB에서 지원 X

### 교차 조인
데카르트 곱, CROSS JOIN
조인 연산 과정에서 시공간적 리소스 점유 축면에서 오버헤드 발생하므로 주의 요함
WHERE 절의 조건문이나 JOIN 키워드를 명시하지 않고 작성할 경우 수행됨

### 자연 조인
NATURAL JOIN
동일한 열명이 있을 때 조인 조건절을 따로 작성하지 않아도 자동으로 조인 수행
실제로 많이 쓰이지는 않음
자연 조인 했는데 공통 열명 없을 시 -> 교차 조인 수행!

## 5. 조인 알고리즘 용어
선후 관계에 따라 드라이빙 테이블과 드리븐 테이블로 구분
### 드라이빙 테이블
먼저 접근하는 outer table
### 드리븐 테이블
드라이빙 테이블의 검색 결과를 통해 뒤늦게 데이터를 검색하는 innter table

=> 가능하면 **적은 결과가 반환**될 것으로 예상되는 것을 드라이빙 테이블로 선정하고 조인 조건절의 열이 **인덱스**로 설정되도록 구성해야함

### 중첩 루프 조인
NL 조인
드라이빙 테이블의 데이터 1건당 드리븐 테이블 반복하여 검색

1,100에 대한 결과를 찾으려면,

[예제1] 기본 키와 인덱스가 없는 두 테이블

1을 찾는 100건 + 1에 대한 연락망 1000건 = 1100
100을 찾는 100건 + 100에 대한 연락망 1000건 = 1100

=> 2200건

[예제2] 학번 열로 인덱스 생성된 두 테이블

인덱스 설정되어있으므로,
학번 1에 대한 1건 + 1에 대한 연락망 2건 = 3
100에 대한 1건 + 100에 대한 연락망 1건 = 2

=> 5건

이때, 데이터 접근 과정에서 랜덤 액세스 발생. 
랜덤 액세스를 줄일 수 있도록 데이터의 액세스 범위를 좁히는 방향으로 인덱스를 설계하고 조건절을 작성해야함
비고유 인덱스의 경우 랜덤 액세스를 유발,
기본 키는 클러스터형 인덱스이므로 키 순서대로 데이터 적재되어 조회 효율 높음

### 블록 중첩 루프 조인
BNL 조인
중첩 루프 조인의 효율성을 높이고자 탄생
조인 버퍼 개념을 도입하여 성능 향상

첫번째 테이블에서 검색된 데이터를 조인 버퍼에 채우고, 이를 비산연락망의 데이터와 조인하면서 
**한 번의 테이블 풀 스캔으로 원하는 데이터를 모두 찾을 수 있음**

### 배치 키 액세스 조인
BKA 조인
중첩 루프 조인 방식은 필연적으로 랜덤 액세스 발생 -> 액세스할 데이터의 범위가 넓은 경우 비효율적
접근할 데이터를 미리 예상하고 가져오는 방식

- 랜덤 버퍼
  : 드리븐 테이블에 필요한 데이터를 미리 예측하고 정렬된 상태로 담는 버퍼

### 해시 조인
선후 관계를 두고 조인을 수행하는 중첩 루프 조인 방식과 달리,
조인에 참여하는 **각 테이블의 데이터를 내부적으로 해시값**으로 만들어 내부 조인을 수행
조인 버퍼에 저장 -> 조인열의 인덱스를 필수로 요구하지 X

# 3. 개념적인 튜닝 용어
## 1. 기초 용어
### 오브젝트 스캔 유형
- 테이블 풀 스캔 : 인덱스 없이 사용하는 유일한 방식

- 인덱스 범위 스캔 : 인덱스 범위 기준으로 스캔, 스캔 결과로 데이터를 찾아가는 방식
  +좁은범위에 효율, 넓은 범위에 비효율
  
- 인덱스 풀 스캔 : 인덱스를 처음부터 끝까지 수행, 상대적으로 적은 정보 -> 테이블 풀 스캔보다 상대적 우수

- 인덱스 고유 스캔 : 기본 키나 고유 인덱스로 테이블에 접근하는 방식, 인덱스를 사용한 방식 중 가장 효율적

- 인덱스 루스 스캔 : 인덱스의 필요한 부분만 골라 스캔, where 절 조건문 기준으로 데이터 필요 유무를 구분

- 인덱스 병합 스캔 : 테이블 내의 인덱스들을 통합해서 스캔
  결합, 교차 방식

### 디스크 접근 방식
저장된 스토리지의 페이지(데이터를 검색하는 최소 단위)에 접근, 페이지 단위로 읽고 쓰기 수행 가능
- 시퀀셜 액세스
  서로 연결된 페이지를 차례로 읽음
  순차 접근 방식
  테이블 풀 스캔에서 활용
  
- 랜덤 액세스
  여기저기 원하는 페이지를 임의로 읽음
  물리적 위치 고려 X
  데이터의 접근 수행 시간 오래 걸림
  접근 범위를 줄이고 효율적인 인덱스 활용을 하도록 튜닝


### 조건 유형
- 액세스 조건
  
  디스크에 있는 데이터에 어떻게 접근할 것인지 다룸
  
  where 절의 특정 조건문을 이용해 소량의 데이터를 가져오고, 인덱스를 통해 시간 낭비를 줄이는 조건절을 선택 -> 스토리지 엔진의 데이터에 접근 , MySQL 엔진으로 가져옴
  
- 필터 조건

  액세스 조건을 이용해 mysql엔진으로 가져온 데이터를 기준으로, 추가로 가공
  
  필터 조건으로 필터링할 데이터가
  
  -> 없다면 : 매우 훌룡한 SQL문

  -> 있다면 : 비효율적인 SQL문

## 2. 응용 용어
### 선택도
테이블의 특정 열을 기준으로 해당 열의 조건절에 따라 선택되는 데이터 비율

선택도 = 선택한 데이터 건 수 + 전체 데이터 건 수

변형된 선택도 = 1 / DISTINCT(COUNT 열명)

### 카디널리티
하나의 데이터유형으로 정의되는 데이터 행의 개수
전체 행에 대한 특정 열의 중복 수치

카디널리티 = 전체 데이터 건수 * 선택도

### 힌트
데이터를 빨리 찾을 수 있게 추가 정보를 전달
`/*! */` 형태의 주석 사용

- 강력하지 않은 힌트는 옵티마이저가 비효율적이라고 예측하여 무시
- STRAIGHT_JOIN : from 절에 작성된 테이블 순으로 조인을 유도
- USE INDEX : 특정 인덱스를 사용하도록 유도
- FORCE INDEX : 특정 인덱스를 사용하도록 강하게 유도
- IGNORE INDEX : 특정 인덱스를 사용하지 못하도록 유도

### 콜레이션
특정 문자셋으로 데이터베이스에 저장된 값을 비교하거나 정렬하는 작업의 규칙

### 통계정보
옵티마이저가 통계정보에 기반을 두고 실행 계획을 수립
테이블 통계 정보와 인덱스 통계정보 등을 토대로 어떤 인덱스를 활용해 데이터에 액세스 할 것인지, 어떤 테이블을 드라이빙 테이블로 선택할 것인지 등

### 히스토그램
테이블의 열값이 어떻게 분포되어 있는지를 확인하는 통계정보

  
