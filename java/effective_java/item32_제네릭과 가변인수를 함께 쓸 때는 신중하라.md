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
   * Livelock
 * ThreadLocal
   * 쓰레드 지역변수를 사용하면 동기화 하지 않아도 한 쓰레드에서만 접근 가능하므로 쓰레드 안전하다.
   * 한 쓰레드 내에서 공유하는 데이터로, 메서드 매개변수에 매번 전달하지 않고 전역 변수처럼 사용할 수 있다.

#### ㅇㅇ
 * ㅇㅇ



