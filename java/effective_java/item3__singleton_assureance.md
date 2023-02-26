# 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 내용 요약
 * 싱글턴 : 인스턴스를 오직 하나만 생성할 수 있는 클래스
 * 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다. (인터페이스 없이 정의한 경우)
    * 클라이언트 코드를 테스트할때마다 싱글턴을 쓰는건, 리소스적으로 비싸다.
    * 대신 인터페이스가 있다면, 이 인터페이스로 구현한 Mock 클래스를 따로 만들어두고 이 Mock 객체로 테스트가 가능하다.
 * 싱글턴을 만드는 방식은 보통 2가지 (+1)


#### 첫번째 방법 : private 생성자 & public static final 멤버로 제공
 * 생성자를 private으로 감춘 후, 유일한 인스턴스 접근 수단으로 public static 멤버를 하나 마련해준다.
 * private 생성자는 public static 멤버를 초기화할 때 한번만 호출
 * 장점1 : 해당 클래스가 싱글턴임이 API에 명백히 드러난다. (final 제어자 덕분)
 * 장점2 : 간결함
 * 단점1 : 리플렉션 API를 이용해 private 생성자를 호출 가능하므로 싱글턴이 깨질 수 있다.
    * 이를 방어하려면 2번째 객체 생성시 예외를 던지게 하면 된다. - 객체내 객체 생성여부를 체크하는 boolean 필드를 하나두고 이를 통해 확인 가능.
    * (간결함이라는 장점은 깨지는..)
 * 단점2 : 클래스를 직렬화/역직렬화하는 경우가 있다. 이때 생성자가 호출된다. 그래서 싱글턴이 깨질 수 있다.
    * 이를 방어하려면, readResolve() 라는 메서드르 클래스 내에 선언해줘야 한다. (override는 아닌데, 직렬화 과정에서 사용되는;; 이상한 구조)
    * `private Object readResolve() {return INSTANCE}`
    * (간결함이라는 장점은 깨지는..)
```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  pirvate Elvis() {};
}
```

#### 두번째 방법 : private 생성자 & private static final 멤버 & public static 팩터리 메서드로 제공
 * 생성자를 private으로 감춘 후, 정적 팩터리 메서드를 통해 private static 멤버를 제공.
 * 장점1 : API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점
    * static 팩터리 메서드에 private 멤버가 아닌, 신규 객체를 생성함으로써 클라이언트 코드 변경없이 싱글턴에서 싱글턴에서 아니게 변경 가능.
 * 장점2 : 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 점
    * 제네릭한 타입으로 싱글턴 팩터리를 만들 수 있다.

```java
public class Meta<T> {
  private static final Meta<Object> INSTANCE = new Elvis();
  pirvate Meta() {};
  @SuppressWarnings("unchecked")
  public static <T> Meta<T> getInstance() {return (Meta<T>) INSTANCE;} 
  // 클래스 옆에 <T>가 있는데 static 옆에 <T>는 왜 들어가는가? scope이 다르기 때문. 여기선 클래스에서 선언한 제네릭과 다른 문자를 사용해도 된다.
  public void say (T t) {log.info(t);} // 하지만 메서드에서는 클래스에 있는 제네릭 T를 그대로 사용해야 한다.
}

---

// `Meta<String> meta1 = Meta.getInstance();`
// `Meta<Integer> meta2 = Meta.getInstance();`
// meta1과 meta2는 같은 해시코드를 가지지만, == 비교는 타입이 달라서 불가능하다.

```

 * 장점3 : 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다
 * 단점은 위 첫번째 방법과 동일하며 해결책도 동일하다.

```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  pirvate Elvis() {};
  public static Elvis getInstance() {return INSTANCE;}
}
```

