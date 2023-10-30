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

#### Chooser와 Union API 개선 : PECS(producer-extends, consumer-super)
 * 매개변수 T가 클래스에서 사용할 원소를 생산한다면 producer
 * 매개변수 T가 클래스에서 사용된 원소를 소비한다면(가져간다면) consumer
 * 단, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다.
     * 타입을 정확히 지정해야 하는 상황으로, 이때는 와일드 카드 타입을 쓰지 말아야 한다.

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
 * 보통은 그냥 제네릭을 써라.
   * 그러다 제네릭을 사용한 클래스를 확장할때 와일드카드를 활용해라. (`? extends E` 이런식으로)
 * 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라
   * 한정적 타입이라면 한정적 와일드카드로
   * 비한정적 타입이라면 비한정적 와일드카드로
 * 와일드카드 하나만 (비한정적 와일드카드) 잘 쓰이지 않는다. 불특정 타입을 의미하므로 null 말고는 넣을 수 없다.
 * 비한정적 와일드카드 활용 방안
   * Object 클래스의 메서드를 쓸때 (ex: 무슨 타입인진 모르겠지만 toString으로 출력할거야)
```
public static void printList(List<?> list) {
  for (Object e : list)
    log.info(e + " ")
}
```

## 완벽 공략
#### 타입 추론 (Type Inference)
 * 타입을 추론하는 컴파일러의 기능
    * `List<Box<Integer>> boxList = new ArrayList<>();` 우항은 `<>` 만 쓰더라도 좌항을 통해 타입을 추론한다.
 * 모든 인자의 가장 구체적인 공통 타입
 * 제네릭 메서드 : 메세드 매개변수를 기반으로 추론
 * 제네릭 클래스 생성자를 호출할때 `<>` 사용하면 타입을 추론한다.
 * 자바 컴파일러는 `타겟 타입`을 기반으로 호출하는 제네릭 메서드의 타입 매개변수를 추론한다.
   * `List<String> stringList = Collections.emptyList();` String 이라는 타겟 타입을 통해 타입을 추론한다.
   * 자바8부터는 `타겟 타입`이 `메서드의 인자` 까지 확장되면서 타입 추론이 강화되었다.
     * `BoxExample.processStringList(Collections.emptyList());` processStringList 라는 메서드의 인자로 String 타입 리스트가 정의되어 있음을 통해 String 리스트를 추론하여 제공한다. 



