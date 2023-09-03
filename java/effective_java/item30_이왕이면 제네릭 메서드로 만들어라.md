# 30. 이왕이면 제네릭 메서드로 만들어라

## 핵심 정리
### 제네릭 메서드로 쓰면 좋은 케이스
#### 매개변수화 타입을 받는 정적 유틸리티 메서드
 * 한정저거 와일드카드 카입을 사용하면 더 유연하게 개선 가능.
#### 제네릭 싱글턴 팩터리
 * 불변 객체 하나를 어떤 타입으로든 매개변수화 할 수 있다.

 * 제네릭을 사용하지 않은 항등 함수 예시
```
public static Function<String, String> strIdentityFunction(){
  return (t) -> t;
}
public static Function<Number, Number> numIdentityFunction(){
  return (t) -> t;
}
```
 * 제네릭을 사용한 항등 함수 예시
   * 타입이 다르다고 해서 팩터리 메서드를 새롭게 만들 필요가 없다.
```
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarning("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
  return (UnaryOperator<T>) IDENTITY_FN; 
}
```

#### 재귀적 타입 한정
 * 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정한다.
```
public static <E extends Comparable<E>> E max(Collection<E> c) {
  ...
}
```