### 세번째 방법 : 원소가 하나인 Enum 생성
 * 싱글턴 제공 방법 중 가장 권장하는 방법
 * public 필드 방식과 비슷하지만 더 간격하고, 추가 노력없이 직렬화 가능하고, 리플렉션 공격을 막아준다.
    * 리플렉션으로 생성자를 가져오려고 하면, 생성자가 없다는 예외(NoSuchMethodException)가 발생한다. 실제론 있음. 자바 내부적으로 접근을 막아놓았다.
 * 단, 만들려는 클래스가 다른 클래스를 상속해야한다면 이 방법을 사용할 수 없다.



## 완벽 공략
### 메서드 참조
 * https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html
#### 이해를 위한 코드 예제
```java
List<Person> people = Lists.newArrayList();
people.add(new Person("1992.04.04"));
people.add(new Person("1990.07.20"));
people.add(new Person("1999.01.11"));

// 1. 익명 내부 클래스
people.sort(new Comparator<Person>() {
   @Override
   public int compare(Person a, Person b) {
      return a.birth.compareTo(b.birth);
	}
})

// 2. lambda
people.sort((p1, p2) -> p1.birth.compare(p2.birth));

// 3. static 메서드 참조 (Person 클래스에 static 메서드가 있을때)
// compareBirth : int 리턴하고, 파라미터가 2개있는 함수
people.sort(Person::compareBirth);

// 4. 인스턴스 메서드 참조 (Person 클래스에 static 아닌 메서드가 있을때)
Person p = new Person("1992.04.04");
people.sort(p::compareBirth);

// 5. 임의 객체 메서드 참조 (Person 클래스에 static 아닌 메서드가 있을때, 4번 다른 방식)
// compareBirth : int 리턴하고, 파라미터가 1개있는 함수(static에서의 첫번째 인자는 자기 자신으로 간주)
people.sort(Person::compareBirth);

// 6. 생성자 참조 (Function<LocalDate, Person> aNew = Person::new;)
people = dataList.stream().map(Person::new).collect(Collectors.toList());
```


### 함수형 인터페이스
 * https://blogs.oracle.com/javamagazine/post/understanding-java-method-invocation-with-invokedynamic
 * https://docs.oracle.com/javase/8/docs/api/java/lang/invoke/LambdaMetafactory.html
 * 함수형 인터페이스는 람다 표현식과 메소드 참조에 대한 “타겟 타입”을 제공한다.
 * 타겟 타입은 변수 할당, 메소드 호출, 타입 변환에 활용할 수 있다.
 * 함수형 인터페이스는 @FunctionalInterface 어노테이션으로 정의 가능하며, java.util.function 라이브러리에 포함되어 있다.
 * 함수형 인터페이스는 대표적으로 아래 4가지가 있다.


#### Function, Consumer, Supplier, Predicate
 * Function : 2개의 타입을 받는다. 첫번째 타입 -> 두번째 타입 리턴. (ex: `Function<Integer, String> f = Objects::toString`)
 * Supplier : 받는 타입은 없고, 나오는 타입만 있다. (ex: `Supplier<Person> s = Person::new`)
 * Consumer : 받는 타입만 있고, 나오는 타입은 없다. (ex: `Consumer<String> c = System.out::println`)
 * Predicate : 1개의 타입을 받아서 Boolean을 리턴한다. 일종의 Function 이다. (ex: `Predicate<Integer> p = Objects::isNull`)


### 객체 직렬화
 * https://docs.oracle.com/javase/8/docs/platform/serialization/spec/serialTOC.html
 * 객체를 byte 스트림으로 변환하는 기술 (byte -> object는 역직렬화)
 * Serializable 인터페이스 구현
    * 객체 내에서 직렬화하지 않을 필드는 `transient` 를 사용해서 선언 가능
    * static한 필드는 직렬화되지 않음
 * serialVersionUID는 언제 사용하는가?
    * serialVersionUID는 Serializable 인터페이스 구현하면 JVM이 런타임 중에 자동으로 세팅한다
    * 클래스가 변경되면 serialVersionUID가 바뀐다
    * serialVersionUID 이름처럼, 이 값이 바뀐 클래스는 역직렬화가 안된다.
       * 역직렬화하고 싶다면 미리 선언해서 고정시켜두면 된다. 


