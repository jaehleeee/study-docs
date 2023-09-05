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

#### 제네릭 가변인수 배열 참조를 밖으로 노출하면 힙 오염 전달 가능
 * 예외적으로, @SafeVarargs 사용한 메서드에 넘기는 것은 안전
 * 예외적으로, 배열의 내용의 일부 함수를 호출하는 일반 메서드로 넘기는 것은 안전

#### 아이템 28 조언에 따라 가변인수를 List로 바꾼다면
 * 배열없이 제네릭만 사용하므로 컴파일 안전성 보장
 * @SafeVarargs 애너테이션을 사용할 필요가 없다.
 * 실수로 안전하다고 판단할 걱정도 없다. 



## 완벽 공략
#### ㅇㅇ
 * ㅇㅇ
#### ㅇㅇ
 * ㅇㅇ
