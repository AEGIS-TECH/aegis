# 1. 실습 환경 구성하기
mysql 이용
기본 환경 setting 후 github 에서 데이터 가져오기
![image](https://github.com/user-attachments/assets/f022a653-5874-4977-a5a2-766735291f05)


# 2. 실행 계획 수행
## 1. 기본 실행 계획 수행
![image](https://github.com/user-attachments/assets/83093300-3ac2-4f8e-911f-dd7870459550)


## 2. 기본 실행 계획 항목 분석
### id 
실행 순서를 표시하는 숫자
SQL 문이 실행되는 차례를 표기한 것 => **작을수록 먼저 실행, 같은 값이면 두 테이블의 조인**

``` 
mysql> explain
    -> select 사원.사원번호 , 사원.이름, 사원.성 , 급여.연봉,
    -> (select MAX(부서번호)
    -> from 부서사원_매핑 as 매핑 where 매핑.사원번호 = 사원.사원번호) 카운트
    -> from 사원, 급여
    -> where 사원.사원번호 = 10001
    -> and 사원.사원번호 = 급여.사원번호;
```
![image](https://github.com/user-attachments/assets/5dacbf34-e920-4a76-8ff0-16ff9c70f0c7)

### select_type
select 문의 유형을 출력하는 항목
from 절, 서브쿼리, union 등

- simple
  단순 select 문
  
  ![image](https://github.com/user-attachments/assets/6d6a2c21-0cf7-4476-8cb6-2b4c1ead2437)

  ![image](https://github.com/user-attachments/assets/c3f3e3cd-f129-485b-a626-f9923b0a599c)

- primary
  서브쿼리가 포함된 sql 문에서 첫 번째 select 문에 해당하는 구문에 표시
  ![image](https://github.com/user-attachments/assets/8902f90f-8383-442b-8079-aed6a461c2d0)

  또는 아래와 같이 Union all 구문으로 통합된 sql 문에서 처음 select 문이 작성된 쿼리
  ![image](https://github.com/user-attachments/assets/0d9d1613-d413-41df-aaf8-8026067f3ac3)

- subquery
  독립적으로 수행되는 서브쿼리
  ![image](https://github.com/user-attachments/assets/83a474ee-bf87-4302-883c-f6f18507f940)


- derived
  from 절에 작성된 서브쿼리 (from 절의 별도 임시 테이블, 인라인 뷰)
  ![image](https://github.com/user-attachments/assets/538c0916-f5c4-49b3-acd7-b690be5da267)

- union
  union 및 union all 구문으로 합쳐진 select 문에서 첫 번째 select 구문을 제외한 이후의 select 구문
  (첫 번째 구문은 primary)
  ![image](https://github.com/user-attachments/assets/c0434504-907c-440a-8861-7e6b9406712a)

- union result
  union all 이 아닌 union 구문으로 select 절을 결합했을 때 출력
  ![image](https://github.com/user-attachments/assets/f1f041e4-d62b-4e8b-bc8a-4916a617b79c)


- dependent subquery & dependent union
  union 또는 union all을 사용하는 서브쿼리가 메인 테이블의 영향을 받는 경우
  dependent subquery는 union 으로 연결된 쿼리들 중 처음으로 작성한 단위 쿼리
  dependent union은 union 으로 연결된 쿼리들 중 두 번째로 작성한 단위 쿼리
  ![image](https://github.com/user-attachments/assets/bec02f1d-c7be-4b1a-8651-70eee1ad1319)

- uncacheable subquery
  메모리에 상주해야할 서브쿼리가 재사용되지 못할 때
  1) 서브쿼리 안에 사용자 정의 함수, 사용자 변수 포함
  2) RAND(), UUID() 등 조회시마다 결과가 달라지는 경우
  ![image](https://github.com/user-attachments/assets/bae6d569-068f-432f-8102-4170f2b4da56)

  rand() 있는 쿼리임에도 불구하고 primary 만 뜸..

- materialized
  in 절 구문에 연결된 서브쿼리가 임시 테이블을 생성한 뒤, 조인이나 가공 작업을 수행할 때 출력되는 유형
  ![image](https://github.com/user-attachments/assets/a9f34d18-96c1-4b10-a70e-bfa5821789fc)

### table
    말 그대로 테이블!
    서브쿼리나 임시 테이블을 만드는 경우에는 <subquery#> 나 <derived#> 라고 출력
    ![image](https://github.com/user-attachments/assets/2c61f75a-c514-4c76-bacc-936475a2fb0e)

### partitions
    실행 계획의 부가 정보, 데이터가 저장된 논리적인 영역을 표시
    너무 많은 영역의 파티션 -> 파티션 정의 자체를 튜닝

### type
    테이블의 데이터를 어떻게 찾을지에 대한 정보 제공 항목
    
