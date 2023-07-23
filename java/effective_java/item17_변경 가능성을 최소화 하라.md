# 17. 변경 가능성을 최소화 하라
## 핵심 정리 (불변 클래스)
#### 불변 클래스는 가변 클래스보다 설계 및 구현하기 쉽고, 오류 여지가 적어 훨씬 안전하다
#### 불변 클래스 만드는 5가지 규칙
 * 객체 상태 변경 메서드(setter)를 제공하지 않는다.
 * 확장하지 못하게 한다. (public final class로 선언)
 * 모든 필드를 final로 선언
 * 모든 필드를 private으로 선언
 * 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다

#### 불변 클래스 장단점
 * 함수형 프로그래밍에 적합
 * 불변 객체의 장점이 많다.
    * 단순하다
    * 스레드 safe
    * 안심하고 공유 가능
    * 불변 객체끼리는 내부 데이터 공유 가능
 * 실패 원자성을 제공한다 (item 76 ) : 연산에 사용되더라도 그 연산이 실패했을때 값이 바뀌지 않는다.
 * 단점 : 값이 다르다면 반드시 별도의 객체를 만들어야 한다.

#### 불변 클래스 사용 예시 - 복소수 클래스
```
public final class Complex {
  private final double re;
  private final double im;

  public static final Complex ZERO = new Complex(0, 0);
  public static final Complex ONE = new Complex(1, 0);
  public static final Complex I = new Complex(0, 1);

  ...
}
```

#### 불변 클래스 만들때 고려할 점
 * 상속 막기
    * final 키워드
    * private 생성자 + 정적 팩터리 (상속을 허용해주는 방법이다, 제한된 범위내에서)
 * 재정의가 가능한 클래스는 방어적 복사 사용해야 한다
 * 모든 외부 공개 필드는 final 이어야 한다. 

## 완벽 공략
#### JMM (java memory model)
 * : 자바 프로그래밍 언어에서 다중 스레드 환경에서 메모리 접근과 관련된 규칙과 지침을 정의하는 스펙 (jmm은 메모리 구조가 아님.)
 * 다중 스레드 프로그래밍에서 발생하는 문제들을 방지하고 안정성을 보장
 * jmm이 허용하는 한에서는 어떻게 구현하든 상관없다.
 * jmm에서 final 키워드 의미 : 특정 인스턴스의 final 필드는 초기화가 된 이후에만 사용 가능하다. (JLS 17.5)
 * 기본 원칙
   * 스레드간 가시성 (visibility) : 한 스레드 작업 결과(변경사항)를 다른 스레드에서 볼 수 있어야 한다.
   * 연산 순서 (ordering) : 스레드간 실행 순서를 정확하게 지정하여 일관성을 유지
   * 원자성 (atomicity) : 연산이 원자적으로 실행되어야 한다., 한번에 완전히 실행되거나 전혀 실행되지 않아야 한다.

#### Concurrency
 * 병행 : 여러 작업을 번갈아 가며 실행해, 마치 동시에 처리하는 듯 보이게 한다. cpu가 1개여도 괜찮다. (병렬은 동시 작업이기 때문에 cpu가 여러개 필요)
 * concurrent 패키지 : BlockingQueue, Callable, ConcurrentMap, Executor, ExcutorService, Future

#### CountDownLatch
 * 여러 스레드로 실행하는 여러 오퍼레이션이 마칠때 까지 기다릴 때 사용하는 유틸리티
 * 초기화할때 숫자를 입력하고 await() 메서드를 사용해서 숫자가 0이 될때까지 기다린다
 * 숫자를 셀때 countDown() 메서드를 사용한다.
 * 재사용할 수 있는 인스턴스가 아니다.
 * 시작 또는 종료 신호로 사용할 수 있다
(좀더 이해하고 싶다면 예시를 찾아보자)
