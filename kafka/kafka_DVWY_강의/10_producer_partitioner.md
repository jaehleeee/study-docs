# 파티셔너

## 기본 제공 파티셔너
 * 프로듀서 API를 사용하면 2개의 파티셔너를 제공한다.
   * `UniformStickyPartitoner`
   * `RoundRobinPartitioner`
 * 카프카 클라이언트 2.5.0 버젼에서 파티셔너를 지정하지 않으면 UniformStickyPartitoner 가 기본 설정된다.
 * 2개의 파티셔너 모두 메시지 키가 있다면 메시키 키의 해시값과 파티션을 매칭한다.
   * **단, 파티션 개수가 변경되면 메시지 키와 파티션 번호 매칭이 깨지게 된다.**
   * 그러니 파티션 개수를 충분히 늘린 상태에서 운영을 시작해야 한다.
 * 메시지 키가 없을때 동작은 다르다.
    * `RoundRobinPartitioner` : 이름처럼 단순 round-robin
    * `UniformStickyPartitoner` : accumulator에서 레코드들이 배치로 묶일때까지 기다렸다가 전송. 배치로 묶은 데이터는 round-robin
 

## 커스텀 파티셔너
 * 라이브러리에서 파티셔너 인터페이스를 제공하고 있으니 이를 상속하여 커스텀하게 만들 수 있다.
