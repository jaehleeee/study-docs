# 6. 불필요한 객체 생성을 피하라
## 핵심 정리
#### new String() 으로 객체를 만들지 마라.
 * 상수 pool에서 동일한 문자가 있어도 참조하지 못하게 한다.
 * 메모리를 많이 사용한다.

#### 정규표현식 객체는 생성에 시간이 좀 소요된다.
 * 패턴을 자주 사용한다면, static 으로 객체를 미리 한번만 만들어둬라.

#### (jvm이 자동으로 해주는) 불필요한 오토박싱, 언박싱을 피하라



## 완벽 공략
#### deprecated API
 * `@deprecated` : java 1.1부터
 * `@Deprecated` : java 5부터
 * `@link` : deprecated된 API 대신 쓸 수 있는 대안을 권장
 * `@Deprecated(forRemoval = true, since = '1.2")` : 삭제 예정도 알려줄 수 있게 추가되었다. 1.2는 버젼 (java 9부터)

#### 정규 표현식 사용처
 * 문자 1개일땐, compile된 패턴 객체(`Pattern.compile(";")`)를 굳이 안써도 된다.
    * 정규표현식 메서드(`stringObject.split(";")`)를 써도 된다. 내부구조상 성능 차이가 거의 없다.

#### 가비지 컬렉션
 * 기본 과정 : Mark -> Sweep -> Compact
    * Mark : 참조가 남아있는지 체크
    * Sweep : Mark된 객체를 heap에서 제거
    * Compact : Sweep되어 파편화된 메모리 공간을 정리
 * Young Generation(Eden, S0, S1) : Eden 최초에 여기로 다 모아서 꽉차면 S0,S1으로 보낸다. -> { S0 <-> S1 }
 * Old Generation : 오래 살아남은 객체
 * Minor GC, Full GC
 * gc를 평가하는 관점 : Throughput, Latency(Stop-the-world), Footprint (gc를 위한 메모리) 
 * Serial, Paralllel, 
    * Serial : cpu가 낮으면 이걸 써야한다.
    * Paralllel : Serial과 같은 방식인데, 스레드를 많이 쓰므로 cpu가 좀 더 필요하다.
 * CMS, G1, ZGC, Shenandoah
    * 이후 나온 gc들은 latency 최적화되어 있다.
 

#### 초기화 지연 기법 (아이템83)
#### 방어적 복사 (아이템 50)
