# 31. 한정적 와일드카드를 사용해 API 유연성을 높여라

## 핵심 정리
 * 아래와 같은 예시가 있을때
 * 우리는 Number의 하위 타입인 Integer도 Number 타입의 리스트에 넣고 싶다는 니즈가 있을 수 있다.
```
pulibc void pushAll(Iterable<E> src) {
  for (E e : src) {
    push(e);
  }
}

...

List<Number> numberList = new HashList<>();
Iterable<Integer> integers = Arrays.asList(3, 1, 4, 7, 0);
numberList.pushAll(integers); // 컴파일 에러 발생
```

#### Chooser와 Union API 개선
 * Producer(원소를 push 할때) - Extends
   * 상위 타입의 제네릭 클래스엔 하위 타입의 원소를 넣을 수 있다.
 * 하위타입까지 포함시키려면 한정적 와일드카드를 활용하면 된다.

```
pulibc void pushAll(Iterable<? extends E> src) {
  for (E e : src) {
    push(e);
  }
}
```
 * Consumer(월소를 get 할때) - Super
   * 하위 타입의 제네릭 클래스를 꺼내쓸때 상위 타입의 제네릭 클래스에 선언하여 쓸 수 있다.
```
pulibc void popAll(Collection<? super E> dst) {
  while (!isEmpty()) dst.add(pop());
}
```
#### Comparator와 Comparable은 소비자
 * Comparable은 을 직접 구현할지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하려면 와일드카드가 필요하다.

#### 와일드카드 활용 팁 
 * 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라
   * 한정적 타입이라면 한정적 와일드카드로
   * 비한정적 타입이라면 비한정적 와일드카드로
 * 와일드카드 하나만은 잘 쓰이지 않는다. 불특정 타입을 의미하므로 null 말고는 넣을 수 없다.


## 완벽 공략
#### 타입 추론
 * ㅇㅇ
#### ㅇㅇ
 * ㅇㅇ
