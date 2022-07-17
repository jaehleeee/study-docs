# 실행계획 활용

## 인덱스 특징
1. 인덱스는 정렬되어 있다.
 - leftmost 정렬
 - 메모리 위에서 존재
 - 시작지점 선택 후 쭉 스캔하는 방식이다.

## 임시테이블
 * 조회한 레코드를 추가로 정렬하거나 그룹핑해야할때 사용.
 * 기본적으로 메모리 사용이지만, 지정된 크기넘으면 디스크 
 * 사용하게 되므로 속도 저하 발생

## 임시테이블 사용하지 않는 팁
 * order by & group by 는 가능한 인덱스에 있는 컬럼들로 처리한다.
 * select * 사용하지 않는다
 * 정렬이 필요하다면 distinct 보다 group by 사용
 * 서브 쿼리는 최대한 조인으로 변경한다.


 ## type
  * range, index, ALL 타입이라면 쿼리 튜닝 고려
  * range : 특정 점위 데이터만 조회, 조회 데이터가 많을 경우 풀 스캔이 더 효율적일수도 있음.
  * index : 인덱스 풀 스캔 의미, 인덱스 내 첫번째 컬럼부터 조회해야한다.

## extra 중 실행계획이 좋지 못한 경우
 * `using filesort` : 인덱스 사용하지 않는 정렬
 * `using temporary` : group by 결과 재정렬을 위해 임시 테이블 생성
 * `using where` : 인덱스에 없는 trad_ymdt로 필터링 필요, 필수 제거는 아니지만 항상 제거 여부를 판단해야함.
 
 ## extra 중 안심해도 되는 값
 * `Distinct` : 조인시 불필요 값은 조인하지 않고 중복 제거
 * `using index` : FILE IO 없이 인덱스만으로 수행
 * `using idex for group-by` : using index와 동일, group-by 까지 처리한 경우
