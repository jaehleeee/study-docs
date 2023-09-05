# 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라.
 * 제네릭과 가변인수 함께 사용하는 예시 `methodName(List<String>... stringLists)`
#### 제네릭과 가변인수를 함께 사용함으로써 발생한 힙 오염 예시
```
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = List.of(1, 4);
  Ojbect[] objects = stringLists;
  objects[0] = intList; // 힙 오염 발생
  String s = stringLists[0].get(0); // ClassCastException 발생 
}
```

## 핵심 정리
#### 제네릭 가변 인수 배열에 값을 저장하는 것은 안전하지 않다.
 * 힙 오염 발생 가능(컴파일 경고)
 * 자바7에 추가된 @SafeVarargs 애노테이션 사용 가능 

#### 제네릭 가변인수 배열 참조를 밖으로 노출하면 안된다. 즉 리턴하면 안된다. (노출시 힙 오염 전달 가능)
 * 예외적으로, @SafeVarargs 사용한 메서드에 넘기는 것은 안전
 * 예외적으로, 배열의 내용의 일부 함수를 호출하는 일반 메서드로 넘기는 것은 안전

#### 아이템 28 조언에 따라 가변인수를 List로 바꾼다면
 * 배열없이 제네릭만 사용하므로 컴파일 안전성 보장
 * @SafeVarargs 애너테이션을 사용할 필요가 없다.
 * 실수로 안전하다고 판단할 걱정도 없다. 



## 완벽 공략
#### ThreadLocal
 * 여러 쓰레드에서 공유되는 변수는 쓰레드 안전성 관련 문제가 발생할 수 있다.
   * 경합 또는 경쟁조건
   * 교착 상태
   * Livelock : 서로 계속 lock만 교체하고 접근은 못하는 상태
 * ThreadLocal
   * 쓰레드 지역변수를 사용하면 동기화 하지 않아도 한 쓰레드에서만 접근 가능하므로 쓰레드 안전하다.
   * 한 쓰레드 내에서 공유하는 데이터로, 메서드 매개변수에 매번 전달하지 않고 전역 변수처럼 사용할 수 있다.

#### Java에서 동시성을 해결하는 3가지 해결책
1. synchronized: 안전하게 동시성 보장하지만 비용이 크다.
2. volatile: Cpu별로 가지고 있는 cpu cache memory가 아닌, main memory를 사용하는 방식이므로 동시 read는 가능하지만 동시 write로 인한 문제는 해결하지 못한다.
3. Atomic 클래스: CAS(CompareAndSwap) 을 이용하여 동시성 보장하므로 synchronized 보다 적은 비용으로 동시성 보장한다.
   * CAS(CompareAndSwap) 이란? 현재의 스레드가 존재하는 CPU의 cpu cache memory와 main memory에 저장된 값을 비교하여 일치하지 않으면 교체하는 방식이다. 

#### ThreadLocalRandom : 스레드 로컬 랜덤값 생성기
 * java.util.Random은 멀티 스레드 환경에서 CAS(CompareAndSwap) 으로 인해 실패할 가능성이 있어서 성능이 좋지 않다.
 * ThreadLocalRandom을 사용하면 스레드 전용이므로 스레드간 간섭이 발생하지 않는다. 



